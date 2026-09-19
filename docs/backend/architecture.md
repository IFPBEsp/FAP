# Arquitetura Genérica de Backend

## 1. Visão Geral

Este documento descreve uma arquitetura inicial e reutilizável para o backend de sistemas de informação.

A proposta é servir como referência arquitetural para novos projetos antes da implementação dos domínios específicos do negócio.

A stack inicial será baseada em:

```text
Java
Spring Boot
PostgreSQL
```

seguindo:

```text
Monólito Modular
+
Clean Architecture leve
```

Princípios principais:

```text
Começar simples.

Organizar por domínio.

Separar responsabilidades.

Manter regras próximas do negócio.

Isolar infraestrutura e integrações externas.

Preservar histórico quando necessário.

Criar abstrações apenas quando houver benefício real.
```

---

# 2. Estilo Arquitetural

A aplicação será inicialmente entregue como um único backend.

Internamente será dividida em módulos funcionais.

Exemplo conceitual:

```text
Backend
 ├── shared
 ├── auth
 ├── users
 ├── module-a
 ├── module-b
 ├── integrations
 └── dashboard
```

Os módulos de negócio serão definidos pelos requisitos de cada projeto.

A arquitetura não deve antecipar domínios que ainda não existem.

---

# 3. Monólito Modular

O projeto não deve começar automaticamente utilizando microserviços.

O monólito modular permite inicialmente:

```text
um único deploy
transações simples
desenvolvimento local simples
menor complexidade operacional
comunicação direta entre módulos
```

Ao mesmo tempo, a separação interna por domínio reduz o acoplamento e facilita uma possível evolução futura.

---

# 4. Stack Inicial

Ferramentas previstas:

```text
Java
Spring Boot
Spring Web
Spring Security
Spring Data JPA
PostgreSQL
Bean Validation
OpenAPI
ProblemDetail
Docker
```

Outras ferramentas podem ser adicionadas conforme necessidade do projeto.

Exemplos:

```text
migrations
mapeamento de DTOs
storage
cache
mensageria
jobs
observabilidade
```

A arquitetura não deve exigir ferramentas que ainda não possuem uso concreto.

---

# 5. Organização por Domínio

Evitar uma estrutura puramente técnica como:

```text
controllers
services
repositories
entities
```

Preferir organização por domínio:

```text
products
 ├── api
 ├── application
 ├── domain
 └── infrastructure

orders
 ├── api
 ├── application
 ├── domain
 └── infrastructure
```

Código relacionado ao mesmo contexto deve permanecer próximo.

---

# 6. Estrutura Geral

Estrutura conceitual:

```text
src/main/java/com/company/project
 ├── shared
 ├── auth
 ├── users
 ├── integrations
 ├── dashboard
 ├── module-a
 └── module-b
```

Cada módulo poderá possuir:

```text
api
application
domain
infrastructure
```

Nem todo módulo precisa obrigatoriamente de todas as camadas.

A estrutura deve acompanhar a complexidade real.

---

# 7. Camada `api`

Responsável pela interface externa da aplicação.

Pode conter:

```text
controllers
requests
responses
validação de entrada
contratos HTTP
```

Responsabilidades principais:

```text
receber requisição
validar entrada
chamar aplicação
retornar resposta
```

Não deve concentrar regras de negócio.

---

# 8. Camada `application`

Responsável pelos casos de uso e coordenação do fluxo da aplicação.

Pode conter:

```text
services
use cases
commands
queries
transações
mapeamentos
orquestração
```

Princípio:

```text
Começar simples.

Separar em casos de uso menores
quando a complexidade justificar.
```

---

# 9. Camada `domain`

Responsável pelos conceitos e regras do negócio.

Pode conter:

```text
entidades
enums
value objects
regras de negócio
métodos de domínio
eventos de domínio
exceções de domínio
```

Regras importantes devem permanecer próximas dos objetos que representam o negócio.

Preferir operações expressivas como:

```text
order.cancel()
payment.complete()
document.approve()
```

em vez de alterações indiscriminadas de estado.

---

# 10. Camada `infrastructure`

Responsável pelos detalhes técnicos.

Pode conter:

```text
persistência
repositories
integrações externas
storage
clients HTTP
adapters
implementações de gateways
```

Infraestrutura deve ser tratada como detalhe da aplicação, não como regra de negócio.

---

# 11. Fluxo Principal

