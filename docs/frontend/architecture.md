# Arquitetura de frontend

Referência para o frontend de sistemas de informação, pensada como ponto de partida de projetos novos, antes das funcionalidades de negócio. A base é Next.js com App Router, React, TypeScript, Tailwind CSS e shadcn/ui, em arquitetura orientada a features, com o Next atuando como BFF e renderização server-first. As prioridades são organização por domínio, baixo acoplamento, segurança, estado client-side mínimo, contratos sincronizados com o backend, evolução incremental, manutenção fácil e deploy simples com Docker.

O princípio que atravessa o resto do documento: server primeiro e client quando necessário; estado local antes de global; URL antes de store para filtros e navegação; backend como fonte da verdade; `shared` apenas para código realmente compartilhado; abstrações apenas quando houver benefício real.

---

## 1. Estilo arquitetural

O código é organizado por domínio, e código da mesma funcionalidade fica junto. Em fluxos autenticados o navegador não acessa o backend diretamente: o Next.js funciona como Backend for Frontend, e o browser não conhece os detalhes internos dele. Server Components são o padrão, e `"use client"` entra quando existe interatividade real. Server Actions cobrem as mutations quando fizer sentido, e o TanStack Query fica reservado para estados remotos altamente interativos. Filtros, paginação, busca e abas compartilháveis preferem a URL, e o estado global é mínimo. Regras críticas de negócio e segurança continuam no backend, clients gerados por OpenAPI não recebem lógica manual, e código de domínio não vai para `shared` só para "organizar".

### Stack

Base: Next.js, React, TypeScript, Tailwind CSS, shadcn/ui, React Hook Form, Zod, TanStack Query, TanStack Table, nuqs, Zustand, Orval, Sonner, ESLint, Prettier e Docker.

Opcionais: Tiptap, Recharts, date-fns, React Dropzone, Sentry, OpenTelemetry, Playwright, Vitest, Testing Library e MSW. Adicione dependência apenas quando houver necessidade concreta.

### Decisões técnicas

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

## 2. Estrutura geral

```text
src
 ├── app
 │   ├── (auth)
 │   │   └── login
 │   │       └── page.tsx
 │   │
 │   ├── (app)
 │   │   ├── layout.tsx
 │   │   ├── module-a
 │   │   └── module-b
 │   │
 │   ├── api
 │   │   ├── auth
 │   │   ├── module-a
 │   │   ├── module-b
 │   │   └── integrations
 │   │
 │   ├── unauthorized.tsx
 │   ├── not-found.tsx
 │   ├── error.tsx
 │   └── loading.tsx
 │
 ├── features
 │   ├── auth
 │   ├── users
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

`module-a` e `module-b` são os domínios específicos do sistema: orders, inventory, customers, contracts, projects, students, appointments, documents, payments, assets, employees e afins.

### `app`

Guarda rotas, layouts (inclusive aninhados), route groups, route handlers, pages, loading states, error boundaries e not found. A regra das funcionalidades fica em `features`. Route Groups separam contextos sem necessariamente alterar a URL, e os mais comuns são `(auth)`, `(app)` e `(public)`.

### `features`

Contém o código específico de cada domínio: `ProductForm`, `ProductsTable`, `OrderStatusBadge`, `PaymentFilters`, `DocumentActions`. Se uma implementação pertence claramente a um domínio, ela fica nele.

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

Nenhuma feature precisa de todas essas pastas. Crie conforme a necessidade.

### `shared`

Só entra aqui o que é de fato reutilizável entre features: `Button`, `DataTable`, `Pagination`, `EmptyState`, `PageHeader`, `ConfirmDialog`, `DatePicker`, `CurrencyInput`, `useDebounce`, `ApiError`, `formatCurrency`. Coisas como `shared/components/customer-form.tsx`, `shared/utils/order-utils.ts` ou `shared/hooks/use-payment.ts` pertencem a uma feature.

## 3. BFF

O Next.js é o Backend for Frontend, e o browser não chama diretamente o backend autenticado:

```text
Browser → Next.js → Route Handler / Server Action → Backend API → Database / Storage / Integrations
```

O BFF lê cookies httpOnly, adiciona autenticação nas chamadas internas, executa o refresh token, encapsula detalhes do backend, padroniza respostas e erros quando necessário, intermedia uploads e downloads protegidos, oculta tokens do navegador, isola a URL interna da API e adapta contratos.

Os Route Handlers ficam em `app/api` (`products/route.ts`, `products/[id]/route.ts`, `auth/login/route.ts`, `auth/logout/route.ts`), podem expor `GET`, `POST`, `PUT`, `PATCH` e `DELETE`, e o caminho típico é `Browser → GET /api/products → Route Handler → GET backend/products`.

O BFF não é um segundo backend de negócio e não duplica regras. Ele cuida de autenticação, adaptação e segurança, enquanto a regra de negócio continua do outro lado.

## 4. Autenticação

A estratégia combina JWT, refresh token e cookies httpOnly, sempre através do BFF. O browser não acessa os tokens diretamente.

Os cookies `access_token` e `refresh_token` usam `httpOnly: true`, `secure: true` em produção, `sameSite` lax ou strict e `path: /`. Tokens nunca vão para `localStorage`, `sessionStorage`, Zustand ou React state.

### Login

```text
1. Usuário abre /login e envia credenciais.
2. Server Action ou Route Handler recebe os dados.
3. BFF chama o backend, que valida as credenciais.
4. Backend retorna tokens e dados do usuário.
5. BFF grava os tokens em cookies httpOnly.
6. Usuário é redirecionado e os dados da sessão são carregados.
```

### Refresh e logout

```text
Request → 401 → BFF verifica refresh token → POST /auth/refresh-token → novo access token → cookie atualizado → request original refeita

