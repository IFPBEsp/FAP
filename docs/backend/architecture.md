# Arquitetura de backend

Referência arquitetural para o backend de sistemas de informação, pensada como ponto de partida de novos projetos antes de implementar os domínios de negócio.

A stack inicial é Java, Spring Boot e PostgreSQL, em um monólito modular com Clean Architecture leve.

Os princípios que orientam todo o resto do documento: começar simples, organizar por domínio, separar responsabilidades, manter as regras próximas do negócio, isolar infraestrutura e integrações, preservar histórico quando ele importa e criar abstrações apenas quando houver benefício real.

---

## 1. Estilo arquitetural

A aplicação é entregue como um único backend, dividido internamente em módulos funcionais:

```text
backend
 ├── shared
 ├── auth
 ├── users
 ├── module-a
 ├── module-b
 ├── integrations
```

A arquitetura não antecipa domínios que ainda não existem. O monólito modular entrega deploy único, transações simples, desenvolvimento local simples, menos complexidade operacional e comunicação direta entre módulos. A separação interna por domínio reduz o acoplamento e deixa a porta aberta para extrair serviços mais tarde, se isso fizer falta. Microserviços não são o ponto de partida.

## 2. Stack inicial

Previstos desde o começo: Java, Spring Boot (Web, Security, Data JPA), PostgreSQL, Bean Validation, OpenAPI, ProblemDetail e Docker.

Entram conforme a necessidade aparecer: migrations, mapeamento de DTOs, storage, cache, mensageria, jobs e observabilidade. Nenhuma ferramenta deve ser exigida pela arquitetura antes de ter uso concreto.

## 3. Organização por domínio

Em vez de uma estrutura puramente técnica (`controllers`, `services`, `repositories`, `entities`), agrupe o código por domínio, com as camadas dentro de cada módulo:

```text
src/main/java/com/company/project
 ├── shared
 ├── auth
 ├── users
 ├── integrations
 ├── products
 │   ├── api
 │   ├── application
 │   ├── domain
 │   └── infrastructure
 └── orders
     ├── api
     ├── application
     ├── domain
     └── infrastructure
```

Código do mesmo contexto fica junto. Nem todo módulo precisa das quatro camadas: a estrutura acompanha a complexidade real.

### `api`

Interface externa da aplicação: controllers, requests, responses, validação de entrada e contratos HTTP. Recebe a requisição, valida a entrada, chama a aplicação e devolve a resposta. Não concentra regra de negócio.

### `application`

Casos de uso e coordenação do fluxo: services, use cases, commands, queries, transações, mapeamentos e orquestração. Comece simples e quebre em casos de uso menores quando a complexidade justificar.

### `domain`

Conceitos e regras do negócio: entidades, enums, value objects, métodos e eventos de domínio, exceções. As regras importantes ficam próximas dos objetos que representam o negócio, em operações expressivas como `order.cancel()`, `payment.complete()` ou `document.approve()`, em vez de alterações indiscriminadas de estado.

### `infrastructure`

Detalhes técnicos: persistência, repositories, integrações externas, storage, clients HTTP, adapters e implementações de gateways. Infraestrutura é detalhe da aplicação, não regra de negócio.

### Fluxo principal

```text
HTTP Request → Controller → Application → Domain → Repository / Gateway / Provider → Infrastructure
```

A dependência caminha da interface externa em direção às regras da aplicação.

## 4. Shared

`shared` guarda apenas o que é de fato transversal: configuração, segurança, tratamento de erros, paginação, auditoria e validações comuns. O risco conhecido é virar depósito de classes sem domínio claro, então na dúvida vale a regra: domínio antes de shared.

## 5. Autenticação e autorização

A estratégia de autenticação varia por projeto, e uma implementação comum combina access token com refresh token. O backend valida credenciais, emite, renova e encerra a sessão, e identifica o usuário autenticado.

A autorização acontece no backend e pode controlar leitura, criação, edição, remoção, aprovação e a execução de ações específicas. O frontend adapta a interface com base nessas permissões, mas isso nunca substitui a proteção no servidor.

