# InboxAI — Estrutura de Pastas

Mapa completo das pastas relevantes do boilerplate. Quando precisar adicionar algo novo, consulte aqui pra saber **onde colocar**.

## Visêo geral

```
inboxai-boilerplate/
├── .env.example              # template de vars (copia pra .env.local)
├── AGENTS.md                 # instruções pro agente de IA do repo
├── README.md
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── next.config.ts
├── eslint.config.js
├── docker-compose.dev.yml
├── docker-compose.prod.yml
├── convex/                   # backend (Convex)
└── src/                      # frontend (Next.js App Router)
```

## `convex/` — Backend

Todo código de servidor vive aqui. Nada de `app/api/*` no Next.

```
convex/
├── _generated/               # gerado automático pelo Convex CLI — NêO editar
├── schema.ts                 # define tabelas e indexes
├── http.ts                   # HTTP routes (webhooks Clerk, Stripe, Evolution)
├── auth.config.ts            # integraçêo com Clerk
│
├── contact.ts                # CRUD de contato (queries + mutations)
├── message.ts                # CRUD de mensagem
├── deal.ts                   # CRUD de deal + movimentaçêo no pipeline
├── instance.ts               # gestêo de instâncias WhatsApp
├── org.ts                    # operações de org (multi-tenant)
├── user.ts                   # CRUD de usuário
├── agentConfig.ts            # config do agente IA por org
│
├── actions/                  # actions = side-effects (chamadas externas)
│   ├── classifyContact.ts    # chama Anthropic pra classificar lead
│   ├── generateResponse.ts   # chama Anthropic pra gerar resposta
│   ├── sendMessage.ts        # POST pra Evolution API
│   ├── stripeCheckout.ts     # cria checkout session
│   └── webhookProcessors/
│       ├── whatsapp.ts       # processa webhook Evolution
│       ├── clerk.ts          # processa webhook Clerk
│       └── stripe.ts         # processa webhook Stripe
│
├── lib/                      # helpers internos
│   ├── auth.ts               # requireOrg, requireUser
│   ├── prompts.ts            # templates de prompt (com cache)
│   ├── normalizers.ts        # normalizar número WhatsApp E.164, etc
│   └── validators.ts         # schemas Zod
│
└── crons.ts                  # tarefas agendadas (TTL de mensagens, follow-up)
```

### Convenções
- 1 arquivo por entidade de domínio (`contact.ts`, `deal.ts`, etc.)
- Dentro do arquivo: queries + mutations da entidade
- Action vai em `actions/` se faz side-effect externo
- Helper interno (nêo exposto) vai em `lib/`

## `src/` — Frontend