Usuário → Logout → BFF → Backend /auth/logout → cookies removidos → redirect /login
```

Se o refresh falhar, limpe os cookies, invalide a sessão local e redirecione para `/login`. No logout, mesmo que a chamada ao backend falhe, os cookies locais precisam ser tratados adequadamente.

### Usuário autenticado

`GET /auth/me` devolve `id`, `name`, `email`, `role` e `permissions`, que alimentam o menu, o header, o controle visual de permissões, o estado mínimo da sessão e as informações da conta.

## 5. Autorização

A autorização real continua no backend, que protege o recurso. O frontend esconde a ação sem permissão, impede a navegação normal e apresenta 403 quando for o caso.

Um helper de permissões dá o controle fino, e um componente cobre o caso mais comum:

```tsx
can("PRODUCTS_CREATE");

{
  can("PRODUCTS_CREATE") && <Button>Novo produto</Button>;
}

<PermissionGuard permission="PRODUCTS_UPDATE" fallback={null}>
  <Button>Editar</Button>
</PermissionGuard>;
```

As permissões controlam visualizar, criar, editar, excluir, aprovar, cancelar, baixar e reenviar. O menu é filtrado pela mesma informação:

```ts
const menuItems = [
  { label: "Produtos", href: "/products", permission: "PRODUCTS_VIEW" },
  { label: "Pedidos", href: "/orders", permission: "ORDERS_VIEW" },
];
```

Páginas sensíveis validam antes de renderizar (`Page → Session → Permission check → Render`, ou `forbidden()`), e toda mutation protegida verifica sessão e permissão no servidor, sem confiar no estado vindo do Client Component. Usuário não autenticado vai para `/login`; autenticado sem permissão recebe 403.

## 6. Server e Client Components

Server Components são a escolha padrão: páginas de listagem e de detalhes, dashboards, dados iniciais, dados autenticados, conteúdo sem interatividade, layouts, breadcrumbs e navegação baseada na sessão.

```tsx
export default async function ProductsPage() {
  const products = await getProducts();

  return <ProductsTable data={products} />;
}
```

Client Components entram quando existe necessidade real de `useState`, `useEffect`, React Hook Form, APIs do navegador, eventos interativos, Zustand, TanStack Query, modais, upload, drag-and-drop, clipboard ou qualquer componente muito interativo. O critério para `"use client"` é o componente precisar de pelo menos uma dessas capacidades, e um filho interativo não obriga o pai a ser client.

Evite marcar páginas inteiras com `"use client"`. Prefira uma página server com um Client Component pequeno dentro:

```text
ProductPage
 ├── ProductHeader        Server
 ├── ProductSummary       Server
 └── ProductActions       Client
