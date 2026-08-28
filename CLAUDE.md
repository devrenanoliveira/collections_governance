# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

**Governança da Cobrança — Z-ON Card**: a static, no-build-step site that documents the *governance* of the Z-ON Card collections process (discount policy, collection ladder/régua, third-party collections agency governance, vendors, and cost-efficiency health) — as opposed to the separate `zon-dashboard-enhanced` project, which covers collections *results/KPIs*.

Published via GitHub Pages at `https://devrenanoliveira.github.io/collections_governance/`. Repo: `devrenanoliveira/collections_governance`.

The site's content, UI copy, and data are entirely in **Portuguese (pt-BR)** — match that language for any user-facing text, labels, or data you add.

Esta sessão ("Kaioh do Norte", Governança de Dados cross-project do workspace) é a responsável de conteúdo por este dashboard. Qualquer outra sessão que alterar algo aqui (ex.: revisões de UX do Zarbon) deve mandar um resumo curto (o que mudou + pontos de atenção) via `SendMessage` pra quem se identificar como "Kaioh do Norte" no `ListAgents` — endereço muda com frequência, nunca hardcodar um `claude-code-XX`.

## Commands

Do not open `index.html` directly (double-click) — the browser blocks `fetch('data.json')` under `file://`. Serve it locally instead:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000`. There is no build step, package manager, linter, or test suite in this repo.

## Architecture

Five core content files, plus a growing set of standalone `estudo-*.html` study documents (currently three — see below), no framework, no bundler, **no third-party CDN dependencies** (deliberate — Chart.js via cdnjs was blocked in both dev and the user's corporate network, so all charts are hand-rolled SVG):

- **`index.html`** — the entire application: HTML markup for 8 tabs, page-specific CSS (inline `<style>`), and all JavaScript (rendering, charts, calculator, data-update form, product selector). Everything reads from the single global `DATA` object loaded from `data.json` at startup (`fetch('data.json?v=' + Date.now())`).
- **`data.json`** — single source of truth for all content. Namespaced under `produtos.<id>` (e.g. `produtos.zon`) so the site can host governance for multiple credit products; `DATA.produtoPadrao` picks which one loads by default, and a product-selector nav switches between them at runtime (no reload).
- **`style.css`** — base design system shared with sibling dashboards in the same operation (brand color tokens, KPI cards, tables, tab-nav, dark mode, print styles). Not touched for project-specific styling — that lives in `index.html`'s inline `<style>`.
- **`fluxo-whatsapp.html`** — standalone reference page (WhatsApp collections flowchart), linked from the "Régua de Cobrança" tab. Bundles the Mermaid.js library inline (~3.2 MB), which is why it's large; this is the only third-party dependency anywhere in the project, and it's isolated in this file (not loaded by `index.html`).
- **`calculadora-desconto.html`** — a standalone snapshot of the discount-approval calculator (same markup/JS `buildCalcHtml()` produces, wrapped as a full document instead of an iframe fragment), for sharing with people who shouldn't get the full governance dashboard. It embeds the current `oficial`/`agressiva` tables as literal JSON at generation time, so it has **no dependency on `data.json`** and goes stale if the discount policy changes — regenerate it (see README's "Calculadora de Aprovação — Versão Avulsa" section) after any policy edit.
- **`estudo-*.html`** (e.g. `estudo-smartnx-meta.html`, `estudo-salarial-curitiba.html`) — standalone, self-contained one-off studies/comparatives (cost scenarios, salary benchmarks, etc.), each with its own inline `<style>`/`<script>` and no shared dependency on `index.html`, `data.json`, or `style.css`. Surfaced in the "Estudos e Comparativos" tab, loaded into an `<iframe src="...">` and switched via a toggle built from `produtos.<id>.estudos` — see Rendering flow and Data model below. Because each file is fully independent, class names don't need to avoid colliding with the main dashboard's CSS.

### Rendering flow (`index.html`)

1. On load, `fetch('data.json...')` populates global `DATA`; `produtoAtual` tracks the selected product id.
2. `produto()` returns `DATA.produtos[produtoAtual]`.
3. `renderAll()` calls one `render*()` function per tab (`renderVisaoGeral`, `renderDesconto`, `renderRegua`, `renderAssessorias`, `renderFornecedores`, `renderSaude`, `renderEstudos`), each reading straight from `produto()` and writing HTML into the tab's container. Tab switching is just show/hide via `data-tab` attributes — no router.
4. Charts (`drawLineChart`, `drawDualBarChart`) are custom SVG builders driven by native JS — no charting library. They handle hover tooltips, crosshairs, and dark-mode-aware coloring internally.
5. The discount-approval calculator (in the "Política de Desconto" tab) is generated as a **self-contained HTML/JS string** and injected via `iframe srcdoc` (`buildCalcHtml(oficial, agressiva, meta)`), so its logic (`findBand`, `update()`, verdict thresholds) is separate from the main page scope — edit it inside `buildCalcHtml()`, not as top-level functions. The same function's return value is also written directly to `calculadora-desconto.html` as a standalone export (see that file's entry above) — any change to `buildCalcHtml()` should be followed by regenerating that file so the two stay in sync. Inside the generated document, "Calculadora" and "Como funciona" are two `.calc-panel` divs toggled by `.calc-tab-btn` clicks (plain show/hide, no router), and the `margem`/`aperto` policy knobs live inside a collapsed `<details class="advanced">` so day-to-day users (dias/parcelas/desconto) don't see or accidentally edit them.
6. The "Atualizar Dados" tab builds a full `data.json` in-browser from form input (notably computing IEC for Saúde Financeira) so non-technical editors never hand-edit JSON for routine monthly updates.
7. `renderEstudos()` (in the "Estudos e Comparativos" tab) builds a `.filter-btn` toggle row from `produto().estudos` and points a plain `<iframe src="...">` (`#estudoFrame`) at the selected study's `arquivo` — unlike the calculator, studies are static standalone files, not generated strings, so there's no `srcdoc`/JS-string round trip involved.

