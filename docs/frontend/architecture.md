# Arquitetura Genérica de Frontend

## 1. Visão Geral

Este documento descreve uma arquitetura inicial e reutilizável para o frontend de **sistemas de informação**.

A proposta é servir como documentação arquitetural inicial para novos projetos, antes da implementação das funcionalidades específicas de negócio.

O frontend será desenvolvido utilizando:

```text
Next.js App Router
React
TypeScript
Tailwind CSS
shadcn/ui
```

seguindo:

```text
Feature-Based Architecture
+
BFF
+
Server-first Rendering
```

A arquitetura deverá priorizar:

- Organização por domínio/feature.
- Baixo acoplamento.
- Segurança.
- Renderização server-side quando apropriada.
- Estado client-side mínimo.
- Contratos sincronizados com o backend.
- Reutilização controlada.
- Evolução incremental.
- Facilidade de manutenção humana e assistida por IA.
- Deploy simples com Docker.

Princípio central:

```text
Server primeiro.

Client quando necessário.

Estado local antes de global.

URL antes de store para filtros e navegação.

Backend como fonte da verdade.

Shared apenas para código realmente compartilhado.

Abstrações apenas quando houver benefício real.
```

---

# 2. Estilo Arquitetural

A arquitetura escolhida será:

```text
Feature-Based Architecture
+
BFF
+
Server-first Rendering
```

Isso significa que:

```text
O código será organizado por domínio.

O navegador não acessará diretamente o backend
em fluxos autenticados.

O Next.js atuará como Backend for Frontend.

Server Components serão a escolha padrão.

Client Components serão usados quando houver
interatividade real.

Server Actions serão utilizadas para mutations
quando fizer sentido.

TanStack Query será reservado para estados
remotos altamente interativos.

A URL será utilizada para estados navegáveis.

Estado global será mínimo.
```

---

# 3. Stack Principal

Stack base:

```text
Next.js
React
TypeScript
Tailwind CSS
shadcn/ui
React Hook Form
Zod
TanStack Query
TanStack Table
nuqs
Zustand
Orval
Sonner
ESLint
Prettier
Docker
```

Dependências opcionais:

```text
Tiptap
Recharts
date-fns
React Dropzone
Sentry
OpenTelemetry
Playwright
Vitest
Testing Library
MSW
```

Adicionar dependências apenas quando houver necessidade concreta.

---

# 4. Decisões Técnicas

| Área              | Decisão                               |
| ----------------- | ------------------------------------- |
| Framework         | Next.js                               |
| Router            | App Router                            |
| Linguagem         | TypeScript                            |
| Arquitetura       | Feature-Based                         |
| Renderização      | Server-first                          |
| UI                | shadcn/ui                             |
| Estilização       | Tailwind CSS                          |
| Formulários       | React Hook Form                       |
| Validação         | Zod                                   |
| Cache client-side | TanStack Query                        |
| Tabelas           | TanStack Table                        |
| Estado de URL     | nuqs                                  |
| Estado global     | Zustand                               |
| API Client        | Orval                                 |
| Contratos         | OpenAPI                               |
| Autenticação      | Cookies httpOnly                      |
| BFF               | Route Handlers                        |
| Mutations         | Server Actions quando aplicável       |
| Feedback          | Sonner                                |
| Deploy            | Docker                                |
| Qualidade         | ESLint + Prettier + TypeScript strict |

---

# 5. Princípios da Arquitetura

A arquitetura seguirá alguns princípios principais.

## Organização por domínio

Código relacionado a uma mesma funcionalidade deve permanecer próximo.

## Server-first

Server Components são a escolha padrão.

## Client apenas quando necessário

Adicionar `"use client"` somente quando existir necessidade real.

## Backend como fonte da verdade

Regras críticas de negócio e segurança continuam no backend.

## Estado global mínimo

Nem todo estado precisa de Zustand.

## URL como estado navegável

Filtros, paginação, busca e abas compartilháveis devem preferir a URL.

## BFF como fronteira

O navegador não deve conhecer detalhes internos do backend autenticado.

## Código gerado não é código de negócio

Clientes gerados por OpenAPI não devem receber lógica manual.

## Shared não é depósito

Código de domínio não deve ser movido para `shared` apenas para "organizar".

---

# 6. Estrutura Geral

Estrutura sugerida:

```text
src
 ├── app
 │   ├── (auth)
 │   │   └── login
 │   │       └── page.tsx
 │   │
 │   ├── (app)
 │   │   ├── layout.tsx
 │   │   ├── dashboard
 │   │   ├── module-a
 │   │   ├── module-b
 │   │   └── settings
 │   │
 │   ├── api
 │   │   ├── auth
 │   │   ├── module-a
 │   │   ├── module-b
 │   │   └── integrations
 │   │
 │   ├── forbidden.tsx
 │   ├── unauthorized.tsx
 │   ├── not-found.tsx
 │   ├── error.tsx
 │   └── loading.tsx
 │
 ├── features
 │   ├── auth
 │   ├── users
 │   ├── dashboard
 │   ├── module-a
 │   └── module-b
 │
 ├── shared
 │   ├── components
 │   ├── hooks
 │   ├── lib
 │   ├── schemas
 │   ├── types
 │   ├── utils
 │   ├── constants
 │   └── stores
 │
 ├── generated
 │   └── api
 │
 └── config
```

`module-a` e `module-b` representam domínios específicos do sistema.

Exemplos:

```text
orders
inventory
customers
contracts
projects
students
appointments
documents
payments
assets
employees
```

---

# 7. Responsabilidade da Pasta `app`

A pasta `app` representa:

```text
Rotas
Layouts
Route Groups
Route Handlers
Pages
Loading states
Error boundaries
Not found
Layouts aninhados
```

Ela não deve concentrar toda a regra das funcionalidades.

A maior parte da implementação de domínio deve permanecer em:

```text
features
```

---

# 8. Route Groups

Utilizar Route Groups para separar contextos.

Exemplo:

```text
app
 ├── (auth)
 └── (app)
```

Possíveis grupos:

```text
(auth)
(app)
(public)
(onboarding)
```

Route Groups organizam a aplicação sem necessariamente alterar a URL.

---

# 9. Estrutura Interna de uma Feature

Exemplo:

```text
features/products
 ├── api
 │   ├── products.actions.ts
 │   ├── products.queries.ts
 │   └── products.keys.ts
 │
 ├── components
 │   ├── product-form.tsx
 │   ├── products-table.tsx
 │   └── product-details-card.tsx
 │
 ├── hooks
 │   ├── use-products-filters.ts
 │   └── use-product-permissions.ts
 │
 ├── schemas
 │   ├── create-product.schema.ts
 │   └── update-product.schema.ts
 │
 ├── types
 │   └── product.types.ts
 │
 ├── utils
 │   └── product-formatters.ts
 │
 └── constants
     └── product.constants.ts
```