```
src/
├── app/                      # rotas Next.js App Router
│   ├── layout.tsx            # layout raiz com Clerk + Convex providers
│   ├── page.tsx              # landing pública
│   ├── globals.css
│   │
│   ├── (auth)/               # rotas nêo-autenticadas
│   │   ├── sign-in/[[...sign-in]]/page.tsx
│   │   └── sign-up/[[...sign-up]]/page.tsx
│   │
│   ├── (app)/                # rotas autenticadas (middleware Clerk valida)
│   │   ├── layout.tsx        # sidebar + navbar
│   │   ├── dashboard/page.tsx
│   │   ├── inbox/
│   │   │   ├── page.tsx      # lista de threads
│   │   │   └── [contactId]/page.tsx  # thread individual
│   │   ├── contacts/
│   │   │   ├── page.tsx      # tabela de contatos
│   │   │   └── [id]/page.tsx
│   │   ├── deals/page.tsx    # kanban
│   │   ├── reports/page.tsx
│   │   └── settings/
│   │       ├── general/page.tsx
│   │       ├── whatsapp/page.tsx     # conectar instância
│   │       ├── agent/page.tsx        # editar prompt do agente
│   │       ├── team/page.tsx
│   │       └── billing/page.tsx
│   │
│   └── api/                  # SOMENTE webhooks que precisam estar no front
│       └── stripe/webhook/route.ts   # Stripe quer URL do front
│
├── components/               # componentes reutilizáveis
│   ├── ui/                   # shadcn/ui (button, card, dialog, ...)
│   ├── inbox/
│   │   ├── ThreadList.tsx
│   │   ├── ThreadView.tsx
│   │   ├── MessageBubble.tsx
│   │   └── ComposeBar.tsx
│   ├── deals/
│   │   ├── KanbanBoard.tsx
│   │   ├── KanbanColumn.tsx
│   │   └── DealCard.tsx
│   ├── contacts/
│   │   ├── ContactsTable.tsx
│   │   └── ContactDetail.tsx
│   ├── settings/
│   │   ├── WhatsAppConnect.tsx
│   │   └── AgentPromptEditor.tsx
│   ├── shared/
│   │   ├── PageHeader.tsx
│   │   ├── EmptyState.tsx
│   │   └── LoadingState.tsx
│   └── icons.tsx             # lucide re-exports
│
├── features/                 # módulos verticais (lógica + componentes específicos)
│   ├── inbox/
│   │   ├── hooks.ts          # useThreads, useThreadMessages
│   │   └── helpers.ts        # formatMessageTime, etc
│   ├── deals/
│   │   ├── hooks.ts
│   │   └── stages.ts         # config de estágios do pipeline
│   └── contacts/
│       ├── hooks.ts
│       └── filters.ts
│
├── lib/                      # utils puros (sem hook)
│   ├── errors.ts             # DomainError, mapeamento
│   ├── error-messages.ts     # mensagens user-friendly
│   ├── format.ts             # formatBRL, formatPhone, formatDate
│   ├── cn.ts                 # tailwind classnames helper
│   └── env.ts                # validaçêo de env vars com Zod
│
├── hooks/                    # hooks compartilhados
│   ├── useCurrentOrg.ts
│   ├── useCurrentUser.ts
│   └── useToast.ts
│
├── styles/                   # CSS extra além do globals
│   └── animations.css
│
└── test/                     # helpers de teste
    ├── factories/
    │   ├── contact.ts
    │   ├── deal.ts
    │   └── message.ts
    └── setup.ts
```

### Convenções

- **`app/`** — só páginas e layouts. Lógica fica em `features/` ou `components/`.
- **`components/ui/`** — shadcn/ui copiados (nêo dependência). Nêo editar assinatura, só estilizar.
- **`components/<area>/`** — componente que pertence a uma área específica do app.
- **`features/<area>/hooks.ts`** — hooks que combinam várias queries Convex pra entregar dado pronto pro componente.
- **`lib/`** — funções puras, sem React, sem side-effect.
- **`test/factories/`** — funçêo `createDeal({ ...overrides })` que retorna deal válido pra teste.

## Onde colocar coisa nova

| Você está adicionando... | Vai em... |
|--------------------------|-----------|
| Nova rota | `src/app/(app)/<rota>/page.tsx` |
| Componente reutilizável | `src/components/<area>/<Componente>.tsx` |
| Componente shadcn novo | `npx shadcn@latest add <comp>` (vai pra `components/ui/`) |
| Nova entidade no banco | `convex/<entidade>.ts` + entrada em `convex/schema.ts` |
| Funçêo pura | `src/lib/<arquivo>.ts` |
| Hook que combina várias queries | `src/features/<area>/hooks.ts` |
| Chamada externa nova (ex: enviar email) | `convex/actions/<acao>.ts` |
| Tarefa agendada | `convex/crons.ts` |
| Webhook novo (ex: integrar com cliente) | `convex/http.ts` + handler em `convex/actions/webhookProcessors/` |
| Prompt do agente | `convex/lib/prompts.ts` |
| Variável de ambiente | `src/lib/env.ts` (validaçêo Zod) + `.env.example` (documentaçêo) |
| Teste unit | `src/<file>.test.ts` (co-localizado) |
| Teste e2e | `tests/e2e/<flow>.spec.ts` (Playwright) |

## Anti-padrões

- ❌ Lógica de negócio em `app/<rota>/page.tsx` — mover pra hook ou funçêo pura.
- ❌ Fetch em `useEffect` — usa `useQuery` (Convex) ou Server Component.
- ❌ Componente em `app/<rota>/_components/` — esse padrêo é antigo. Usa `src/components/`.
- ❌ Tipo duplicado entre Convex e front — Convex já gera tipos em `_generated/api.d.ts`. Importa de lá.