Fluxo conceitual:

```text
HTTP Request
     ↓
Controller
     ↓
Application
     ↓
Domain
     ↓
Repository / Gateway / Provider
     ↓
Infrastructure
```

A dependência deve caminhar da interface externa em direção às regras da aplicação.

---

# 12. Shared

O módulo `shared` deve conter apenas elementos realmente transversais.

Exemplos:

```text
configuração
segurança
tratamento de erros
paginação
auditoria
validações comuns
```

Evitar transformar `shared` em depósito de classes sem domínio claro.

Princípio:

```text
Domínio antes de Shared.
```

---

# 13. Autenticação

A aplicação deverá possuir uma estratégia de autenticação adequada ao projeto.

Uma implementação comum pode utilizar:

```text
Access Token
+
Refresh Token
```

O backend será responsável por:

```text
validar credenciais
emitir sessão
renovar sessão
encerrar sessão
identificar usuário autenticado
```

Detalhes concretos podem variar conforme o projeto.

---

# 14. Autorização

A autorização deve acontecer no backend.

Permissões poderão controlar:

```text
leitura
criação
edição
remoção
aprovação
execução de ações específicas
```

O frontend poderá adaptar a interface com base nessas permissões, mas nunca substituir a proteção no backend.

---

# 15. Banco de Dados

O banco relacional inicial será:

```text
PostgreSQL
```

Princípios:

```text
migrações versionadas
identificadores consistentes
auditoria quando necessária
soft delete quando fizer sentido
índices baseados em consultas reais
paginação e filtros para grandes conjuntos
```

Alterações de schema não devem depender de mudanças manuais em cada ambiente.

---

# 16. Migrations

A estrutura do banco deve evoluir através de migrations versionadas.

Princípio:

```text
Código
+
Migration
=
Schema reproduzível
```

Toda alteração estrutural relevante deve poder ser aplicada de maneira previsível nos diferentes ambientes.

---

# 17. Auditoria

Entidades que necessitam rastreabilidade podem possuir informações como:

```text
createdAt
updatedAt
createdBy
updatedBy
```

Auditoria deve ser utilizada quando houver valor funcional ou operacional.

Não precisa ser aplicada indiscriminadamente a todas as entidades.

---

# 18. Soft Delete

Soft delete deve ser utilizado somente quando fizer sentido para o negócio.

Pode ser adequado para registros cadastrais.

Pode ser inadequado para informações que precisam preservar histórico explícito.

Princípio:

```text
Soft delete não substitui histórico de negócio.
```

---

# 19. DTOs

A API não deve expor diretamente entidades de persistência.

Separar conceitualmente:

```text
Request
Domain / Entity
Response
```

Isso mantém o contrato HTTP desacoplado da estrutura interna.

---

# 20. Validação

A aplicação deve validar os dados de entrada antes da execução do caso de uso.

Fluxo:

```text
Request
 ↓
Validação estrutural
 ↓
Application
 ↓
Regra de negócio
```

Validações de formato e regras de domínio são responsabilidades diferentes.

---

# 21. Persistência

Repositories representam o acesso aos dados.

Fluxo esperado:

```text
Controller
 ↓
Application
 ↓
Repository
```

Controllers não devem acessar repositories diretamente.

Consultas mais complexas podem utilizar mecanismos específicos conforme a necessidade.

---

# 22. Paginação, Busca e Filtros

Listagens devem considerar:

```text
paginação
busca
filtros
ordenação
```

quando o volume de dados justificar.

O contrato deve ser consistente entre os módulos.

Evitar criar métodos de repository excessivamente específicos quando filtros combináveis forem necessários.

---

# 23. Tratamento de Erros

O tratamento de erros deve ser centralizado e consistente.

Categorias comuns:

```text
validação
recurso não encontrado
não autenticado
não autorizado
conflito
regra de negócio
integração externa
erro inesperado
```

A API deve retornar respostas de erro padronizadas.

---

# 24. API REST

A API deverá seguir convenções consistentes.

Princípios:

```text
recursos no plural
rotas previsíveis
DTOs de entrada e saída
status HTTP adequados
paginação consistente
filtros consistentes
erros padronizados
documentação da API
```

O versionamento pode ser introduzido quando existir necessidade real de manter contratos incompatíveis.

---

# 25. OpenAPI

A API deverá possuir documentação de contrato.