### Data model (`data.json`, per product)

- `desconto.oficial` — official discount floor by aging band (`diasDe`/`diasAte`), cash (`vista`) vs. 12x installment (`parcelado12x`) percentages, plus `mora`/`multa`.
- `desconto.agressiva` — aggressive discount ceiling by aging band, max negotiable `principal`/`mora`/`multa`.
- `regua.atual` / `regua.desejada` — collection ladder state (current vs. target), each with `internaAte`, `ferramentasInternas`, `assessoriasAtivas`, and `etapas` (steps with `dias`, `acao`, `responsavel`, optional `marco: true`).
- `regua.recursosDigitais` — reference links (flowchart, Miro board); `url: null` renders as "Em breve".
- `assessorias` — third-party agency commissioning: `comissaoBase` by aging band, `distribuicaoCarteira` (portfolio share by agency/band), `multiplicadores` (efficiency-based commission multiplier matrix), `comissaoIndireta`, `concorrencial`, `rituais`.
- `fornecedores` — vendor list: `nome`, `categoria`, `papel` (short one-line summary — also used verbatim in the Visão Geral "em implantação" banner, keep it a single readable sentence), `status` (`ativo` | `em_implantacao`), optional `topicos` (array of `{label, valor}`, added 2026-08-28 — rendered as a bullet list in the Fornecedores card in place of `papel` when present; `papel` itself is never removed, since the Visão Geral banner still reads it directly).
- `saudeFinanceira` — `metaIec`, `linhasInvestimento` (cost lines), `meses` (monthly history: `investimentoPorLinha`, `investimentoTotal`, `recuperacao`, `iec`).
- `estudos` — one-off studies list: `id` (matches the toggle button and must be unique within the product), `titulo` (toggle label), `descricao`, `arquivo` (the standalone `estudo-*.html` file to load in the iframe), `data` (shown next to the title).

To add a new product: copy an existing `produtos.<id>` block in `data.json`, give it a unique id, fill in or leave `null`/`[]`. Full field-by-field structure is documented in [README.md](README.md).

To add a new study: create a self-contained `estudo-<slug>.html` file at the repo root (no dependency on `index.html`/`data.json`/`style.css` — it must work opened directly via `file://` too), then append an entry to `produtos.<id>.estudos` in `data.json` pointing `arquivo` at it.

## Key business rules (encoded in the calculator / render logic, not just data)