```

## 7. Server Actions

Server Actions cobrem create, update, delete, mudança de status, submissão de formulário, comandos de negócio e ações protegidas:

```ts
"use server";

export async function createProductAction(input: CreateProductInput) {
  // validar sessão
  // validar permissão
  // chamar backend
  // tratar resposta
}
```

Cada action é uma entrada protegida e valida autenticação, permissão e input. Botão escondido não torna a action segura.

Use Server Action quando o fluxo estiver fortemente associado à aplicação React (submit de formulário, alteração de status, mutation de tela). Use Route Handler quando for necessário um endpoint HTTP explícito: BFF REST, download, upload, callback, proxy ou endpoint consumido por query client-side.

## 8. TanStack Query

Entra quando há necessidade real de cache e sincronização client-side: telas muito interativas, refetch, polling, dados por modal, abas independentes, infinite scroll, mutations client-side e optimistic update. Para uma página simples carregada uma vez, use Server Component; para mutation simples, Server Action.

As query keys ficam centralizadas por domínio (`products.keys.ts`, `orders.keys.ts`, `users.keys.ts`):

```ts
export const productKeys = {
  all: ["products"] as const,

  list: (params: ProductFilters) =>
    [...productKeys.all, "list", params] as const,

  detail: (id: string) => [...productKeys.all, "detail", id] as const,
};
```

## 9. Estado

Estados navegáveis ficam na URL, com `nuqs`: `page`, `size`, `search`, `sort`, filtros, abas e intervalos de data.

```text
/products?page=0&size=20&search=mouse&status=ACTIVE
```

Isso permite dar refresh sem perder filtros, compartilhar link, usar back/forward, salvar bookmark, e ainda reduz o estado global e a sincronização manual.

Zustand fica para estado realmente global: usuário autenticado, permissões, tema, estado reduzido da sidebar e preferências globais. Mesmo nesses casos, vale checar se React Context ou Server Components já resolvem. O critério é o estado precisar ser acessado por partes distantes da aplicação e não pertencer à URL, não vir da API e não ser formulário. Filtros, paginação, formulários, dados da API, cache de requisições e estado temporário de modal simples têm ferramentas melhores.

```text
Por tipo de estado
Form state                      → React Hook Form
URL state                       → nuqs
Remote server state interativo  → TanStack Query
Global UI state                 → Zustand
Server data                     → Server Component
Local UI state                  → useState

Por tipo de operação
Leitura inicial                 → Server Component
Form submit                     → Server Action
Endpoint HTTP BFF               → Route Handler
Interação client-side frequente → TanStack Query
Filtro / paginação              → URL + nuqs
Estado global mínimo            → Zustand
```

## 10. Formulários

O padrão é React Hook Form com Zod:

```ts
const schema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});
```

Os arquivos ficam na feature, separados por responsabilidade:

```text
features/products
 ├── schemas
 │   ├── create-product.schema.ts
 │   └── update-product.schema.ts
 ├── components
 │   └── product-form.tsx
 └── api
     ├── create-product.action.ts
     └── update-product.action.ts