Nem toda feature precisa ter todas essas pastas.

Criar somente quando necessário.

---

# 10. Regra de `features`

A pasta:

```text
features
```

deve conter código específico dos domínios.

Exemplos:

```text
ProductForm
ProductsTable
OrderStatusBadge
PaymentFilters
DocumentActions
```

Se uma implementação pertence claramente a um domínio, ela deve permanecer nele.

---

# 11. Regra de `shared`

`shared` deve conter apenas código verdadeiramente reutilizável entre features.

Exemplos aceitáveis:

```text
Button
DataTable
Pagination
EmptyState
PageHeader
ConfirmDialog
DatePicker
CurrencyInput
useDebounce
ApiError
formatCurrency
```

Evitar:

```text
shared/components/customer-form.tsx
shared/utils/order-utils.ts
shared/hooks/use-payment.ts
```

quando esses elementos pertencem claramente a uma feature.

---

# 12. BFF

O Next.js atuará como:

```text
Backend for Frontend
```

Fluxo:

```text
Browser
   ↓
Next.js
   ↓
Route Handler / Server Action
   ↓
Backend API
   ↓
Database / Storage / Integrations
```

O browser não deve chamar diretamente o backend autenticado.

---

# 13. Responsabilidades do BFF

O BFF será responsável por:

- Ler cookies httpOnly.
- Adicionar autenticação nas chamadas internas.
- Executar refresh token.
- Encapsular detalhes do backend.
- Padronizar respostas quando necessário.
- Padronizar erros.
- Intermediar uploads protegidos.
- Intermediar downloads protegidos.
- Ocultar tokens do navegador.
- Isolar a URL interna da API.
- Adaptar contratos quando necessário.

---

# 14. Route Handlers

Estrutura:

```text
app/api/products/route.ts

app/api/products/[id]/route.ts

app/api/auth/login/route.ts

app/api/auth/logout/route.ts
```

Exemplo conceitual:

```text
Browser
 ↓
GET /api/products
 ↓
Route Handler
 ↓
GET backend/products
```

Route Handlers podem expor:

```text
GET
POST
PUT
PATCH
DELETE
```

---

# 15. BFF não é Segundo Backend de Negócio

O BFF não deve duplicar regras do backend.

Evitar:

```text
Browser
 ↓
BFF
 ↓
Regra de negócio duplicada
 ↓
Backend
 ↓
Outra regra de negócio
```

Preferir:

```text
Browser
 ↓
BFF
 ↓
Autenticação / adaptação
 ↓
Backend
 ↓
Regra de negócio
```

O BFF representa uma camada de interface e segurança para o frontend.

---

# 16. Autenticação

Estratégia:

```text
JWT
+
Refresh Token
+
Cookies httpOnly
+
BFF
```

O browser não deve acessar diretamente os tokens.

---

# 17. Cookies de Autenticação

Cookies possíveis:

```text
access_token
refresh_token
```

Configuração:

```text
httpOnly: true
secure: true em produção
sameSite: lax ou strict
path: /
```

Não armazenar tokens em:

```text
localStorage
sessionStorage
Zustand
React state
```

---

# 18. Fluxo de Login

```text
1. Usuário abre /login.

2. Envia credenciais.

3. Server Action ou Route Handler recebe os dados.

4. BFF chama o backend.

5. Backend valida credenciais.

6. Backend retorna tokens e dados do usuário.

7. BFF grava os tokens em cookies httpOnly.

8. Usuário é redirecionado para a aplicação.

9. Dados do usuário autenticado são carregados.
```

---

# 19. Refresh Token

Fluxo:

```text
Request
 ↓
Backend responde 401
 ↓
BFF verifica refresh token
 ↓
POST /auth/refresh-token
 ↓
Novo access token
 ↓
Cookie atualizado
 ↓
Request original refeita
```

Caso o refresh falhe:

```text
limpar cookies
+
invalidar sessão local
+
redirecionar para /login
```

---

# 20. Logout

Fluxo:

```text
Usuário
 ↓
Logout
 ↓
BFF
 ↓
Backend /auth/logout
 ↓
Cookies removidos
 ↓
Redirect /login
```

Mesmo que o backend falhe, os cookies locais devem ser tratados adequadamente.

---

# 21. Usuário Autenticado

A aplicação poderá obter:

```text
GET /auth/me
```

com dados como:

```text
id
name
email
role
permissions
```

Esses dados poderão alimentar:

- Menu.
- Header.
- Controle visual de permissões.
- Estado mínimo da sessão.
- Informações da conta.

---

# 22. Autorização

A autorização real continua no backend.

O frontend possui responsabilidade visual.

Estratégia:

```text
Backend:
protege recurso.

Frontend:
esconde ação sem permissão
+
impede navegação normal
+
apresenta 403.
```

---

# 23. Helper de Permissões

Exemplo:

```ts
can("PRODUCTS_CREATE");
can("PRODUCTS_UPDATE");
can("USERS_VIEW");
```

Uso:

```tsx
{
  can("PRODUCTS_CREATE") && <Button>Novo produto</Button>;
}
```

---

# 24. Menu por Permissão

Exemplo:

```ts
const menuItems = [
  {
    label: "Produtos",
    href: "/products",
    permission: "PRODUCTS_VIEW",
  },
  {
    label: "Pedidos",
    href: "/orders",
    permission: "ORDERS_VIEW",
  },
];
```

O menu é filtrado conforme as permissões.

---

# 25. Proteção de Rotas

Se o usuário não estiver autenticado:

```text
redirect /login
```

Se estiver autenticado, mas não possuir permissão:

```text
403
```

A proteção visual nunca substitui a autorização no backend.

---

# 26. Server Components

Server Components devem ser priorizados.

Usar para:

- Páginas de listagem.
- Páginas de detalhes.
- Dashboards.
- Dados iniciais.
- Dados autenticados.
- Conteúdo sem interatividade no browser.
- Layouts.
- Breadcrumbs.
- Navegação baseada na sessão.

Exemplo:

```tsx
export default async function ProductsPage() {
  const products = await getProducts();

  return <ProductsTable data={products} />;
}
```

---

# 27. Client Components

Usar quando existir necessidade real de:

- `useState`.
- `useEffect`.
- React Hook Form.
- APIs do navegador.
- Eventos interativos.
- Zustand.
- TanStack Query.
- Modais.
- Upload.
- Drag-and-drop.
- Clipboard.
- Componentes altamente interativos.

Evitar colocar:

```tsx
"use client";
```

em páginas inteiras sem necessidade.

---