- **IEC** (Índice de Eficiência de Custo) = Investimento Total ÷ Recuperação do mês; lower is better. Target: keep below the historical average (`metaIec`).
- **Discount approval verdict** (in `buildCalcHtml()`'s `update()`): proposal ≤ floor → auto-approved; floor < proposal ≤ (ceiling − 3pp) → safe range; (ceiling − 3pp) < proposal ≤ ceiling → borderline, needs review; proposal > ceiling → out of policy, needs special approval. If floor and ceiling are both 0% for the band, any requested discount is out of policy.
- **Floor/ceiling scaling by installment count** (`n` = 1–12, same `update()`): at `n=1` (à vista), floor/ceiling are the band's full `vista`/`principal` values, unscaled. At `n=2`, floor = the band's `parcelado12x` value and ceiling = `principal × margem/100` (`margem` defaults to 90%) — this is the single flat value every installment plan used to get before per-installment scaling existed, so 2x deliberately never grants more than the old policy did. From `n=2` to `n=12`, both floor and ceiling fall linearly (`t = (n-2)/10`) down to `(100 − aperto)%` of their 2x value at `n=12` (`aperto` defaults to 85%). `margem` and `aperto` are policy knobs, not proposal data — they're tucked into the collapsed "Parâmetros de política (avançado)" block precisely so they aren't confused with per-negotiation inputs.
- **Commission multiplier**: efficiency attainment below 75% for two consecutive quarters triggers automatic contract termination for the agency (this is domain knowledge reflected in copy/data, not enforced by code).
- **Indirect commission**: 30% of the band's commission is payable only if there was a valid contact attempt (CPC, delivered SMS/email, WhatsApp interaction) in the 5 days before payment; otherwise it zeroes out and the recovered amount reverts to the creditor.

## Revisão de UX (sessão "Zarbon", 28/08/2026→)

Zarbon (revisão visual/UX cross-project do workspace) está revisando este dashboard
aba a aba, a pedido do usuário. Mudanças feitas aqui devem ser reportadas a quem se
identificar como "Kaioh do Norte" no `ListAgents`, conforme a regra descrita no topo
deste arquivo.

- **Bug corrigido (28/08/2026): fonte de botão não herdava do `body`.** `<button>`/
  `<input>`/`<select>`/`<textarea>` não herdam `font-family` por padrão nos
  navegadores — todo botão do dashboard (inclusive dentro do iframe da calculadora
  de desconto) renderizava em Arial em vez da fonte do design system, mesmo com
  `body{font-family:...}` definido em `style.css`. Fix: `button, input, select,
  textarea { font-family: inherit; }` logo após o reset universal — uma linha, sem
  tocar em nenhum token/estrutura de componente (respeita o aviso no topo do
  `style.css`). Mesmo bug encontrado e corrigido no mesmo dia em
  `zon-dashboard-powered`, `acionamentos-zon` (ambas branches) e Qualidade — não era
  específico daqui; este arquivo é o template-base de onde Qualidade/Comitê de Risco
  herdaram os tokens, então vale conferir se algum dashboard futuro derivado também
  precisa do mesmo reset.
- **Observações levantadas, ainda não implementadas** (revisão em andamento, tab a
  tab): botões fragmentados em pelo menos 3 tratamentos visuais diferentes sem uma
  classe unificada (`.product-btn`/`.filter-btn` em pill 20px, `.btn`/`.btn.gold`
  retangular 6px, `.tab-btn` sem radius — mesmo problema já resolvido no
  `zon-dashboard-powered` com `.btn-primary`); ícones em emoji (🌓 modo escuro, 🖨️
  exportar PDF) em vez do padrão de ícone SVG `stroke-width="2.2"` usado nos outros 3
  dashboards; comentário no topo deste arquivo referencia `references/design-system.md`,
  que não existe em nenhum lugar do workspace.

## Known non-blocking issues

- `desconto.oficial` bands for 1081–1440, 1441–1800, and >1801 days all share `"prioridade": 14` — harmless, since that field isn't used for tie-breaking in the current UI.
- The "Quadro Miro — Régua Digital" reference link may point to a private board; users without access will see a Miro permission error.
- No automated tests are persisted in this repo (Playwright was used ad hoc during development for render/console/screenshot checks, but nothing is checked in).