OpenAPI poderá descrever:

```text
endpoints
requests
responses
autenticação
erros
paginação
filtros
```

O contrato poderá também servir como base para geração de clients em outros sistemas.

---

# 26. Integrações Externas

Integrações externas devem permanecer isoladas das regras de negócio.

Exemplos:

```text
pagamentos
assinatura
storage
email
ERP
CRM
serviços governamentais
provedores de identidade
```

Fluxo conceitual:

```text
Application
    ↓
Gateway / Provider
    ↓
Infrastructure
    ↓
External API
```

Secrets e credenciais devem vir de configuração externa.

---

# 27. Interfaces

Interfaces devem ser utilizadas quando houver necessidade real de abstração.

Faz sentido principalmente para:

```text
gateways
providers
strategies
adapters
ports
integrações externas
múltiplas implementações
```

Não criar automaticamente:

```text
Service
ServiceImpl
```

para todo serviço interno.

Princípio:

```text
Interface quando houver motivo.

Classe concreta quando for suficiente.
```

---

# 28. Padrões de Projeto

Padrões devem resolver problemas reais.

Podem ser utilizados quando necessário:

```text
Repository
Gateway
Provider
Strategy
Factory
Query Service
Domain Events
State
```

Não devem ser introduzidos apenas por formalidade.

Princípio:

```text
Padrão deve reduzir complexidade,
não criar complexidade.
```

---

# 29. Histórico de Negócio

Algumas operações não devem simplesmente sobrescrever o registro anterior.

Exemplos conceituais:

```text
renegociação
reabertura
substituição
reprocessamento
nova versão
reenvio
```

Quando o histórico for relevante:

```text
registro anterior
      ↓
nova operação
      ↓
novo estado ou registro
      ↓
vínculo entre os dois
```

Princípio:

```text
Não destruir informações necessárias
para explicar como o estado atual foi alcançado.
```

---

# 30. Storage

Módulos de negócio não devem depender diretamente de SDKs específicos de storage.

Fluxo:

```text
Application
    ↓
Storage abstraction
    ↓
Infrastructure
    ↓
Storage provider
```

A implementação concreta poderá variar sem alterar o domínio.

---

# 31. Processos Externos

Quando uma integração possuir ciclo de vida, o estado relevante deverá ser persistido localmente quando necessário.

Exemplos:

```text
PENDING
PROCESSING
COMPLETED
FAILED
```

Isso permite:

```text
consultar estado
exibir histórico
processar callbacks
realizar retry
auditar falhas
```

A aplicação não deve depender exclusivamente do estado temporário retornado pelo provedor externo.

---

# 32. Webhooks

Webhooks devem ser tratados como entradas externas da aplicação.

Fluxo conceitual:

```text
External Provider
      ↓
Webhook
      ↓
Validação
      ↓
Idempotência
      ↓
Application
      ↓
Domain
```

Quando possível, validar autenticidade através do mecanismo oferecido pelo provedor.

---

# 33. Idempotência

Operações que podem ser repetidas precisam ser avaliadas quanto à idempotência.

Isso é especialmente importante em:

```text
webhooks
jobs
retries
integrações externas
operações financeiras
```

Princípio:

```text
Executar novamente a mesma operação
não deve gerar efeitos duplicados indevidos.
```

---

# 34. Jobs

Rotinas agendadas devem apenas disparar casos de uso.

Fluxo:

```text
Job
 ↓
Application
 ↓
Domain
 ↓
Repository
```

Evitar concentrar regras de negócio diretamente no scheduler.

Jobs devem ser seguros para reexecução sempre que possível.

---

# 35. Comunicação Entre Módulos

Módulos devem se comunicar através de interfaces claras da camada de aplicação.

Preferir:

```text
Module A
   ↓
Module B Application
```

em vez de:

```text
Module A
   ↓
Module B Repository
```

quando o acesso direto começar a gerar acoplamento.

---

# 36. Eventos e Processamento Assíncrono

A aplicação deve começar síncrona quando isso for suficiente.

Eventos internos ou processamento assíncrono podem ser introduzidos quando existirem necessidades como:

```text
processamento demorado
retry
grande volume
desacoplamento
múltiplas reações ao mesmo evento
```

Não adicionar filas ou mensageria apenas por antecipação.

---

# 37. Transações

Transações devem ser controladas na camada de aplicação.