# 28. Regra de Client Boundary

Preferir:

```text
Server Page
   ↓
Client Component pequeno
```

em vez de:

```text
Client Page
   ↓
Tudo client-side
```

Exemplo:

```text
ProductPage
 ├── ProductHeader        Server
 ├── ProductSummary       Server
 └── ProductActions       Client
```

---

# 29. Server Actions

Server Actions poderão ser usadas para:

```text
Create
Update
Delete
Change status
Form submission
Business command
Protected action
```

Exemplo:

```ts
"use server";

export async function createProductAction(input: CreateProductInput) {
  // validar sessão
  // validar permissão
  // chamar backend
  // tratar resposta
}
```

---

# 30. Segurança em Server Actions

Cada Server Action deve ser tratada como uma entrada protegida.

Não assumir que:

```text
botão escondido = action segura
```

A action deve validar quando necessário:

```text
autenticação
permissão
input
```

---

# 31. Server Actions x Route Handlers

Usar Server Action quando o fluxo estiver fortemente associado à aplicação React.

Exemplo:

```text
Submit de formulário
Alteração de status
Mutation de tela
```

Usar Route Handler quando for necessário um endpoint HTTP explícito.

Exemplo:

```text
BFF REST
download
upload
callback
proxy
endpoint consumido por client-side query
```

---

# 32. TanStack Query

Utilizar TanStack Query quando houver necessidade real de cache e sincronização client-side.

Casos:

```text
Telas muito interativas
Refetch
Polling
Dados por modal
Abas independentes
Infinite scroll
Mutations client-side
Optimistic update
```

Não utilizar automaticamente para toda requisição.

---

# 33. Regra de Dados

Escolha sugerida:

```text
Dados iniciais e páginas simples
→ Server Component

Mutation simples
→ Server Action

Tela altamente interativa
→ TanStack Query
```

---

# 34. Query Keys

Centralizar:

```text
products.keys.ts
orders.keys.ts
users.keys.ts
```

Exemplo:

```ts
export const productKeys = {
  all: ["products"] as const,

  list: (params: ProductFilters) =>
    [...productKeys.all, "list", params] as const,

  detail: (id: string) => [...productKeys.all, "detail", id] as const,
};
```

---

# 35. URL State

Estados navegáveis devem preferir a URL.

Exemplo:

```text
/products?page=0&size=20&search=mouse&status=ACTIVE
```

Utilizar `nuqs`.

Aplicações:

```text
page
size
search
sort
filters
tabs
date ranges
```

---

# 36. Vantagens do Estado em URL

Permite:

- Refresh sem perder filtros.
- Compartilhamento de link.
- Navegação back/forward.
- Bookmark.
- Menos estado global.
- Menos sincronização manual.

---

# 37. O que Não Deve Ficar em Zustand

Não usar Zustand para:

```text
filtros
paginação
formulários
dados da API
cache de requisições
estado temporário de modal simples
```

Esses casos possuem ferramentas melhores.

---

# 38. Zustand

Utilizar apenas para estado realmente global.

Exemplos:

```text
Usuário autenticado
Permissões
Tema
Estado reduzido da sidebar
Preferências globais
```

Mesmo nesses casos, avaliar se React Context ou Server Components já resolvem o problema.

---

# 39. Formulários

Padrão:

```text
React Hook Form
+
Zod
```

Exemplo:

```ts
const schema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});
```

---

# 40. Estrutura de Formulário

```text
features/products
 ├── schemas
 │   ├── create-product.schema.ts
 │   └── update-product.schema.ts
 │
 ├── components
 │   └── product-form.tsx
 │
 └── api
     ├── create-product.action.ts
     └── update-product.action.ts
```

---

# 41. Validação

A validação frontend melhora experiência do usuário.

Ela não substitui o backend.

Fluxo:

```text
Zod
 ↓
Server Action / BFF
 ↓
Backend Validation
 ↓
Domain Rules
```

---

# 42. Erros de Formulário

Erros de campos vindos do backend devem ser convertidos para:

```text
form.setError(...)
```

quando possível.

Exemplo:

```text
email → Email já cadastrado
name → Nome obrigatório
```

Erros gerais devem utilizar feedback global.

---

# 43. Feedback

Utilizar:

```text
Sonner
```

para:

```text
sucesso
erro de negócio
erro de integração
aviso
```

Evitar excesso de toast para informações permanentes.

---

# 44. Tratamento de Erros

Tipo padrão:

```ts
type ApiError = {
  type: string;
  title: string;
  status: number;
  detail: string;
  fields?: Record<string, string>;
};
```

---

# 45. Estratégia de Exibição de Erros

```text
Validation Error
→ campo

Business Error
→ toast ou mensagem contextual

401
→ limpar sessão + login

403
→ forbidden

404
→ not-found

Unexpected Error
→ error boundary + log
```

---

# 46. Error Boundaries

Utilizar arquivos do App Router:

```text
error.tsx
not-found.tsx
loading.tsx
```

Quando necessário:

```text
forbidden.tsx
unauthorized.tsx
```

Estados de erro devem ser consistentes em toda aplicação.

---

# 47. Loading

Utilizar:

```text
loading.tsx
Skeleton
Suspense
```

Preferir skeleton compatível com o layout final.

Evitar loading global em toda navegação quando carregamentos locais forem suficientes.

---

# 48. Empty States

Listagens vazias devem possuir estados próprios.

Exemplo:

```text
Nenhum produto encontrado.

[Adicionar produto]
```

Diferenciar:

```text
Nenhum registro cadastrado
```

de:

```text
Nenhum resultado para os filtros aplicados
```

---

# 49. API Client

A API backend deve disponibilizar OpenAPI.

O frontend utilizará:

```text
Orval
```

para gerar:

```text
Types
Schemas
HTTP Clients
React Query Hooks
Contracts
```

---

# 50. Código Gerado

Estrutura:

```text
src/generated/api
 ├── schemas
 ├── auth
 ├── users
 ├── products
 └── orders
```

Regra:

```text
Nunca editar código gerado manualmente.
```

---

# 51. Customizações de API

Customizações devem ficar fora de `generated`.

Exemplo:

```text
features/products/api
shared/lib/api
```

Fluxo:

```text
OpenAPI
 ↓
Orval
 ↓
generated/api
 ↓
feature/api
 ↓
UI
```

---

# 52. Regeneração dos Contratos

Deve existir comando padronizado.

Exemplo:

```json
{
  "scripts": {
    "api:generate": "orval"
  }
}
```

Mudanças no contrato backend devem poder gerar atualização previsível no frontend.

---

# 53. BFF Client

Criar helpers para chamadas server-side.

Exemplo:

