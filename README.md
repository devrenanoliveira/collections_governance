# Governança da Cobrança — README Completo

## Conteúdo do Repositório

O projeto consiste em cinco arquivos principais, mais um conjunto de estudos avulsos que cresce com o tempo:

- **index.html**: O site com oito abas (Visão Geral, Política de Desconto, Régua de Cobrança, Governança de Assessorias, Fornecedores, Saúde Financeira, Estudos e Comparativos, Atualizar Dados)
- **style.css**: Estilo visual consistente com o dashboard de resultados
- **data.json**: Fonte única dos dados — toda atualização passa por este arquivo
- **fluxo-whatsapp.html**: Página com fluxograma detalhado de atendimento via WhatsApp
- **calculadora-desconto.html**: Versão avulsa da Calculadora de Aprovação de Negociação (ver seção própria abaixo)
- **estudo-\*.html** (ex.: `estudo-smartnx-meta.html`, `estudo-salarial-curitiba.html`): estudos e comparativos avulsos, um arquivo por estudo (ver seção "Estudos e Comparativos Gerais" abaixo)

## Calculadora de Aprovação — Versão Avulsa

`calculadora-desconto.html` é a mesma calculadora da aba "Política de Desconto", mas exportada como um arquivo único e autônomo — sem depender do `index.html`, do `data.json` ou de conexão com a internet. Pode ser aberta com duplo clique e enviada para qualquer pessoa (assessoria, aprovador, etc.) sem dar acesso ao dashboard completo de governança.

Ela tem duas abas: **Calculadora** (uso do dia a dia — dias em atraso, parcelas, desconto proposto) e **Como funciona** (explica piso/teto e o escalonamento por número de parcelas, para quem não tem contexto do dashboard). Os parâmetros de política (margem e aperto) ficam num bloco recolhido "Parâmetros de política (avançado)", dentro da aba Calculadora, para não serem alterados sem querer.

**Importante**: os valores de piso/teto ficam embutidos no arquivo no momento da exportação — se a política de desconto mudar no `data.json`, este arquivo fica desatualizado até ser gerado de novo. Para regenerar, no console do navegador com o `index.html` aberto (ou pedindo para o Claude Code):

```js
const p = DATA.produtos.zon;
buildCalcHtml(p.desconto.oficial, p.desconto.agressiva, {produto: p.nome, ultimaAtualizacao: p.ultimaAtualizacao});
```

O retorno é o HTML completo do arquivo — salve como `calculadora-desconto.html`.

## Estudos e Comparativos Gerais

A aba **"Estudos e Comparativos"** reúne análises pontuais de cenário e comparativos de custo que não fazem parte do ciclo mensal de governança (política de desconto, régua, assessorias, saúde financeira) — cada estudo é um documento HTML independente, com seu próprio design e sem dependência do `data.json`, exibido dentro da aba via `<iframe>` e alternado por um toggle.

Estudos atuais:

- **SmartNX × Novo Modelo Meta** (`estudo-smartnx-meta.html`): comparativo de custo de Atendimento e Cobrança diante do novo tarifário da Meta e da proposta da SmartNX.
- **Estudo Salarial — Curitiba** (`estudo-salarial-curitiba.html`): bases salariais de referência para Operador de Call Center e Operador de Cobrança, jornadas de 6h e 8h.
- **Meta de Recuperação de Perdas 2027** (`estudo-meta-perdas-2027.html`): confronto entre a meta do diretor para recuperação de perdas (Pós-Prejuízo, faixas H–J) e a projeção construída a partir do histórico real de pagamento, para embasar a meta de PLR de 2027.

### Adicionar Novo Estudo

1. Criar o arquivo `estudo-<nome>.html` na raiz do repositório — precisa ser autocontido (CSS/JS inline, sem depender de `index.html`, `data.json` ou `style.css`) para funcionar tanto dentro do iframe quanto aberto sozinho.
2. No `data.json`, dentro de `produtos.<id>.estudos`, adicionar um item novo: `id` (único), `titulo` (aparece no toggle), `descricao`, `arquivo` (nome do arquivo do passo 1) e `data`.
3. Publicar os dois arquivos — o estudo aparece automaticamente como um novo toggle na aba.