Elas devem representar limites claros de um caso de uso.

Evitar transações em controllers.

Também deve ser evitado manter transações de banco abertas durante operações externas demoradas sem necessidade.

---

# 38. Configuração

A aplicação deverá receber configurações através de variáveis de ambiente.

Exemplos de grupos:

```text
aplicação
banco
autenticação
storage
integrações
jobs
observabilidade
```

Secrets nunca devem ser armazenados diretamente no código-fonte.

---

# 39. Logs e Observabilidade

A aplicação deve registrar informações suficientes para operação e diagnóstico.

Exemplos:

```text
inicialização
falhas
integrações
jobs
webhooks
operações relevantes
```

Não registrar:

```text
senhas
tokens
secrets
dados sensíveis sem necessidade
```

Healthchecks também devem estar disponíveis para infraestrutura quando necessário.

---

# 40. Segurança

Princípios mínimos:

```text
senhas armazenadas de forma segura

autenticação e autorização no backend

secrets fora do código

entrada validada

erros sem exposição de detalhes internos

logs sem credenciais

integrações protegidas

arquivos privados protegidos
```

Segurança deve ser aplicada independentemente do comportamento do frontend.

---

# 41. Estrutura Inicial do Repositório

Estrutura conceitual:

```text
backend
 ├── docs
 ├── src
 │   ├── main
 │   │   ├── java
 │   │   └── resources
 │   └── test
 │
 ├── .env.example
 ├── Dockerfile
 └── build file
```

Os detalhes poderão variar conforme a implementação escolhida.

---

# 42. Ordem Inicial de Implementação

Fluxo sugerido:

```text
1. Criar projeto backend.

2. Configurar banco.

3. Configurar migrations.

4. Criar estrutura modular.

5. Criar shared básico.

6. Configurar tratamento de erros.

7. Configurar validação.

8. Configurar segurança.

9. Implementar autenticação.

10. Implementar autorização.

11. Configurar documentação da API.

12. Implementar primeiro domínio.

13. Implementar integrações quando necessárias.

14. Adicionar jobs, storage e eventos conforme necessidade.

15. Evoluir a arquitetura conforme a complexidade real.
```

---

# 43. O que Evitar

Evitar:

```text
Tudo em controllers.

Tudo em services.

Tudo em shared.

Controller acessando repository.

Controller contendo regra de negócio.

Entity exposta diretamente pela API.

Domínio dependendo de SDK externo.

Secrets no código.

Interfaces sem necessidade.

Factories sem necessidade.

Strategies sem variação real.

Mensageria sem necessidade.

Microserviços antes da necessidade.

RuntimeException genérica para qualquer erro.

Regras de negócio dentro de jobs.

Duplicação de regras entre módulos.
```

---

# 44. Princípios Finais

A arquitetura deverá seguir:

```text
Domínio antes de camada técnica.

Regra de negócio próxima do domínio.

Controller simples.

Application coordenando casos de uso.

Infraestrutura isolada.

Integração externa atrás de uma fronteira.

Histórico preservado quando necessário.

Idempotência onde houver repetição possível.

Interfaces apenas quando agregarem valor.

Complexidade apenas quando houver necessidade.
```

---

# 45. Resumo

A arquitetura inicial será:

```text
Java
+
Spring Boot
+
PostgreSQL
+
Monólito Modular
+
Clean Architecture leve
```

A organização principal será:

```text
shared
auth
users
modules de negócio
integrations
dashboard
```

Cada módulo poderá possuir:

```text
api
application
domain
infrastructure
```

O fluxo principal será:

```text
HTTP
 ↓
API
 ↓
Application
 ↓
Domain
 ↓
Repository / Gateway / Provider
 ↓
Infrastructure
```

O backend continuará sendo responsável pelas regras críticas do sistema.

Controllers deverão permanecer simples.

Regras de negócio devem permanecer próximas do domínio.

Infraestrutura e integrações externas devem permanecer isoladas.

Abstrações devem ser introduzidas apenas quando resolverem um problema real.

O objetivo deste documento não é definir cada detalhe da implementação.

Ele estabelece os fundamentos que deverão orientar o desenvolvimento do backend.

Regra final:

```text
Começar simples.

Organizar por domínio.

Separar responsabilidades.

Proteger regras de negócio.

Isolar infraestrutura.

Preservar histórico.

Evoluir somente quando houver necessidade.
```