```text
shared/lib/api
 ├── backend-client.ts
 ├── authenticated-fetch.ts
 ├── refresh-token.ts
 └── api-error.ts
```

Responsabilidades:

```text
base URL
headers
auth
refresh
erro
correlation id
```

---

# 54. Listagens

Listagens principais devem considerar:

```text
Server-side pagination
Sorting
Filtering
Search
Debounce
URL state
Empty state
Loading
Permissions
```

---

# 55. TanStack Table

TanStack Table ficará responsável principalmente por comportamento de tabela.

Exemplos:

```text
columns
sorting UI
row selection
column visibility
renderização
```

Ela não precisa controlar sozinha todo estado remoto.

---

# 56. Paginação

Contrato genérico:

```ts
type PageResponse<T> = {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
};
```

Exemplo de URL:

```text
?page=0&size=20
```

---

# 57. Busca

Usar debounce em buscas textuais.

Exemplo:

```text
search input
 ↓
debounce
 ↓
URL
 ↓
Server Request
```

Evitar chamada ao backend a cada caractere sem controle.

---

# 58. Ordenação

A ordenação também deve ser representável na URL.

Exemplo:

```text
?sort=name,asc
```

ou:

```text
?sortBy=name&direction=asc
```

O padrão deve acompanhar o backend.

---

# 59. Filtros

Filtros devem possuir:

```text
valor inicial vindo da URL
estado visual
serialização previsível
botão limpar quando necessário
```

Filtros complexos devem continuar compartilháveis por link quando possível.

---

# 60. Abas

Quando uma aba representa uma subárea relevante da aplicação, preferir nested routing.

Exemplo:

```text
/products/[id]/details
/products/[id]/history
/products/[id]/documents
```

em vez de:

```text
/products/[id]?tab=documents
```

quando cada aba possuir conteúdo e carregamento próprios.

---

# 61. Nested Routing

Benefícios:

- Deep link.
- Back/forward natural.
- Loading independente.
- Error boundary independente.
- Organização de código.
- Menos estado manual.

---

# 62. Layout Administrativo

Estrutura típica:

```text
Sidebar
Header
Breadcrumbs
Main Content
User Menu
Theme Toggle
Logout
```

Também deve possuir:

```text
loading states
error states
empty states
responsive behavior
```

---

# 63. Sidebar

Menu pode conter:

```text
Dashboard
Domínio A
Domínio B
Usuários
Configurações
```

Renderizado conforme permissões.

A sidebar deve ser responsiva.

Em mobile:

```text
Drawer / Sheet
```

pode substituir o comportamento desktop.

---

# 64. Breadcrumbs

Breadcrumbs devem preferir ser derivados da rota ou de configuração de navegação.

Exemplo:

```text
Produtos
>
Produto XPTO
>
Editar
```

Evitar definir breadcrumbs manualmente em todas as páginas se existir padrão reutilizável.

---

# 65. Tema

Suporte:

```text
light
dark
system
```

Tema é uma das poucas preferências aceitáveis em estado global ou persistência local.

---

# 66. Design System

`shadcn/ui` será utilizado como base.

Isso não significa utilizar componentes sem padronização.

Definir padrões internos para:

```text
Button
Input
Select
Dialog
Sheet
Table
Card
Badge
Alert
Form
DatePicker
Combobox
```

---

# 67. Componentes Compartilhados

Exemplos:

```text
PageHeader
PageContainer
DataTable
Pagination
SearchInput
EmptyState
LoadingSkeleton
ConfirmDialog
PermissionGuard
StatusBadge
DateRangePicker
```

Somente promover um componente para `shared` quando houver reutilização real ou padrão global claro.

---

# 68. Componentes de Feature

Exemplos:

```text
ProductForm
OrderItemsTable
CustomerAddressForm
PaymentStatusCard
```

Mesmo que utilizem componentes `shared`, continuam pertencendo às respectivas features.

---

# 69. Configuração de Tabelas

Evitar uma tabela universal extremamente abstrata.

Preferir:

```text
DataTable genérico
+
columns específicas da feature
+
toolbar específica da feature
```

Exemplo:

```text
shared/DataTable
          ↑
ProductsTable
```

---

# 70. Uploads

Fluxo:

```text
Browser
 ↓
Next.js BFF
 ↓
Backend
 ↓
Storage
```

Aplicações:

```text
Documentos
Imagens
Logos
Anexos
Importações
```

---

# 71. Upload Client Component

Uploads normalmente exigirão Client Components por dependerem de:

```text
File API
drag-and-drop
progress
preview
```

A submissão continua passando pelo BFF.

---

# 72. Validação de Upload

Validar no frontend:

```text
tipo
tamanho
quantidade
```

Mas repetir obrigatoriamente as validações relevantes no backend.

---

# 73. Downloads

Downloads protegidos devem passar pelo BFF.

Fluxo:

```text
Browser
 ↓
BFF
 ↓
Backend
 ↓
Storage
```

Especialmente para arquivos:

```text
privados
jurídicos
financeiros
pessoais
```

---

# 74. Preview de Arquivos

Quando apropriado:

```text
PDF → iframe / viewer
Imagem → preview
Texto → visualização
```

Sempre respeitando autorização.

---

# 75. Estado de Processos Externos

Quando o backend possui processos externos com estado:

```text
PENDING
PROCESSING
COMPLETED
FAILED
```

a interface deve apresentar claramente esse ciclo.

Exemplo:

```text
StatusBadge
LastUpdatedAt
RetryAction
RefreshStatusAction
```

Não criar estados frontend diferentes sem necessidade.

---

# 76. Status do Backend como Fonte da Verdade

O frontend não deve assumir que uma operação externa terminou apenas porque a chamada inicial retornou sucesso.

Exemplo:

```text
Usuário envia documento
 ↓
API responde "request created"
 ↓
UI mostra SENT / PROCESSING
 ↓
Backend recebe webhook
 ↓
Status vira COMPLETED
```

A interface reflete o estado persistido pelo backend.

---

# 77. Polling

Polling não deve ser habilitado automaticamente.

Usar quando:

- O processo exige atualização frequente.
- WebSocket/SSE não estão disponíveis.
- Atualização manual não é suficiente.

Caso contrário:

```text
Refresh manual
```

pode ser suficiente.

---

# 78. Confirmações

Ações destrutivas ou relevantes devem possuir confirmação.

Exemplos:

```text
Excluir
Cancelar
Reabrir
Enviar
Renegociar
Substituir
```

Utilizar:

```text
ConfirmDialog
```

---

# 79. Ações Irreversíveis

Deixar visualmente explícito:

```text
ação
consequência
recurso afetado
```

Evitar confirmações genéricas:

```text
Tem certeza?
```

Preferir:

