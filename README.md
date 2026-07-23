# Governança da Cobrança — site estático

## O que tem aqui
- `index.html` — o site (abas: Visão Geral, Política de Desconto, Régua de Cobrança, Governança de Assessorias, Fornecedores, Saúde Financeira, Atualizar Dados).
- `style.css` — estilo visual (mesmo padrão do dashboard de resultados).
- `data.json` — **fonte única dos dados**. Editar este arquivo é a única coisa necessária para atualizar o conteúdo do site.

## Como publicar (uma vez, no início)

1. Crie um repositório novo no GitHub (pode ser privado ou público — se for privado, o GitHub Pages exige plano pago para publicar; se puder ser público, fica de graça).
2. Envie estes 3 arquivos (`index.html`, `style.css`, `data.json`) para a raiz do repositório — pode arrastar e soltar direto na página do GitHub ("Add file" → "Upload files").
3. Vá em **Settings → Pages**, em "Source" selecione a branch `main` e a pasta `/ (root)`, salve.
4. Em alguns minutos o site estará no ar em `https://SEU-USUARIO.github.io/NOME-DO-REPO/`.

## Como atualizar depois (rotina)

1. No GitHub, abra o arquivo `data.json` e clique no ícone de lápis (editar).
2. Altere o trecho da seção que mudou (ex: um novo mês em `saudeFinanceira.meses`, uma nova faixa de desconto, um novo fornecedor).
3. Role até o final da página e clique em "Commit changes".
4. Espere ~1 minuto e recarregue o site — ele atualiza sozinho.

Para a Saúde Financeira (IEC), use a aba **"Atualizar Dados"** dentro do próprio site: preencha os valores do mês, ele calcula o IEC e gera o trecho de JSON pronto para colar dentro do array `"meses"` em `data.json`.

## Testar localmente antes de publicar

Não abra o `index.html` com duplo clique — o navegador bloqueia a leitura do `data.json` por segurança quando o arquivo é aberto direto do disco (`file://`). Em vez disso, dentro da pasta, rode:

```
python -m http.server 8000
```

e acesse `http://localhost:8000` no navegador. Depois de publicado no GitHub Pages isso não é mais necessário (funciona normalmente por `https://`).

## Estrutura do `data.json` (resumo)

- `desconto`: fluxo do processo + tabelas `oficial` (piso) e `agressiva` (teto) por faixa de dias — alimenta também a calculadora embutida na aba de Política de Desconto.
- `regua.atual` e `regua.desejada`: cada uma com lista de `etapas` (dias, ação, responsável) e as ferramentas/assessorias ativas naquela régua.
- `assessorias`: objetivo, estrutura de metas, tabela de comissão base, matriz de multiplicadores, comissionamento indireto, estrutura concorrencial e rituais de gestão.
- `fornecedores`: lista de objetos com `nome`, `categoria`, `papel` e `status` (`ativo` ou `em_implantacao`).
- `saudeFinanceira`: `metaIec`, lista de `linhasInvestimento` e array `meses` (cada mês com investimento por linha, total, recuperação e IEC).

Qualquer campo novo dentro desses padrões (nova linha de fornecedor, novo mês, nova faixa) você consegue adicionar sozinha só copiando a estrutura de um item existente. Se precisar de uma seção nova ou mudança de layout, aí sim volte a pedir ajuda.