## 6. Banco de dados

O banco relacional inicial é PostgreSQL, com migrações versionadas, identificadores consistentes, auditoria quando necessária, soft delete quando fizer sentido, índices baseados em consultas reais e paginação e filtros para grandes conjuntos.

Alterações de schema não dependem de mudança manual em cada ambiente: código mais migration devem produzir um schema reproduzível, aplicável de forma previsível em qualquer ambiente.

### Auditoria

Entidades que precisam de rastreabilidade podem carregar `createdAt`, `updatedAt`, `createdBy` e `updatedBy`. Use auditoria onde houver valor funcional ou operacional, não em todas as entidades por hábito.

### Soft delete

Soft delete serve a registros cadastrais e costuma ser inadequado para informações que precisam preservar histórico explícito, porque soft delete não é histórico de negócio.

## 7. Contratos e validação

A API não expõe entidades de persistência diretamente. Separe Request, Domain/Entity e Response para manter o contrato HTTP desacoplado da estrutura interna.

A validação estrutural acontece antes do caso de uso, e a regra de domínio depois dele:

```text
Request → Validação estrutural → Application → Regra de negócio
```

Formato e regra de domínio são responsabilidades diferentes.

## 8. Persistência e listagens

Repositories representam o acesso aos dados, sempre por intermédio da aplicação:

```text
Controller → Application → Repository
```

Controllers não acessam repositories. Consultas mais complexas podem usar mecanismos específicos conforme a necessidade.

Listagens devem prever paginação, busca, filtros e ordenação quando o volume justificar, com contrato consistente entre os módulos. Quando os filtros forem combináveis, evite criar um método de repository para cada combinação.

## 9. Tratamento de erros

O tratamento é centralizado e devolve respostas padronizadas. As categorias comuns são validação, recurso não encontrado, não autenticado, não autorizado, conflito, regra de negócio, integração externa e erro inesperado.

## 10. API REST e OpenAPI

A API segue convenções consistentes: recursos no plural, rotas previsíveis, DTOs de entrada e saída, status HTTP adequados, paginação e filtros consistentes, erros padronizados e documentação. Versionamento entra quando existir necessidade real de manter contratos incompatíveis.

O contrato é descrito em OpenAPI, cobrindo endpoints, requests, responses, autenticação, erros, paginação e filtros. Ele também serve de base para gerar clients em outros sistemas.

## 11. Integrações externas

Integrações ficam isoladas das regras de negócio, atrás de um gateway ou provider:

```text
Application → Gateway / Provider → Infrastructure → External API
```

Isso vale para pagamentos, assinatura, storage, email, ERP, CRM, serviços governamentais e provedores de identidade. Secrets e credenciais vêm de configuração externa.

### Estado de processos externos

Quando a integração tem ciclo de vida, persista localmente o estado relevante (`PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`). Com isso é possível consultar estado, exibir histórico, processar callbacks, fazer retry e auditar falhas, sem depender do estado temporário devolvido pelo provedor.

### Webhooks

Webhooks são entradas externas da aplicação:

```text
External Provider → Webhook → Validação → Idempotência → Application → Domain
```

Valide a autenticidade pelo mecanismo que o provedor oferecer.

### Idempotência

Operações que podem se repetir precisam ser avaliadas quanto à idempotência, principalmente em webhooks, jobs, retries, integrações externas e operações financeiras. Executar a mesma operação de novo não deve gerar efeitos duplicados indevidos.

## 12. Storage

Módulos de negócio não dependem de SDKs específicos de storage. A abstração fica no caminho, para que a implementação concreta mude sem tocar no domínio:

```text
Application → Storage abstraction → Infrastructure → Storage provider
```

## 13. Interfaces e padrões

Interfaces entram quando existe necessidade real de abstração: gateways, providers, strategies, adapters, ports, integrações externas e casos com múltiplas implementações. Não crie o par `Service`/`ServiceImpl` para todo serviço interno. Interface quando houver motivo, classe concreta quando for suficiente.