```text
Cancelar este pedido impedirá novas alterações.
Deseja continuar?
```

---

# 80. Dashboard

Dashboards devem priorizar Server Components.

Fluxo:

```text
Page Server Component
 ↓
BFF
 ↓
Dashboard Endpoint
```

Componentes interativos podem ser client-side.

---

# 81. Filtros de Dashboard

Exemplos:

```text
Hoje
Últimos 7 dias
Últimos 30 dias
Este mês
Personalizado
```

O estado do período pode permanecer na URL.

Exemplo:

```text
/dashboard?startDate=...&endDate=...
```

---

# 82. Gráficos

Gráficos são Client Components.

A página principal ainda pode permanecer Server Component.

Exemplo:

```text
DashboardPage          Server
 ├── SummaryCards      Server
 ├── RecentItems       Server
 └── RevenueChart      Client
```

---

# 83. Formatação de Dados

Formatadores específicos de domínio devem permanecer na feature.

Exemplo:

```text
features/payments/utils/payment-formatters.ts
```

Formatadores universais podem ir para:

```text
shared/utils
```

Exemplos:

```text
formatDate
formatCurrency
formatPercentage
```

---

# 84. Datas

Padronizar tratamento de datas.

Definir claramente:

```text
UTC no backend?
Timezone local?
Formato de envio?
Formato de exibição?
```

A camada de UI deve formatar, mas não alterar semanticamente a data.

---

# 85. Máscaras

Máscaras são responsabilidade de apresentação.

Exemplos:

```text
telefone
documento
CEP
moeda
percentual
```

O valor enviado ao backend deve seguir o contrato definido.

Evitar acoplar valor mascarado com valor persistido.

---

# 86. Responsividade

O sistema deve funcionar em:

```text
Desktop
Tablet
Mobile
```

Mesmo quando o foco principal for painel administrativo.

Tabelas grandes podem utilizar:

```text
scroll horizontal
cards responsivos
columns adaptativas
```

dependendo do caso.

---

# 87. Acessibilidade

Componentes devem preservar:

- Navegação por teclado.
- Labels.
- Focus states.
- Contraste.
- `aria-*` quando necessário.
- Estrutura semântica.

shadcn/ui e Radix facilitam isso, mas não eliminam a responsabilidade da implementação.

---

# 88. Segurança

Regras principais:

```text
Tokens em cookies httpOnly
Backend nunca acessado diretamente pelo browser autenticado
Permissões validadas no servidor
CSRF tratado quando necessário
CSP configurada
Sem secrets em código client-side
Sem tokens em logs
```

---

# 89. Variáveis Públicas

Qualquer variável:

```text
NEXT_PUBLIC_*
```

deve ser considerada visível ao navegador.

Nunca colocar:

```text
API_SECRET
JWT_SECRET
PRIVATE_TOKEN
REFRESH_TOKEN_SECRET
```

em variáveis públicas.

---

# 90. API Interna

Preferir:

```env
API_INTERNAL_URL=https://api.example.com
```

e evitar, quando o browser não precisa da API:

```env
NEXT_PUBLIC_API_URL=https://api.example.com
```

---

# 91. CSRF

Como autenticação usa cookies, avaliar proteção CSRF para operações sensíveis.

Possibilidades dependem da infraestrutura:

```text
SameSite
Origin validation
CSRF token
BFF-only mutations
```

A estratégia deve ser documentada conforme o projeto.

---

# 92. Content Security Policy

Configurar CSP quando apropriado.

Exemplo de recursos que exigem atenção:

```text
scripts
images
iframes
fonts
external editors
analytics
```

Evitar políticas excessivamente permissivas.

---

# 93. Dados Sensíveis

Informações sensíveis podem ser parcialmente mascaradas.

Exemplos:

```text
***.***.***-12
**** **** **** 1234
jo***@email.com
```

O nível de proteção depende do domínio.

---

# 94. Logs Frontend

Não registrar:

```text
JWT
Refresh token
Passwords
Secrets
Dados sensíveis completos
```

Logs técnicos devem evitar exposição desnecessária.

---

# 95. Error Tracking

Pode ser adicionado futuramente:

```text
Sentry
OpenTelemetry
APM
```

Sempre revisar informações enviadas para terceiros.

---

# 96. Deploy

Deploy padrão:

```text
Docker
+
VPS / Cloud
```

Build:

```text
next build
```

Configuração:

```text
output: standalone
```

---

# 97. Docker

Estrutura:

```text
Dockerfile
docker-compose.yml
.env.example
```

O container deverá receber configurações por ambiente.

---

# 98. Variáveis de Ambiente

Exemplo:

```env
NEXT_PUBLIC_APP_URL=https://app.example.com

API_INTERNAL_URL=https://api.example.com

AUTH_ACCESS_COOKIE_NAME=access_token
AUTH_REFRESH_COOKIE_NAME=refresh_token

NODE_ENV=production
```

---

# 99. Ambientes

Sugestão:

```text
local
development
staging
production
```

O comportamento da aplicação não deve depender de valores hardcoded.

---

# 100. Qualidade de Código

Utilizar:

```text
ESLint
Prettier
TypeScript strict
```

O TypeScript deve funcionar preferencialmente com:

```json
{
  "strict": true
}
```

Evitar uso indiscriminado de:

```ts
any;
```

---

# 101. Tipos

Preferir tipos derivados dos contratos gerados.

Evitar redefinir manualmente:

```ts
type Product = ...
```

quando o Orval já gerou o contrato equivalente.

Criar tipos locais apenas quando representarem:

```text
estado de UI
view model
form state
adaptação específica
```

---

# 102. Naming

Arquivos de componentes:

```text
product-form.tsx
products-table.tsx
product-details-card.tsx
```

Schemas:

```text
create-product.schema.ts
update-product.schema.ts
```

Actions:

```text
create-product.action.ts
update-product.action.ts
delete-product.action.ts
```

Query Keys:

```text
products.keys.ts
orders.keys.ts
```

Hooks:

```text
use-products-filters.ts
use-product-permissions.ts
```

---

# 103. Componentes

Componentes React:

```tsx
ProductForm;
ProductsTable;
ProductDetailsCard;
```

Arquivos:

```text
product-form.tsx
products-table.tsx
product-details-card.tsx
```

---

# 104. Hooks

Hooks devem possuir responsabilidade clara.

Evitar:

```text
useProductEverything
useGlobalStuff
```

Preferir:

```text
useProductFilters
useProductPermissions
useProductSelection
```

---

# 105. Utils

Não criar `utils.ts` gigantes.

Preferir:

```text
currency.ts
date.ts
string.ts
product-formatters.ts
```

Quando o utilitário for específico de uma feature, manter na feature.

---

# 106. Constants

