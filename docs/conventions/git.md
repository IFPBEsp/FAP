# Convenções de Git

Padrão de commits e de branches do projeto, baseado na especificação [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/).

## Por que padronizar

Um histórico estruturado documenta o desenvolvimento e facilita percorrer a linha do tempo do projeto: fica mais claro o caminho que o software seguiu, como ele evoluiu e o que motivou cada alteração no código.

Isso pesa principalmente na manutenção. Commits padronizados explicam as razões por trás de uma mudança sem depender de procurar o autor, que pode já ter esquecido o próprio código ou nem estar mais na empresa para ajudar.

## Padrão de commits

```text
{TYPE}: {DESCRIPTION}
```

### TYPE

O TYPE indica o tipo de alteração do commit. Antes de commitar, veja em qual deles suas alterações melhor se encaixam.

| Tipo | Quando usar | Exemplos |
| --- | --- | --- |
| `feat` | Adição de uma nova funcionalidade ao código | `feat: implementa fluxo de login` · `feat: adiciona CRUD de Obrigacao` |
| `fix` | Correção de bugs que **já estão em produção** | `fix: corrige validacao no formulario de registro` · `fix: corrige data sendo salva no banco de dados no fuso horario local` |
| `chore` | Tarefas de manutenção e pequenos ajustes | `chore: desabilita botao de submissao enquanto a requisicao esta pendente` · `chore: corrige nome do endpoint do Controller de Licitacao` |
| `refactor` | Refatorações que não adicionam funcionalidade nem corrigem bugs | `refactor: encapsula logica em uma funcao no arquivo Utils` · `refactor: renomeia nome do metodo de Usuario` |
| `docs` | Documentação, em arquivos `.md` ou em comentários de documentação | `docs: atualiza README.md` · `docs: adiciona changelog das mudancas do sonarqube` |
| `style` | Ajustes de formatação | `style: formata arquivo de acordo com o prettier` · `style: adiciona \n no fim do arquivo` |
| `build` | Mudanças nas dependências ou nos scripts de build | `build: ajusta configuracoes de conversao de arquivos typescript para javascript na geracao da build` · `build: atualiza versao do gradle` |
| `ci` | Mudanças na configuração de CI e seus scripts | `ci: atualiza gitlab-ci.yml` · `ci: executa testes unitarios antes do commit` |
| `perf` | Melhoria de performance | `perf: adiciona cache na listagem de registros` · `perf: migra query de ORM para uma query nativa SQL` |
| `test` | Modificações em testes | `test: corrige teste e2e cypress da tela de login` · `test: adiciona testes unitarios ao crud de Licitacao` |

Se estiver difícil identificar o tipo, talvez o commit esteja juntando alterações demais. Verifique se ele pode ser subdividido em commits _atômicos_, o que também torna a classificação muito mais fácil.

### DESCRIPTION

A DESCRIPTION resume as alterações do commit. Comece com um verbo no presente do indicativo, sem gerúndio nem pretérito, e não use caracteres específicos de línguas derivadas do latim (`^` `´` `~`).

Em vez de `feat: adicionando validacao de formulario` ou `feat: adicionada validacao de formulario`, prefira:

```text
feat: adiciona validacao de formulario
```

Para construir a descrição, responda: _o que esse commit faz?_ A resposta completa a frase "esse commit …", e é exatamente o texto que vai na DESCRIPTION.

```text
Esse commit adiciona validação de formulário  → feat: adiciona validacao de formulario
Esse commit atualiza a versão do gradle       → build: atualiza a versao do gradle
```

## Padrão de branches

```text
{ISSUE_NUMBER}-{DESCRIPTION}
```

O ISSUE_NUMBER é o número da sua issue no GitLab. Com ele, ao criar o MR o GitLab faz o vínculo automático com a issue, adicionando o MR como _related merge request_ e melhorando a documentação do desenvolvimento.

A DESCRIPTION usa _kebab-case_ e começa com um verbo no presente do indicativo. Resuma o objetivo da branch em 3 a 5 palavras, de forma sucinta e mais abstrata, mesmo que isso fira a estrutura gramatical da frase. Evite nomes muito longos.

Para a issue "Corrigir os bugs (major e minor) apontados pelo SonarQube [syn4tdf-frontend]", em vez de uma descrição longa e gramaticalmente correta como `9999-corrige-os-bugs-apontados-pelo-sonarqube`, foque nas keywords:

```text
9999-corrige-bugs-sonarqube
```

Vale a mesma pergunta usada nos commits, agora com _o que essa branch faz?_:

```text
Essa branch adiciona uma nova modal de importação
→ 9999-adiciona-modal-importacao

Essa branch corrige o armazenamento do fuso horário das datas
de execução do self-test no banco de dados
→ 9999-corrige-timezone-self-test
```

## Referências

- [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/)
- [Angular: Commit Message Guidelines](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)
- [Should I use past or present tense in git commit messages?](https://stackoverflow.com/questions/3580013/should-i-use-past-or-present-tense-in-git-commit-messages)
- [Atomic git commits](https://www.aleksandrhovhannisyan.com/blog/atomic-git-commits/#atomic-commits-and-the-single-responsibility-principle)