```

A validação no frontend melhora a experiência, mas não substitui o backend: `Zod → Server Action / BFF → Backend Validation → Domain Rules`.

Erros de campo vindos do backend devem virar `form.setError(...)` sempre que possível (`email → Email já cadastrado`, `name → Nome obrigatório`). Erros gerais usam feedback global.

Botões de mutation ficam `disabled` e em estado de loading enquanto a requisição corre, o que reduz cliques duplicados. Operações críticas ainda precisam de idempotência no backend.

## 11. Erros, feedback e estados de tela

O tipo padrão de erro:

```ts
type ApiError = {
  type: string;
  title: string;
  status: number;
  detail: string;
  fields?: Record<string, string>;
};
```

A transformação é centralizada em `Backend ProblemDetail → mapApiError() → ApiError → UI`, para não haver tratamento diferente em cada feature. A exibição segue o tipo do erro:

```text
Validation Error  → campo
Business Error    → toast ou mensagem contextual
401               → limpar sessão + login
403               → forbidden
404               → not-found
Unexpected Error  → error boundary + log
```

O feedback pontual usa Sonner, para sucesso, erro de negócio, erro de integração e avisos. Evite toast para informação permanente.

Os error boundaries usam os arquivos do App Router (`error.tsx`, `not-found.tsx`, `loading.tsx`, e `unauthorized.tsx` quando necessário), com estados consistentes em toda a aplicação.

Loading usa `loading.tsx`, Skeleton e Suspense, com skeleton compatível com o layout final. Prefira carregamentos locais a um loading global em toda navegação.

Listagens vazias têm estado próprio, com ação sugerida quando fizer sentido ("Nenhum produto encontrado" + botão de adicionar). Vale diferenciar "nenhum registro cadastrado" de "nenhum resultado para os filtros aplicados".

## 12. API client

O backend disponibiliza OpenAPI e o frontend usa Orval para gerar types, schemas, HTTP clients, hooks de React Query e contratos:

```text
src/generated/api
 ├── schemas
 ├── auth
 ├── users
 ├── products
 └── orders
```

Código gerado nunca é editado à mão. As customizações ficam fora de `generated`, em `features/products/api` ou `shared/lib/api`, no fluxo `OpenAPI → Orval → generated/api → feature/api → UI`.

A regeneração tem comando padronizado, de modo que mudança de contrato no backend produza atualização previsível no frontend:

```json
{
  "scripts": {
    "api:generate": "orval"
  }
}
```

Os helpers de chamada server-side ficam reunidos, cuidando de base URL, headers, auth, refresh, erro, timeout, error mapping e correlation id:

```text
shared/lib/api
 ├── backend-client.ts
 ├── authenticated-fetch.ts
 ├── refresh-token.ts
 └── api-error.ts