Constants globais:

```text
shared/constants
```

Constants específicas:

```text
features/products/constants
```

Não centralizar todos os valores do projeto em um único arquivo.

---

# 107. Config

Configurações da aplicação poderão ficar em:

```text
src/config
```

Exemplos:

```text
app.config.ts
auth.config.ts
navigation.config.ts
```

---

# 108. Navegação

Configuração de menu pode ser centralizada:

```ts
type NavigationItem = {
  label: string;
  href: string;
  permission?: string;
};
```

Isso facilita:

```text
sidebar
breadcrumbs
permission filtering
```

---

# 109. Rotas

Estrutura de uma feature:

```text
/products
/products/new
/products/[id]
/products/[id]/edit
```

Quando existirem subáreas:

```text
/products/[id]/details
/products/[id]/history
/products/[id]/files
```

---

# 110. Quando Criar uma Página

Criar uma rota quando a informação:

- Precisa de URL própria.
- Precisa ser compartilhável.
- Tem navegação própria.
- Possui ciclo de carregamento independente.
- Representa uma tela real do sistema.

Não transformar toda interação em modal.

---

# 111. Quando Usar Modal

Modal funciona bem para:

```text
Confirmação
Formulário curto
Ação contextual
Preview rápido
Seleção auxiliar
```

Evitar modal para fluxos longos ou complexos.

---

# 112. Criar ou Editar

Para formulários maiores:

```text
/products/new
/products/[id]/edit
```

é geralmente mais previsível que um modal.

Para operações pequenas, modal pode ser adequado.

---

# 113. Após Criação

Padrão sugerido:

```text
Create
 ↓
Success
 ↓
Redirect para detalhes
```

Isso permite continuar trabalhando no recurso criado.

---

# 114. Após Edição

Padrão sugerido:

```text
Update
 ↓
Success toast
 ↓
Permanecer na página
```

ou redirecionar para detalhes conforme o fluxo.

---

# 115. Estado de Loading de Ações

Botões de mutation devem ter estado:

```text
disabled
+
loading
```

para reduzir cliques duplicados.

Isso não substitui idempotência backend.

---

# 116. Double Submit

Frontend deve dificultar duplo envio:

```text
disable submit
loading
```

Mas operações críticas precisam ser seguras também no backend.

---

# 117. Processos Assíncronos

Se uma ação iniciar processamento assíncrono:

```text
STARTED
PROCESSING
COMPLETED
FAILED
```

a interface deve comunicar claramente que a ação foi aceita, mas ainda não finalizada.

---

# 118. Refresh Manual

Quando não houver polling:

```text
Atualizar status
```

pode ser uma ação explícita.

Exemplo:

```text
[Atualizar status]
```

A página pode refazer a consulta sem exigir reload completo.

---

# 119. Estado Vindo do Backend

Não duplicar máquinas de estado no frontend.

Se o backend retorna:

```text
PENDING
PROCESSING
COMPLETED
FAILED
```

o frontend deve mapear isso principalmente para apresentação.

Exemplo:

```text
PENDING → badge
FAILED → badge vermelho + ação permitida
```

---

# 120. Permissões por Ação

A interface pode controlar:

```text
visualizar
criar
editar
excluir
aprovar
cancelar
baixar
reenviar
```

Exemplo:

```tsx
<PermissionGuard permission="PRODUCTS_UPDATE">
  <Button>Editar</Button>
</PermissionGuard>
```

---

# 121. Permission Guard

Pode existir componente genérico:

```tsx
<PermissionGuard permission="PRODUCTS_CREATE" fallback={null}>
  <CreateButton />
</PermissionGuard>
```

O helper não substitui proteção server-side.

---

# 122. Server-side Permission Check

Páginas sensíveis podem validar permissões antes da renderização.

Fluxo:

```text
Page
 ↓
Session
 ↓
Permission check
 ↓
Render
```

ou:

```text
forbidden()
```

---

# 123. Autorização em Server Actions

Toda mutation protegida deve verificar:

```text
sessão
+
permissão
```

no servidor.

Não confiar no estado vindo do Client Component.

---

# 124. API Error Mapper

Centralizar transformação de erros.

Exemplo:

```text
Backend ProblemDetail
        ↓
mapApiError()
        ↓
ApiError
        ↓
UI
```

Isso evita tratamento diferente em cada feature.

---

# 125. HTTP Client

Centralizar comportamento como:

```text
base URL
headers
timeouts
auth
refresh
error mapping
```

Não espalhar chamadas `fetch` configuradas manualmente por todo projeto.

---

# 126. Correlation ID

Quando o backend utilizar correlation ID, o BFF pode propagá-lo.

Fluxo:

```text
Browser request
 ↓
Next BFF
 ↓
X-Correlation-Id
 ↓
Backend
```

Isso facilita observabilidade.

---

# 127. Timeouts

Chamadas externas do BFF devem considerar timeout.

Evitar requests que permanecem indefinidamente aguardando o backend.

Erros de timeout devem gerar feedback adequado.

---

# 128. Retry

Frontend não deve refazer automaticamente mutations sem avaliar segurança.

Retry automático é mais aceitável para:

```text
GET
```

e deve ser usado com cuidado para:

```text
POST
PATCH
DELETE
```

---

# 129. Cache

Não adicionar cache sem entender os requisitos.

Possíveis níveis:

```text
Next.js cache
TanStack Query cache
Browser cache
Backend cache
CDN
```

Cada um possui finalidade diferente.

---

# 130. Revalidation

Após mutations realizadas via Server Action:

```text
revalidatePath
```

ou:

```text
revalidateTag
```

pode ser utilizado conforme estratégia do projeto.

Evitar revalidar toda aplicação desnecessariamente.

---

# 131. Suspense

Utilizar Suspense para quebrar carregamentos independentes.

Exemplo:

```text
Dashboard
 ├── Summary
 ├── RecentItems
 └── Chart
```

Pode permitir carregamento progressivo.

---

# 132. Streaming

O App Router permite entregar conteúdo progressivamente.

Utilizar apenas onde melhorar experiência.

Não adicionar complexidade se a tela for simples.

---

# 133. SEO

Em sistemas administrativos internos, SEO normalmente não é prioridade.

Ainda assim, metadados podem ser usados para:

```text
título
favicon
nome do sistema
descrições
```

---

# 134. Metadata

Utilizar API de metadata do Next.js quando necessário.

Exemplo:

```ts
export const metadata = {
  title: "Produtos",
};
```

---

# 135. Internacionalização

Não adicionar i18n inicialmente se o produto possuir apenas um idioma.

Se necessário futuramente, introduzir uma solução dedicada.

Evitar strings espalhadas caso multi-idioma já seja requisito conhecido.

---