O mesmo vale para os padrões de projeto. Repository, Gateway, Provider, Strategy, Factory, Query Service, Domain Events e State resolvem problemas concretos e não devem ser introduzidos por formalidade: um padrão precisa reduzir complexidade, não criar.

## 15. Jobs, eventos e transações

Rotinas agendadas apenas disparam casos de uso, e devem ser seguras para reexecução sempre que possível:

```text
Job → Application → Domain → Repository
```

Nada de regra de negócio no scheduler.

Módulos conversam pela camada de aplicação uns dos outros. Prefira `Module A → Module B Application` a `Module A → Module B Repository` assim que o acesso direto começar a gerar acoplamento.

A aplicação começa síncrona enquanto isso for suficiente. Eventos internos ou processamento assíncrono entram quando houver processamento demorado, retry, grande volume, necessidade de desacoplamento ou várias reações ao mesmo evento. Filas e mensageria não entram por antecipação.

Transações são controladas na camada de aplicação e representam o limite de um caso de uso. Evite transação em controller e evite manter transação aberta durante operação externa demorada sem necessidade.

## 16. Configuração, logs e segurança

A aplicação recebe configuração por variáveis de ambiente, agrupadas por aplicação, banco, autenticação, storage, integrações, jobs e observabilidade. Secrets nunca ficam no código-fonte.

Os logs registram o suficiente para operar e diagnosticar (inicialização, falhas, integrações, jobs, webhooks e operações relevantes) e nunca senhas, tokens, secrets ou dados sensíveis sem necessidade. Healthchecks ficam disponíveis para a infraestrutura quando necessário.

Segurança mínima, aplicada independentemente do que o frontend faça: senhas armazenadas de forma segura, autenticação e autorização no backend, secrets fora do código, entrada validada, erros sem exposição de detalhes internos, logs sem credenciais, integrações protegidas e arquivos privados protegidos.

## 17. Estrutura do repositório

```text
backend
 ├── docs
 ├── src
 │   ├── main
 │   │   ├── java
 │   │   └── resources
 │   └── test
 ├── .env.example
 ├── Dockerfile
 └── docker-compose.yml
```

Os detalhes variam conforme a implementação escolhida.

## 18. Ordem inicial de implementação

1. Criar o projeto backend.
2. Configurar banco e migrations.
3. Criar a estrutura modular e o `shared` básico.
4. Configurar tratamento de erros e validação.
5. Configurar segurança e implementar autenticação e autorização.
6. Configurar a documentação da API.
7. Implementar o primeiro domínio.
8. Implementar integrações quando forem necessárias.
9. Adicionar jobs, storage e eventos conforme a necessidade.
10. Evoluir a arquitetura conforme a complexidade real.

## 19. O que evitar

- Tudo em controllers, tudo em services ou tudo em `shared`.
- Controller acessando repository ou contendo regra de negócio.
- Entity exposta diretamente pela API.
- Domínio dependendo de SDK externo.
- Secrets no código.
- Interfaces, factories e strategies sem necessidade ou sem variação real.
- Mensageria sem necessidade e microserviços antes da hora.
- `RuntimeException` genérica para qualquer erro.
- Regra de negócio dentro de jobs.
- Regras duplicadas entre módulos.

## 20. Resumo

A arquitetura inicial é Java, Spring Boot e PostgreSQL em um monólito modular com Clean Architecture leve, organizado em `shared`, `auth`, `users`, os módulos de negócio e `integrations`. Cada módulo pode ter `api`, `application`, `domain` e `infrastructure`, e o fluxo principal atravessa essas camadas nessa ordem:

```text
HTTP → API → Application → Domain → Repository / Gateway / Provider → Infrastructure
```

O backend continua responsável pelas regras críticas do sistema. Controllers permanecem simples, as regras ficam próximas do domínio, infraestrutura e integrações ficam isoladas e abstrações só entram quando resolverem um problema real.

Este documento não define cada detalhe da implementação. Ele estabelece os fundamentos que orientam o desenvolvimento: começar simples, organizar por domínio, separar responsabilidades, proteger as regras de negócio, isolar a infraestrutura, preservar histórico e evoluir só quando houver necessidade.