```

Concentre isso no client em vez de espalhar `fetch` configurado na mão pelo projeto. Chamadas externas do BFF precisam de timeout, com feedback adequado quando ele estourar, para não deixar requests esperando indefinidamente. Quando o backend usar correlation id, o BFF propaga o header: `Browser request → Next BFF → X-Correlation-Id → Backend`.

Retry automático é aceitável em `GET` e pede cuidado em `POST`, `PATCH` e `DELETE`: o frontend não deve refazer mutations sem avaliar a segurança disso.

## 13. Listagens e tabelas

Listagens principais consideram paginação server-side, ordenação, filtros, busca com debounce, estado na URL, empty state, loading e permissões. O TanStack Table cuida do comportamento de tabela (columns, UI de ordenação, seleção de linha, visibilidade de coluna, renderização) e não precisa controlar sozinho todo o estado remoto.

O contrato de paginação:

```ts
type PageResponse<T> = {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
};
```

A URL carrega paginação (`?page=0&size=20`) e ordenação (`?sort=name,asc` ou `?sortBy=name&direction=asc`, acompanhando o padrão do backend). A busca textual passa por debounce antes de chegar ao servidor (`search input → debounce → URL → Server Request`), para não chamar o backend a cada caractere.

Filtros têm valor inicial vindo da URL, estado visual, serialização previsível e botão de limpar onde couber. Mesmo os complexos continuam compartilháveis por link sempre que possível.

Evite a tabela universal extremamente abstrata. Prefira um `DataTable` genérico em `shared` com columns e toolbar específicas de cada feature.

## 14. Navegação e rotas

A estrutura de rotas de uma feature:

```text
/products
/products/new
/products/[id]
/products/[id]/edit
```

Quando uma aba representa uma subárea relevante, com conteúdo e carregamento próprios, prefira nested routing (`/products/[id]/details`, `/products/[id]/history`, `/products/[id]/documents`) a `?tab=documents`. Isso dá deep link, back/forward natural, loading e error boundary independentes, organização de código e menos estado manual. O critério: cada seção merece URL própria, pode ser acessada diretamente e tem carregamento próprio.

Crie uma rota quando a informação precisa de URL própria, precisa ser compartilhável, tem navegação própria, tem ciclo de carregamento independente ou representa uma tela real do sistema. Modal funciona bem para confirmação, formulário curto, ação contextual, preview rápido e seleção auxiliar, e mal para fluxos longos: formulários maiores são mais previsíveis em `/products/new` e `/products/[id]/edit`.

Depois de criar, o padrão é redirecionar para os detalhes, o que permite continuar trabalhando no recurso. Depois de editar, o padrão é toast de sucesso e permanecer na página, ou redirecionar para os detalhes conforme o fluxo.

A configuração de menu pode ser centralizada, alimentando sidebar, breadcrumbs e filtragem por permissão:

```ts
type NavigationItem = {
  label: string;
  href: string;
  permission?: string;
};
```

## 15. Layout e design system

O layout administrativo típico tem sidebar, header, breadcrumbs, conteúdo principal, menu de usuário, toggle de tema e logout, além de loading states, error states, empty states e comportamento responsivo.

A sidebar lista domínios, usuários e configurações, renderizados conforme as permissões, e em mobile pode virar drawer ou sheet. Os breadcrumbs são derivados da rota ou da configuração de navegação (`Produtos > Produto XPTO > Editar`), em vez de definidos à mão em cada página. O tema suporta light, dark e system, e é uma das poucas preferências que cabem em estado global ou persistência local.

shadcn/ui é a base, o que não dispensa padronizar internamente `Button`, `Input`, `Select`, `Dialog`, `Sheet`, `Table`, `Card`, `Badge`, `Alert`, `Form`, `DatePicker` e `Combobox`.

Componentes compartilhados: `PageHeader`, `PageContainer`, `DataTable`, `Pagination`, `SearchInput`, `EmptyState`, `LoadingSkeleton`, `ConfirmDialog`, `PermissionGuard`, `StatusBadge`, `DateRangePicker`. Só promova um componente para `shared` quando houver reutilização real ou padrão global claro. Componentes de feature como `ProductForm`, `OrderItemsTable`, `CustomerAddressForm` e `PaymentStatusCard` continuam nas suas features, mesmo usando componentes de `shared`.

## 16. Uploads, downloads e arquivos

Uploads e downloads protegidos passam pelo BFF (`Browser → Next.js BFF → Backend → Storage`). Isso vale para documentos, imagens, logos, anexos e importações, e é especialmente importante para arquivos privados, jurídicos, financeiros ou pessoais.

O upload normalmente exige Client Component, por depender de File API, drag-and-drop, progresso e preview, mas a submissão continua passando pelo BFF. Valide tipo, tamanho e quantidade no frontend e repita as validações relevantes no backend.

Preview segue a autorização: PDF em iframe ou viewer, imagem em preview, texto em visualização.

## 17. Processos assíncronos

Quando o backend tem processos externos com estado (`PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`), a interface apresenta esse ciclo claramente, com `StatusBadge`, `LastUpdatedAt`, `RetryAction` e `RefreshStatusAction`. O frontend mapeia os status do backend para apresentação (`PENDING → badge`, `FAILED → badge vermelho + ação permitida`), sem inventar estados próprios nem duplicar a máquina de estados.

O frontend também não assume que uma operação externa terminou só porque a chamada inicial deu certo:

```text
Usuário envia documento → API responde "request created" → UI mostra SENT / PROCESSING → backend recebe webhook → status vira COMPLETED
```

A interface reflete o estado persistido pelo backend. Se uma ação inicia processamento assíncrono, ela comunica que foi aceita, mas ainda não concluída.

Polling não é ligado por padrão. Use quando o processo exigir atualização frequente, WebSocket e SSE não estiverem disponíveis e a atualização manual não bastar. Fora disso, um botão explícito de atualizar status costuma resolver, refazendo a consulta sem exigir reload completo.

## 18. Confirmações

Ações destrutivas ou relevantes (excluir, cancelar, reabrir, enviar, renegociar, substituir) passam por `ConfirmDialog`, deixando explícitos a ação, a consequência e o recurso afetado. Em vez do genérico "Tem certeza?", prefira algo como "Cancelar este pedido impedirá novas alterações. Deseja continuar?".

## 19. Formatação de dados

Formatadores universais como `formatDate`, `formatCurrency` e `formatPercentage` ficam em `shared/utils`. Os específicos de domínio ficam na feature, como `features/payments/utils/payment-formatters.ts`.

O tratamento de datas precisa ser padronizado e explícito: UTC no backend ou timezone local, formato de envio e formato de exibição. A UI formata a data, sem alterar a semântica dela.

Máscaras (telefone, documento, CEP, moeda, percentual) são responsabilidade de apresentação. O valor enviado ao backend segue o contrato definido, e o valor mascarado não deve ficar acoplado ao valor persistido.

## 20. Responsividade e acessibilidade

O sistema funciona em desktop, tablet e mobile, mesmo quando o foco é painel administrativo. Tabelas grandes podem usar scroll horizontal, cards responsivos ou colunas adaptativas, conforme o caso.

Os componentes preservam navegação por teclado, labels, focus states, contraste, `aria-*` quando necessário e estrutura semântica. shadcn/ui e Radix ajudam, mas a responsabilidade continua sendo da implementação.

## 21. Segurança

Tokens ficam em cookies httpOnly, o browser autenticado nunca acessa o backend diretamente, as permissões são validadas no servidor, CSRF é tratado quando necessário, a CSP é configurada, e não existem secrets em código client-side nem tokens em log.

Toda variável `NEXT_PUBLIC_*` é visível ao navegador, então `API_SECRET`, `JWT_SECRET`, `PRIVATE_TOKEN` e `REFRESH_TOKEN_SECRET` nunca podem ser públicas. Quando o browser não precisa da API, prefira `API_INTERNAL_URL=https://api.example.com` a `NEXT_PUBLIC_API_URL`.