# 136. Testes

Mesmo que não façam parte da primeira implementação, a arquitetura deve permitir:

```text
Unit tests
Component tests
Integration tests
E2E tests
```

---

# 137. Testes Unitários

Possíveis ferramentas:

```text
Vitest
```

Indicados para:

```text
formatters
schemas
helpers
permission logic
mappers
```

---

# 138. Testes de Componentes

Pode utilizar:

```text
Testing Library
```

Para testar:

```text
forms
buttons
conditional rendering
permissions
validation
```

---

# 139. E2E

Pode utilizar:

```text
Playwright
```

Fluxos prioritários:

```text
Login
Logout
Criar registro
Editar registro
Permissão negada
Refresh de sessão
Fluxos críticos do negócio
```

---

# 140. Mock de API

Pode utilizar:

```text
MSW
```

quando for útil desacoplar testes da API real.

---

# 141. Documentação

Documentação inicial recomendada:

```text
docs
 ├── frontend-architecture.md
 ├── creating-feature.md
 ├── forms.md
 ├── tables.md
 ├── permissions.md
 ├── bff.md
 └── api-generation.md
```

---

# 142. Estrutura Inicial do Repositório

```text
frontend
 ├── docs
 │   └── architecture.md
 │
 ├── public
 │
 ├── src
 │   ├── app
 │   ├── config
 │   ├── features
 │   ├── generated
 │   └── shared
 │
 ├── .env.example
 ├── .gitignore
 ├── Dockerfile
 ├── eslint.config.js
 ├── next.config.ts
 ├── orval.config.ts
 ├── package.json
 ├── prettier.config.js
 ├── tailwind.config.ts
 └── tsconfig.json
```

---

# 143. Ordem Inicial de Implementação

Em um repositório vazio:

```text
1. Criar projeto Next.js.
2. Habilitar TypeScript strict.
3. Configurar ESLint.
4. Configurar Prettier.
5. Configurar Tailwind.
6. Configurar shadcn/ui.
7. Criar estrutura app/features/shared/generated/config.
8. Criar layout base.
9. Configurar tema.
10. Criar componentes globais básicos.
11. Configurar API client.
12. Configurar Orval.
13. Gerar contratos iniciais.
14. Criar BFF base.
15. Criar authenticated fetch.
16. Criar tratamento de refresh token.
17. Criar auth.
18. Criar login.
19. Criar logout.
20. Criar /auth/me.
21. Criar sistema de permissões.
22. Criar layout autenticado.
23. Criar sidebar.
24. Criar breadcrumbs.
25. Criar error boundaries.
26. Criar loading skeletons.
27. Criar PageHeader.
28. Criar DataTable base.
29. Configurar React Hook Form + Zod.
30. Configurar nuqs.
31. Configurar TanStack Query quando necessário.
32. Criar primeira feature de negócio.
33. Criar Dockerfile.
34. Criar .env.example.
35. Configurar pipeline CI.
```

---

# 144. Checklist de Nova Feature

```text
[ ] Qual domínio esta feature representa?
[ ] Quais rotas serão necessárias?
[ ] Quais permissões serão necessárias?
[ ] A página pode ser Server Component?
[ ] Existe necessidade real de Client Component?
[ ] Existe formulário?
[ ] Precisa React Hook Form?
[ ] Precisa Zod?
[ ] Precisa Server Action?
[ ] Precisa Route Handler?
[ ] Precisa TanStack Query?
[ ] Existe estado navegável?
[ ] Deve ficar na URL?
[ ] Precisa Zustand?
[ ] Existe tabela?
[ ] Tem paginação?
[ ] Tem filtros?
[ ] Tem ordenação?
[ ] Precisa nested routes?
[ ] Tem upload?
[ ] Tem download?
[ ] Existe ação destrutiva?
[ ] Precisa confirmação?
[ ] Existe fluxo assíncrono?
[ ] Como erros serão exibidos?
[ ] Existe estado vazio?
[ ] Existe loading?
[ ] Os tipos já existem no OpenAPI?
```

---

# 145. Checklist de Componente

```text
[ ] É específico de uma feature?
[ ] É realmente compartilhado?
[ ] Precisa ser Client Component?
[ ] Pode permanecer Server Component?
[ ] Possui props claras?
[ ] Possui responsabilidade única?
[ ] Existe acessibilidade adequada?
[ ] Possui estado desnecessário?
[ ] Está duplicando regra de negócio?
```

---

# 146. Checklist de Formulário

```text
[ ] React Hook Form.
[ ] Schema Zod.
[ ] Validação frontend alinhada ao backend.
[ ] Erros backend mapeados para campos.
[ ] Loading de submit.
[ ] Double submit bloqueado visualmente.
[ ] Permissão validada no servidor.
[ ] Feedback de sucesso.
[ ] Feedback de erro.
[ ] Valores mascarados são normalizados.
```

---

# 147. Checklist de Listagem

```text
[ ] Paginação.
[ ] Busca.
[ ] Debounce.
[ ] Filtros.
[ ] Ordenação.
[ ] Estado na URL.
[ ] Loading skeleton.
[ ] Empty state.
[ ] Permissões de ações.
[ ] Responsividade.
```

---

# 148. Checklist de BFF

```text
[ ] O browser realmente precisa deste endpoint?
[ ] O token permanece server-side?
[ ] Authorization é adicionada corretamente?
[ ] 401 dispara refresh quando aplicável?
[ ] Refresh possui proteção contra loop?
[ ] Erros estão padronizados?
[ ] Timeout está configurado?
[ ] Logs não expõem tokens?
[ ] Upload/download está protegido?
[ ] Correlation ID é propagado quando necessário?
```

---

# 149. Checklist de Segurança

```text
[ ] Tokens em cookies httpOnly.
[ ] Cookies secure em produção.
[ ] SameSite definido.
[ ] Nenhum token em localStorage.
[ ] Nenhum secret NEXT_PUBLIC.
[ ] Server Actions validam autenticação.
[ ] Server Actions validam autorização.
[ ] Backend continua protegendo endpoints.
[ ] CSP revisada.
[ ] CSRF considerado.
[ ] Dados sensíveis mascarados quando necessário.
[ ] Logs sem dados sensíveis.
[ ] Downloads protegidos via BFF.
```

---

# 150. O que Evitar

Evitar:

```text
Tudo em components/

Tudo em hooks/

Tudo em utils/

Toda página como "use client"

TanStack Query para qualquer GET

Zustand para qualquer estado

Filtros em estado global

Tokens em localStorage

Browser acessando diretamente API autenticada

Duplicar tipos do OpenAPI

Editar código gerado

Regra de negócio no frontend

Shared contendo código específico de domínio

Componente genérico extremamente configurável
para resolver apenas dois casos

Uma abstração para cada componente simples
```

