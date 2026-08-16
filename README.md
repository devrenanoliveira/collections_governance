# Governança da Cobrança — README Completo

## Conteúdo do Repositório

O projeto consiste em quatro arquivos principais:

- **index.html**: O site com sete abas (Visão Geral, Política de Desconto, Régua de Cobrança, Governança de Assessorias, Fornecedores, Saúde Financeira, Atualizar Dados)
- **style.css**: Estilo visual consistente com o dashboard de resultados
- **data.json**: Fonte única dos dados — toda atualização passa por este arquivo
- **fluxo-whatsapp.html**: Página com fluxograma detalhado de atendimento via WhatsApp

## Publicação Inicial

1. Criar repositório no GitHub (público para GitHub Pages gratuito)
2. Fazer upload dos quatro arquivos para a raiz
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

O site detecta automaticamente todos os produtos em `data.json` e exibe um seletor. Cada produto tem as mesmas sete abas e governança independente.

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

Novos itens (fornecedores, meses, faixas, produtos) podem ser adicionados copiando estruturas existentes sem modificar código.

## Aba Saúde Financeira — Seletor de Mês de Referência

Ao lado do título da aba há um seletor **"Mês de referência"**, populado automaticamente com todos os meses cadastrados em `saudeFinanceira.meses` do produto atual. Por padrão ele abre no mês mais recente, mas pode ser trocado para qualquer mês do histórico.

Trocar o mês recalcula dinamicamente:

- Os 4 cards de KPI no topo (IEC do mês, Meta de IEC, Investimento e Recuperação do mês)
- O comparativo "vs. mês anterior" no card de IEC, que passa a comparar com o mês imediatamente anterior ao selecionado
- O destaque visual (coluna em azul claro) na tabela "Investimento por linha, por mês", marcando a coluna do mês escolhido

Os gráficos de tendência (IEC mensal vs. meta e Investimento vs. Recuperação) continuam mostrando sempre a série histórica completa, independentemente do mês selecionado — o seletor afeta apenas os cards e o destaque da tabela.
