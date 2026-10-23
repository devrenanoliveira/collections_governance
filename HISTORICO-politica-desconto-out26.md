# Política de desconto — nova política (02/11/2026) e campanha de Out/26

Registro de handoff (22/10/2026): o que foi decidido, por quê, o que está no repo e o que ainda está aberto. Os valores são os de `data.json`; este arquivo explica o raciocínio.

## 1. Campanha pré-aprovada Out/26 (no ar desde 21/10)

- Cadastro: `desconto.campanhas[]` (`id: pre-aprovada-out26`). Vigência 21/10 a 30/10/2026. Atrasos de 151 dias em diante, todas as assessorias.
- Tabela (à vista / parcelado único 2x–12x): 151–180 35/25; 181–360 55/40; 361–720 73/58; 721–1080 80/65; 1081–1440 85/70; 1441–1800 87/77; 1801+ 90/85.
- Regras: entrada com vencimento até **D+5 contado da data do acordo** (vale só para a entrada; nas demais parcelas não há limite) e até 30/10. Vencimento acima disso é negado. Quebrou o acordo, **não há nova aprovação**. Ações digitais autorizadas com os descontos da tabela. "Pré Black Friday" é só uma opção de nome, não confirmada. Mora e multa seguem a política.
- A calculadora tem o checkbox "Campanha pré-aprovada", que lê `campanhas[]` (função `campanhaParaCalculadora()` em `index.html`) e só vale dentro da vigência. Não existe segunda cópia da tabela.
- **Pendente:** medir o resultado (≈ 04/11): principal recuperado por DU antes × durante, desconto médio por faixa, quebras. O D+5 não dá para verificar no relatório de Acordos. Custo exato e ponto de equilíbrio da campanha **não foram calculados**.

## 2. Nova política (aprovada pelo Renan; vigora a partir de 02/11/2026)

- Está em `desconto.politicaNova` (16 faixas, `vigenciaInicio: 2026-11-02`). A oficial (piso e teto atuais) segue vigente até lá; em 02/11 é só promover `politicaNova` a oficial.
- Formato: Fase / Dias / À vista / 2x–6x / 7x–12x / Mora / Multa. 7x–12x = 80% do 2x–6x.
- Piso à vista por faixa (5–30 … 1801+): 0,0,0,0,5,20,25,40,45,50,60,65,70,75,80,90. Piso 2x–6x: 0,0,0,0,2.5,14,18,25,30,35,45,50,55,60,70,85.
- **65–90 dias:** só mora/multa (50/50), principal 0%. **91–120:** versão **B** (5% à vista, 2,5% em 2x–6x, 2% em 7x–12x, mora/multa 100/100); o Renan confirmou que 91–120 segue o print da tabela dele. Motivo: simplificar (desconto num lado só nas faixas iniciais) sem abrir mão de cobrar encargos já aos 91 dias.
- Faixas 181–360 e 361–720 ampliadas na direção do benchmark que o Renan obteve (181–360 termina perto de 70%; 361–720 perto de 85% na referência; a nossa ficou mais conservadora).
- **Exceção (teto de aprovação), política "conservadora":** à vista 65–90 5, 91–120 15, 121–150 30, 151–180 40, 181–240 57, 241–300 66, 301–360 74, 361–540 78, 541–720 82, 721–1080 83, 1081–1440 85, 1441–1800 87, 1801+ 90. Mora/multa na exceção: 5–30 30/50, 31–60 50/80, 61–64 60/90, demais 100/100. As versões moderada e arrojada foram descartadas.
- **Parcelado na exceção:** não há tabela; a calculadora usa o "aperto" personalizado: teto = excVista × margem × (1 − aperto × (n−2)/10), nunca abaixo do piso. O Renan usa **aperto 20 e margem 90** (a pergunta "margem 100% e só aperto?" foi feita, sem decisão registrada; o padrão do arquivo é aperto 85).
- O botão **Nova (02/11)** da calculadora aplica essa política; **Atual** mantém a vigente.

## 3. Base da análise (para refazer ou auditar)

- Acordos de set/out/2026 (≈35 DU). Desconto efetivo = 1 − Principal ÷ saldo. A "exceção" mede acordos acima do teto no pior caso (cliente continua pedindo o mesmo desconto); parcelado aparece com limites 2x–6x e 7x–12x porque a distribuição de parcelas é desconhecida.
- Achado central: o piso é, na prática, o preço. De 121 dias em diante 80–90% dos acordos fecham exatamente no piso; em 65–120 fecham abaixo dele.
- Peso dos encargos (estimativa): (Saldo Atual − Saldo Contábil) ÷ Contábil na Carteira de 22/10: ≈7% (65–90), 21% (91–120), 35% (121–150), 53% (151–180). **Hipótese não verificada:** que essa diferença seja só mora/multa (a API tem `valorMora`/`valorMulta` para confirmar).
- Planilhas de apoio (Excel, **não estão no GitHub**, ficaram na área de trabalho do Renan): "Proposta - Política desconto (…)" em várias versões, "Exceção - conservadora, moderada e arrojada", "Campanha pré-aprovada - Out26 (final)". Os números relevantes estão neste arquivo e em `data.json`.

## 4. Em aberto

1. Em 02/11, promover `politicaNova` a oficial (e arquivar a antiga).
2. Medir a nova política na data (decisão do Renan: medir a partir de 02/11, sem piloto).
3. Confirmar com dado se encargos = mora + multa (peso por faixa).
4. Resultado da campanha de Out/26 e o nome "Pré Black Friday".
5. Alçada de aprovação da exceção (sugestão feita, sem decisão).
6. Exceção do parcelado: o Renan achou o teto apertado; ficou o aperto personalizado, sem regra fixa.

## 5. Onde está cada coisa

- Dado: `data.json` → `desconto.politicaNova`, `desconto.campanhas[]`.
- Tela: a política nova aparece dentro do botão "Política Oficial (Piso)" (sob "Política atual · vigente" e "Política nova"); não há botão próprio. Ao promover em 02/11, retirar a seção "Política atual".
- Código: `index.html` (`buildCalcHtml`, `limNova`, `campanhaParaCalculadora`); `calculadora-desconto.html` é gerado (regenerar ao mudar dado).
- Convenções de edição: `CLAUDE.md`.