Como a autenticação usa cookies, avalie proteção CSRF para operações sensíveis. As opções dependem da infraestrutura (SameSite, validação de Origin, CSRF token, mutations apenas via BFF) e a escolha deve ser documentada em cada projeto. Configure CSP onde for apropriado, com atenção a scripts, imagens, iframes, fontes, editores externos e analytics, evitando políticas permissivas demais.

Informações sensíveis podem aparecer parcialmente mascaradas (`***.***.***-12`, `**** **** **** 1234`, `jo***@email.com`), no nível que o domínio exigir. Logs de frontend nunca registram JWT, refresh token, senhas, secrets ou dados sensíveis completos.

Error tracking com Sentry, OpenTelemetry ou APM pode ser adicionado depois, sempre revisando o que é enviado a terceiros.

## 22. Cache e renderização

Cache não se adiciona sem entender o requisito. Existem níveis diferentes, cada um com sua finalidade: Next.js, TanStack Query, browser, backend e CDN. Depois de mutations via Server Action, `revalidatePath` ou `revalidateTag` podem ser usados conforme a estratégia do projeto, sem revalidar a aplicação inteira à toa.

Em sistemas administrativos internos SEO normalmente não é prioridade, ainda que metadados sirvam para título, favicon, nome do sistema e descrições:

```ts
export const metadata = {
  title: "Produtos",
};
```

i18n não entra no início se o produto tiver um idioma só. Se multi-idioma já for requisito conhecido, evite espalhar strings pelo código e introduza uma solução dedicada.

## 23. Estrutura do repositório e deploy

```text
frontend
 ├── docs
 ├── public
 ├── src
 │   ├── app
 │   ├── config
 │   ├── features
 │   ├── generated
 │   └── shared
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

O deploy padrão é Docker em VPS ou cloud, com `next build` e `output: standalone`, acompanhado de `docker-compose.yml` e `.env.example`. Os ambientes sugeridos são local, development, staging e production, e o container recebe a configuração por ambiente, sem valores hardcoded:

```env
NEXT_PUBLIC_APP_URL=https://app.example.com