---

# 151. Regra de Escolha por Tipo de Estado

```text
Form state
→ React Hook Form

URL state
→ nuqs

Remote server state interativo
→ TanStack Query

Global UI state
→ Zustand

Server data
→ Server Component

Local UI state
→ useState
```

Essa regra evita utilizar Zustand ou React Query para resolver todos os problemas.

---

# 152. Regra de Escolha por Tipo de Operação

```text
Leitura inicial
→ Server Component

Form submit
→ Server Action

Endpoint HTTP BFF
→ Route Handler

Interação client-side frequente
→ TanStack Query

Filtro/paginação
→ URL + nuqs

Estado global mínimo
→ Zustand
```

---

# 153. Fluxo de Leitura

```text
Browser
 ↓
Next.js Server Component
 ↓
BFF / Backend Client
 ↓
Backend
 ↓
Response
 ↓
HTML / RSC
```

---

# 154. Fluxo de Mutation com Server Action

```text
Client Form
 ↓
Server Action
 ↓
Auth check
 ↓
Permission check
 ↓
Backend
 ↓
Result
 ↓
Revalidate / Redirect / Error
```

---

# 155. Fluxo Client-side Interativo

```text
Client Component
 ↓
TanStack Query
 ↓
BFF Route Handler
 ↓
Backend
```

Utilizar somente quando o comportamento client-side justificar.

---

# 156. Fluxo de Upload

```text
Browser
 ↓
Client Component
 ↓
BFF
 ↓
Backend
 ↓
Storage
```

---

# 157. Fluxo de Download

```text
Browser
 ↓
BFF
 ↓
Backend
 ↓
Storage
 ↓
Stream
 ↓
Browser
```

---

# 158. Fluxo de Autenticação

```text
Login Form
 ↓
Server Action / BFF
 ↓
Backend
 ↓
JWT + Refresh Token
 ↓
Cookies httpOnly
 ↓
Authenticated Layout
```

---

# 159. Evolução Arquitetural

Possível evolução:

```text
Server Components
      ↓
Server Actions
      ↓
Client interactivity
      ↓
TanStack Query
      ↓
Realtime / Polling
      ↓
WebSocket / SSE
```

Não iniciar pelo último estágio se o primeiro já resolve o problema.

---

# 160. Melhorias Futuras

Possíveis evoluções:

```text
PWA
Offline support
Sentry
OpenTelemetry
Feature Flags
Internationalization
Storybook
Visual regression tests
Playwright
MSW
Realtime
WebSockets
Server-Sent Events
Advanced analytics
Design tokens
```

Adicionar apenas quando houver necessidade real.

---

# 161. Critérios para Criar uma Nova Abstração

Criar uma abstração quando:

- Existem múltiplos usos reais.
- Existe comportamento repetido.
- Existe variação real.
- Existe complexidade que precisa ser escondida.
- Existe política global.

Não abstrair simplesmente porque duas linhas parecem semelhantes.

---

# 162. Critérios para Mover Algo para `shared`

Mover para `shared` quando:

```text
É utilizado por múltiplas features
+
Não pertence conceitualmente a uma feature
+
Tem comportamento suficientemente estável
```

Caso contrário, deixar próximo do domínio.

---

# 163. Critérios para Client Component

Adicionar `"use client"` somente se o componente precisar de pelo menos uma capacidade client-side.

Exemplos:

```text
hooks React client-side
event handlers
browser APIs
React Hook Form
TanStack Query
Zustand
```

Não usar apenas porque um componente filho é interativo.

---

# 164. Critérios para TanStack Query

Utilizar quando houver:

```text
cache client-side
refetch
polling
optimistic update
mutation interativa
múltiplos consumers
dados remotos dinâmicos
```

Não utilizar por padrão para uma página simples carregada uma vez.

---

# 165. Critérios para Zustand

Utilizar quando:

```text
estado precisa ser acessado por partes distantes
e
não pertence à URL
e
não vem da API
e
não é formulário
```

Caso contrário, avaliar solução mais local.

---

# 166. Critérios para Nested Routes

Utilizar quando:

```text
cada seção merece URL própria
+
pode ser acessada diretamente
+
possui carregamento próprio
```

Exemplo:

```text
/entity/[id]/details
/entity/[id]/history
/entity/[id]/files
```

---

# 167. Critérios para BFF

Utilizar BFF quando a operação exigir:

```text
token server-side
cookie httpOnly
refresh token
proteção de segredo
adaptação de API
proxy de upload/download
```

No projeto autenticado, esse será o fluxo padrão.

---

# 168. Resumo Arquitetural

A arquitetura frontend utilizará:

```text
Next.js App Router
+
React
+
TypeScript
+
Feature-Based Architecture
+
BFF
+
Server-first Rendering
```

A interface será organizada em:

```text
app
features
shared
generated
config
```

`app` será responsável por:

```text
rotas
layouts
route handlers
error boundaries
loading
```

`features` será responsável por:

```text
componentes de domínio
actions
queries
schemas
hooks
types
utils específicos
```

`shared` será responsável apenas por elementos realmente reutilizáveis.

`generated` conterá contratos produzidos automaticamente a partir de OpenAPI.

O fluxo autenticado principal será:

```text
Browser
   ↓
Next.js / BFF
   ↓
Backend API
```

Tokens serão mantidos em:

```text
cookies httpOnly
```

e nunca ficarão disponíveis diretamente no JavaScript do navegador.

A estratégia de renderização será:

```text
Server Components
por padrão

Client Components
quando houver interatividade real
```

Mutations simples deverão preferir:

```text
Server Actions
```

Telas altamente interativas poderão utilizar:

```text
TanStack Query
```

Filtros, paginação, ordenação e estados navegáveis deverão preferir:

```text
URL
+
nuqs
```

Formulários utilizarão:

```text
React Hook Form
+
Zod
```

Estado global deverá ser mínimo e utilizar:

```text
Zustand
```

somente quando realmente necessário.

Contratos HTTP serão derivados do OpenAPI através de:

```text
Orval
```

Código gerado nunca deverá receber alterações manuais.

O frontend não deverá duplicar regras críticas de negócio.

Essas regras continuam pertencendo ao backend.

O objetivo final é manter uma arquitetura:

```text
segura
modular
previsível
server-first
type-safe
com pouco estado global
fácil de navegar
fácil de evoluir
```

Regra final:

```text
Server antes de Client.

Feature antes de Shared.

URL antes de Store.

Estado local antes de global.

Contrato gerado antes de tipo duplicado.

BFF antes de expor tokens.

Componente simples antes de abstração genérica.

Complexidade somente quando houver necessidade.
```
