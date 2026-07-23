# vindi-sales-dashboard (Vindi Sales Dashboard — Better Education)

Dashboard Next.js para visualizar as vendas do Better Education pela API Vindi, com crossmatch de dados reais e visualização de discrepâncias. É o antecessor/base do `vindi-dashboard-v2-` (Alumni Dashboard Enhanced) — compartilha grande parte do código.

## Stack

- **Linguagem:** TypeScript 5 (`strict: true`) + alguns scripts utilitários em JavaScript.
- **Framework:** Next.js 14 (Pages Router — `pages/`), React 18.
- **UI:** Tailwind CSS 3 (`@tailwindcss/forms`), `lucide-react`, `recharts` (gráficos), `class-variance-authority` + `clsx` + `tailwind-merge`.
- **HTTP:** `node-fetch` para chamar a API Vindi.
- **Banco:** nenhum banco relacional — dados vêm da API Vindi (e crossmatch com planilha) em runtime.
- **Deploy:** Vercel (`vercel.json`, `framework: nextjs`).
- **Package manager:** npm (`package-lock.json`).

## Comandos

```bash
npm install
npm run dev     # Next dev em http://localhost:3000
npm run build   # build de produção
npm run start
npm run lint    # next lint
```

Não há `type-check` no `package.json` — rode `npx tsc --noEmit` para checar tipos. Scripts avulsos na raiz (`test-*.js`, `debug-*.js`, `crossmatch-analyzer.js`) são exploratórios e rodam com `node <arquivo>.js`.

## Estrutura

- `pages/` — Pages Router: `index.tsx`, `working.tsx`, `minimal.tsx` (variantes de UI), `_app.tsx`.
- `pages/api/dashboard-data-v2.ts` — endpoint principal que busca dados na Vindi e monta o crossmatch.
- `src/lib/utils.ts` — utilitários (`cn()` etc.).
- `src/components/` — `ClientDashboard.tsx` (UI principal), `ErrorBoundary.tsx`, `ui/` (card, alert).
- `src/hooks/useDashboardData.ts`, `src/types/dashboard.ts`, `src/styles/globals.css`.
- Raiz: scripts de debug/teste (`test-api.js`, `crossmatch-analyzer.js`, `debug-cpf-matching.js`, etc.).
- `README.md`, `PROJECT_STATUS.md`.

## Convenções de código

- TypeScript `strict`; alias `@/*` → `./src/*`.
- ESLint via `next lint` (`eslint-config-next`, sem `.eslintrc` custom). Rode `npm run lint` antes de commitar.
- Código novo em TypeScript dentro de `src/`/`pages/`; os `*.js` da raiz são utilitários legados — evite crescê-los.
- Estilo com Tailwind; use `cn()` para compor classes. Gráficos com `recharts`.

## Variáveis de ambiente

Configurar em `.env.local` (dev) e em **Vercel → Environment Variables** (prod). Nunca commitar valores. Nomes usados:

| Nome | Para quê |
| --- | --- |
| `VINDI_API_KEY` | Auth na API Vindi (secret) |
| `VINDI_API_URL` | Base da API (`https://app.vindi.com.br/api/v1`, ou sandbox) |

Ver `.env.example`. Atenção: `.gitignore` ignora `.env` e `.env*.local`.

## CI/CD & Deploy

- **Deploy:** Vercel com auto-deploy — push na `main` publica produção; PRs geram Previews. Basta configurar `VINDI_API_KEY` no projeto.
- **CI:** não há workflows em `.github/`. Recomendação (PR): `npm ci` → `npm run lint` → `npx tsc --noEmit` → `npm run build`.

## Boas práticas de PR

- Branches: `feat/…`, `fix/…`, `chore/…`. Conventional Commits.
- PRs pequenos e focados. Checklist:
  - [ ] `npm run build` e `npm run lint` passam; `npx tsc --noEmit` sem erros.
  - [ ] Sem `VINDI_API_KEY` ou dados reais no diff (inclusive nos `test-*.js`).
  - [ ] Screenshots em mudanças de dashboard/UI/gráficos.
- ≥1 review; squash merge; `main` sempre deployável.

## Testes

Sem suíte formal — apenas scripts manuais na raiz executados com `node` (ex.: `test-crossmatch.js`, `test-cpf-normalization.js`, `test-email-matching.js`). Ao alterar a lógica de crossmatch/normalização, valide com esses scripts (dados anonimizados); mínimo proporcional seria promovê-los a testes unitários (Vitest/Jest).

## Segurança & dados

- Trafega **dados pessoais reais** de clientes (nomes, CPF/CNPJ, e-mails, pagamentos) — sensível sob LGPD. Nunca commitar exports/dumps nem a chave Vindi.
- `VINDI_API_KEY` só em env vars; nunca em código, README ou nos scripts de teste.
- Revisar deps periodicamente (`npm audit`).

## Gotchas

- Várias páginas de dashboard coexistem (`index`, `working`, `minimal`) — identifique a rota "oficial" antes de refatorar.
- Não há cache/banco: cada requisição bate na Vindi. Cuidado com rate-limit e latência ao adicionar features.
- **Sobreposição forte com `vindi-dashboard-v2-`** (mesmos scripts de debug/teste e UI base). Este repo é a versão mais enxuta; o v2 adiciona cache local, Express backend e endpoint `dashboard-data-v3`. Correções de lógica costumam valer para ambos.
- Não há `.eslintrc` custom — o lint depende do config default do Next.