API_INTERNAL_URL=https://api.example.com

AUTH_ACCESS_COOKIE_NAME=access_token
AUTH_REFRESH_COOKIE_NAME=refresh_token

NODE_ENV=production
```

## 24. Qualidade de código

ESLint, Prettier e TypeScript em modo strict (`"strict": true`), evitando o uso indiscriminado de `any`.

Prefira tipos derivados dos contratos gerados a redefinir `type Product = ...` quando o Orval já gerou o equivalente. Tipos locais só para estado de UI, view model, form state e adaptações específicas.

### Naming

```text
Componentes   product-form.tsx, products-table.tsx, product-details-card.tsx
Schemas       create-product.schema.ts, update-product.schema.ts
Actions       create-product.action.ts, update-product.action.ts, delete-product.action.ts
Query keys    products.keys.ts, orders.keys.ts
Hooks         use-products-filters.ts, use-product-permissions.ts
```

Em React, os componentes são `ProductForm`, `ProductsTable` e `ProductDetailsCard`, nos arquivos em kebab-case acima. Hooks têm responsabilidade clara: `useProductFilters`, `useProductPermissions` e `useProductSelection`, e não `useProductEverything` ou `useGlobalStuff`.

Nada de `utils.ts` gigante. Prefira `currency.ts`, `date.ts`, `string.ts`, `product-formatters.ts`, mantendo na feature o que for específico dela. O mesmo vale para constants, que ficam em `shared/constants` quando globais e em `features/products/constants` quando específicas. As configurações da aplicação ficam em `src/config`: `app.config.ts`, `auth.config.ts`, `navigation.config.ts`.

## 25. Ordem inicial de implementação

Em um repositório vazio:

```text
 1. Criar projeto Next.js com TypeScript strict.
 2. Configurar ESLint, Prettier e Tailwind.
 3. Configurar shadcn/ui.
 4. Criar a estrutura app / features / shared / generated / config.
 5. Criar layout base, tema e componentes globais básicos.
 6. Configurar API client e Orval, e gerar os contratos iniciais.
 7. Criar o BFF base, o authenticated fetch e o tratamento de refresh token.
 8. Criar auth, login, logout e /auth/me.
 9. Criar o sistema de permissões e o layout autenticado.
10. Criar sidebar e breadcrumbs.
11. Criar error boundaries e loading skeletons.
12. Criar PageHeader e DataTable base.
13. Configurar React Hook Form + Zod e nuqs.
14. Configurar TanStack Query quando for necessário.
15. Criar a primeira feature de negócio.
16. Criar Dockerfile e .env.example, e configurar a pipeline de CI.
```

## 26. O que evitar

- Tudo em `components/`, tudo em `hooks/` ou tudo em `utils/`.
- Toda página marcada como `"use client"`.
- TanStack Query para qualquer GET e Zustand para qualquer estado.
- Filtros em estado global.
- Tokens em `localStorage`.
- Browser acessando diretamente a API autenticada.
- Duplicar tipos do OpenAPI ou editar código gerado.
- Regra de negócio no frontend.
- `shared` contendo código específico de domínio.
- Componente genérico extremamente configurável para resolver apenas dois casos, ou uma abstração para cada componente simples.

## 27. Fluxos principais

```text
Leitura
Browser → Server Component → BFF / Backend Client → Backend → Response → HTML / RSC

Mutation com Server Action
Client Form → Server Action → auth check → permission check → Backend → Revalidate / Redirect / Error

Client-side interativo
Client Component → TanStack Query → BFF Route Handler → Backend

Upload
Browser → Client Component → BFF → Backend → Storage

Download
Browser → BFF → Backend → Storage → Stream → Browser

Autenticação
Login Form → Server Action / BFF → Backend → JWT + Refresh Token → cookies httpOnly → Authenticated Layout
```

O fluxo client-side interativo só se justifica quando o comportamento no navegador exigir.