## Publicação Inicial

1. Criar repositório no GitHub (público para GitHub Pages gratuito)
2. Fazer upload dos arquivos do projeto para a raiz
3. Em Settings → Pages, selecionar branch `main` e pasta `/ (root)`
4. Aguardar alguns minutos; o site fica disponível em `https://SEU-USUARIO.github.io/NOME-DO-REPO/`

## Atualizações Rotineiras

Editar o arquivo `data.json` diretamente no GitHub:

1. Abrir `data.json` e clicar no ícone de lápis
2. Alterar o trecho necessário (novo mês, faixa de desconto, fornecedor)
3. Fazer commit
4. Recarregar o site após ~1 minuto

Para atualizações de Saúde Financeira, usar a aba **"Atualizar Dados"** do próprio site, que calcula o IEC e gera o JSON pronto.

## Testes Locais

Não abrir `index.html` com duplo clique — o navegador bloqueia `data.json` por segurança. Em vez disso, na pasta do projeto, executar:

```
python -m http.server 8000
```

Acessar `http://localhost:8000`. Após publicação no GitHub Pages, funciona normalmente via HTTPS.

## Suporte a Múltiplos Produtos

O site detecta automaticamente todos os produtos em `data.json` e exibe um seletor. Cada produto tem as mesmas oito abas e governança independente.

### Adicionar Novo Produto

1. Abrir `data.json` e localizar a chave `"produtos"`
2. Copiar um bloco completo existente (ex: `"zon"`)
3. Colar como novo item com id único (ex: `"produtob"`)
4. Alterar o campo `"nome"` para o nome real
5. Preencher as seções ou deixar como `null`/`[]` se não estiverem prontas
6. Opcionalmente, atualizar `"produtoPadrao"` para definir qual abre por padrão

## Estrutura do data.json

Cada produto contém:

- **nome**: Identificação no seletor e cabeçalho
- **desconto**: Processo, tabelas oficial (piso) e agressiva (teto) por dias
- **regua.recursosDigitais**: Links de referência (fluxogramas, quadros); `url: null` exibe "Em breve"
- **regua.atual e regua.desejada**: Etapas, dias, ações, responsáveis e ferramentas
- **assessorias**: Metas, comissão base, multiplicadores, estrutura concorrencial, rituais
- **fornecedores**: Nome, categoria, papel, status (ativo ou em_implantacao)
- **saudeFinanceira**: Meta IEC, linhas de investimento, meses com investimento, recuperação e IEC
- **estudos**: Lista de estudos avulsos — id, titulo, descricao, arquivo (o HTML standalone) e data

Novos itens (fornecedores, meses, faixas, produtos, estudos) podem ser adicionados copiando estruturas existentes sem modificar código.

## Aba Saúde Financeira — Seletor de Mês de Referência

Ao lado do título da aba há um seletor **"Mês de referência"**, populado automaticamente com todos os meses cadastrados em `saudeFinanceira.meses` do produto atual. Por padrão ele abre no mês mais recente, mas pode ser trocado para qualquer mês do histórico.

Trocar o mês recalcula dinamicamente:

- Os 4 cards de KPI no topo (IEC do mês, Meta de IEC, Investimento e Recuperação do mês)
- O comparativo "vs. mês anterior" no card de IEC, que passa a comparar com o mês imediatamente anterior ao selecionado
- O destaque visual (coluna em azul claro) na tabela "Investimento por linha, por mês", marcando a coluna do mês escolhido

Os gráficos de tendência (IEC mensal vs. meta e Investimento vs. Recuperação) continuam mostrando sempre a série histórica completa, independentemente do mês selecionado — o seletor afeta apenas os cards e o destaque da tabela.
