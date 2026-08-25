# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

**Governança da Cobrança — Z-ON Card**: a static, no-build-step site that documents the *governance* of the Z-ON Card collections process (discount policy, collection ladder/régua, third-party collections agency governance, vendors, and cost-efficiency health) — as opposed to the separate `zon-dashboard-enhanced` project, which covers collections *results/KPIs*.

Published via GitHub Pages at `https://devrenanoliveira.github.io/collections_governance/`. Repo: `devrenanoliveira/collections_governance`.

The site's content, UI copy, and data are entirely in **Portuguese (pt-BR)** — match that language for any user-facing text, labels, or data you add.

## Commands

Do not open `index.html` directly (double-click) — the browser blocks `fetch('data.json')` under `file://`. Serve it locally instead:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000`. There is no build step, package manager, linter, or test suite in this repo.

## Architecture

Four content files, no framework, no bundler, **no third-party CDN dependencies** (deliberate — Chart.js via cdnjs was blocked in both dev and the user's corporate network, so all charts are hand-rolled SVG):

- **`index.html`** — the entire application: HTML markup for 7 tabs, page-specific CSS (inline `<style>`), and all JavaScript (rendering, charts, calculator, data-update form, product selector). Everything reads from the single global `DATA` object loaded from `data.json` at startup (`fetch('data.json?v=' + Date.now())`).
- **`data.json`** — single source of truth for all content. Namespaced under `produtos.<id>` (e.g. `produtos.zon`) so the site can host governance for multiple credit products; `DATA.produtoPadrao` picks which one loads by default, and a product-selector nav switches between them at runtime (no reload).
- **`style.css`** — base design system shared with sibling dashboards in the same operation (brand color tokens, KPI cards, tables, tab-nav, dark mode, print styles). Not touched for project-specific styling — that lives in `index.html`'s inline `<style>`.
- **`fluxo-whatsapp.html`** — standalone reference page (WhatsApp collections flowchart), linked from the "Régua de Cobrança" tab. Bundles the Mermaid.js library inline (~3.2 MB), which is why it's large; this is the only third-party dependency anywhere in the project, and it's isolated in this file (not loaded by `index.html`).

### Rendering flow (`index.html`)

1. On load, `fetch('data.json...')` populates global `DATA`; `produtoAtual` tracks the selected product id.
2. `produto()` returns `DATA.produtos[produtoAtual]`.
3. `renderAll()` calls one `render*()` function per tab (`renderVisaoGeral`, `renderDesconto`, `renderRegua`, `renderAssessorias`, `renderFornecedores`, `renderSaude`), each reading straight from `produto()` and writing HTML into the tab's container. Tab switching is just show/hide via `data-tab` attributes — no router.
4. Charts (`drawLineChart`, `drawDualBarChart`) are custom SVG builders driven by native JS — no charting library. They handle hover tooltips, crosshairs, and dark-mode-aware coloring internally.
5. The discount-approval calculator (in the "Política de Desconto" tab) is generated as a **self-contained HTML/JS string** and injected via `iframe srcdoc` (`buildCalcHtml()`), so its logic (`findBand`, `update()`, verdict thresholds) is separate from the main page scope — edit it inside `buildCalcHtml()`, not as top-level functions.
6. The "Atualizar Dados" tab builds a full `data.json` in-browser from form input (notably computing IEC for Saúde Financeira) so non-technical editors never hand-edit JSON for routine monthly updates.

### Data model (`data.json`, per product)

- `desconto.oficial` — official discount floor by aging band (`diasDe`/`diasAte`), cash (`vista`) vs. 12x installment (`parcelado12x`) percentages, plus `mora`/`multa`.
- `desconto.agressiva` — aggressive discount ceiling by aging band, max negotiable `principal`/`mora`/`multa`.
- `regua.atual` / `regua.desejada` — collection ladder state (current vs. target), each with `internaAte`, `ferramentasInternas`, `assessoriasAtivas`, and `etapas` (steps with `dias`, `acao`, `responsavel`, optional `marco: true`).
- `regua.recursosDigitais` — reference links (flowchart, Miro board); `url: null` renders as "Em breve".
- `assessorias` — third-party agency commissioning: `comissaoBase` by aging band, `distribuicaoCarteira` (portfolio share by agency/band), `multiplicadores` (efficiency-based commission multiplier matrix), `comissaoIndireta`, `concorrencial`, `rituais`.
- `fornecedores` — vendor list: `nome`, `categoria`, `papel`, `status` (`ativo` | `em_implantacao`).
- `saudeFinanceira` — `metaIec`, `linhasInvestimento` (cost lines), `meses` (monthly history: `investimentoPorLinha`, `investimentoTotal`, `recuperacao`, `iec`).

To add a new product: copy an existing `produtos.<id>` block in `data.json`, give it a unique id, fill in or leave `null`/`[]`. Full field-by-field structure is documented in [README.md](README.md).

## Key business rules (encoded in the calculator / render logic, not just data)

- **IEC** (Índice de Eficiência de Custo) = Investimento Total ÷ Recuperação do mês; lower is better. Target: keep below the historical average (`metaIec`).
- **Discount approval verdict** (in `buildCalcHtml()`'s `update()`): proposal ≤ floor → auto-approved; floor < proposal ≤ (ceiling − 3pp) → safe range; (ceiling − 3pp) < proposal ≤ ceiling → borderline, needs review; proposal > ceiling → out of policy, needs special approval. If floor and ceiling are both 0% for the band, any requested discount is out of policy. For installment proposals, the effective ceiling is reduced by a configurable margin (default 90%).
- **Commission multiplier**: efficiency attainment below 75% for two consecutive quarters triggers automatic contract termination for the agency (this is domain knowledge reflected in copy/data, not enforced by code).
- **Indirect commission**: 30% of the band's commission is payable only if there was a valid contact attempt (CPC, delivered SMS/email, WhatsApp interaction) in the 5 days before payment; otherwise it zeroes out and the recovered amount reverts to the creditor.

## Known non-blocking issues

- `desconto.oficial` bands for 1081–1440, 1441–1800, and >1801 days all share `"prioridade": 14` — harmless, since that field isn't used for tie-breaking in the current UI.
- The "Quadro Miro — Régua Digital" reference link may point to a private board; users without access will see a Miro permission error.
- No automated tests are persisted in this repo (Playwright was used ad hoc during development for render/console/screenshot checks, but nothing is checked in).
