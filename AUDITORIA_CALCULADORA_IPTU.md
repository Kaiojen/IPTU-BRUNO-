# RELATÓRIO DE AUDITORIA TÉCNICA — CALCULADORA IPTU VÉRTICE

**Data da auditoria:** 18/05/2026  
**Arquivo auditado:** `index.html`  
**Linhas:** 1.582  
**Autor do relatório:** Análise técnica automatizada

---

## 1. LOCALIZAÇÃO DA CALCULADORA

### Arquivos do projeto

O repositório contém **apenas dois arquivos relevantes**:

| Arquivo | Tamanho | Função |
|---|---|---|
| `index.html` | 1.582 linhas | Todo o site: HTML, CSS e JavaScript em arquivo único |
| `README.md` | Mínimo | Documentação genérica do projeto |

Não há framework, não há bundler, não há backend, não há banco de dados. É um arquivo HTML estático puro.

### Distribuição interna do `index.html`

| Seção | Linhas | Função |
|---|---|---|
| `<head>` + estilos | 1–133 | Configuração Tailwind, fontes, CSS customizado |
| Header | 138–189 | Navegação fixada no topo |
| Hero | 191–273 | Título principal + CTA |
| Transparency Bar | 275–285 | Aviso de que não há promessa de redução |
| Stats | 287–311 | Números de social proof |
| **Calculadora** | **313–618** | Formulário + 5 telas de resultado |
| Indicators | 620–677 | Quando solicitar auditoria |
| Parameters Auditados | 679–779 | O que a Vértice analisa |
| Processo | 781–832 | 4 passos metodológicos |
| Público / Entregáveis | 834–925 | Perfil de atendimento |
| Depoimentos | 927–1000 | 3 testemunhos |
| Authority Banner | 1002–1046 | Credenciais CREA/ART |
| FAQ | 1048–1125 | 5 perguntas |
| Formulário de Contato | 1127–1241 | Captação de lead |
| Footer | 1243–1312 | Rodapé |
| WhatsApp Flutuante | 1314–1323 | Botão fixo |
| **JavaScript** | **1327–1579** | Toda a lógica da calculadora |

### Rota/acesso

- Página única (`index.html`), seção `#triagem`
- Não há roteamento, não há SPA, não há Next.js

---

## 2. FLUXO ATUAL DO USUÁRIO

```
ENTRADA
└── Usuário acessa #triagem
    └── Preenche 4 campos obrigatórios + até 5 opcionais
        └── Clica em "Verificar Indícios Técnicos"

PROCESSAMENTO
└── runCalc() executa 5 verificações independentes
    └── Calcula score de 0 a ≥4
        └── Score determina nível (1 a 4)

RESULTADO
├── Outro município → tela "Fora da área de atuação"
├── Score 0 → Nível 1: "Sem indício claro"
├── Score 1 → Nível 2: "Precisamos de mais dados"
├── Score 2–3 → Nível 3: "Possível inconsistência"
└── Score ≥4 → Nível 4: "Alto potencial de revisão"

PRÓXIMA AÇÃO
├── Nível 1 → WhatsApp "enviar carnê para conferência"
├── Nível 2 → WhatsApp "falar com especialista"
├── Nível 3 → Formulário de análise técnica + WhatsApp
└── Nível 4 → WhatsApp urgente + Formulário formal
```

**Distinção entre resultados:** Existem 4 níveis distintos de resposta, cada um com ícone, cor e CTA diferentes. Não existe resultado explicitamente "negativo" — o nível 1 já convida a enviar o carnê mesmo sem indício. Não existe resultado "inconclusivo" formal; o nível 2 é o mais próximo disso.

---

## 3. DADOS DE ENTRADA ATUAIS

| Campo | ID | Obrigatório? | Tipo | Finalidade aparente | Usado no cálculo? | Observações |
|---|---|---|---|---|---|---|
| Tipo do Imóvel | `c_tipo` | ✅ Sim | Select | Determinar alíquota | ✅ Sim | Mapeia para tabela RATES |
| Município | `c_municipio` | ✅ Sim | Select | Validar escopo geográfico | ✅ Sim | Se "outro" → encerra sem calcular |
| Valor Anual IPTU | `c_iptu` | ✅ Sim | Number | Valor pago pelo contribuinte | ✅ Sim | Input numérico sem formatação |
| Valor Venal | `c_venal` | ✅ Sim | Number | Base de cálculo esperada | ✅ Sim | Input numérico sem formatação |
| Área Cadastrada | `c_area_cad` | ❌ Não | Number | Confrontar com área real | ✅ Condicional | Só usado se ambas as áreas forem preenchidas |
| Área Real | `c_area_real` | ❌ Não | Number | Área física declarada | ✅ Condicional | Só usado se ambas as áreas forem preenchidas |
| Aumento brusco | `c_aumento` | ❌ Não | Select | Sinalizar reajuste anormal | ✅ Sim | Apenas "sim" gera score |
| Reforma/mudança | `c_reforma` | ❌ Não | Select | Sinalizar alteração física | ✅ Sim | Apenas "sim" gera score |
| Documentação | `c_docs` | ❌ Não | Select | Checar disponibilidade | ⚠️ Parcial | Afeta apenas mensagem informativa |

### Campos que sobram (existem, mas contribuição é marginal)

- `c_docs`: Só gera um texto informativo se `score > 0` e `docs === 'nao'`. Não afeta o score, não muda o nível. Sua presença é essencialmente decorativa.

### Campos que faltam e seriam necessários

- **Inscrição imobiliária** — identificador único do imóvel para consulta futura
- **Bairro / Zona** — essencial pois o valor venal unitário varia enormemente por localização
- **Ano de construção** — a Prefeitura aplica fator de vetustez (depreciação pela idade), mencionado no próprio site como parâmetro auditado, mas ausente na calculadora
- **Padrão construtivo** (popular / normal / médio / alto / luxo) — determina o VVU (Valor Venal Unitário) e é um dos erros mais frequentes
- **TCL separado** — a calculadora compara "IPTU informado no carnê" com "IPTU esperado", mas o carnê contém IPTU + TCL (Taxa de Coleta de Lixo) juntos
- **Nome e contato do usuário** — o formulário de triagem não captura lead; apenas o formulário de contato separado faz isso

### Campos mal nomeados ou que podem induzir ao erro

- **"Valor Anual do IPTU no Carnê":** o placeholder é `45.000`, sugerindo que o usuário deve digitar sem `R$`. O label diz "IPTU no Carnê" — usuário pode inserir o total do carnê (IPTU + TCL + outros encargos) sem saber separar. Isso gera distorção direta no cálculo.

---

## 4. LÓGICA DE CÁLCULO IMPLEMENTADA HOJE

### Tabela de alíquotas hardcoded (linhas 1364–1371)

```javascript
const RATES = {
    residencial: 0.012,   // 1,2%
    comercial:   0.025,   // 2,5%
    galpao:      0.015,   // 1,5%
    condominio:  0.020,   // 2,0%
    terreno:     0.030,   // 3,0%
    misto:       0.020    // 2,0%
};
```

### Fórmula central (linha 1419)

```
IPTU_esperado = valor_venal × alíquota_do_tipo
divergência% = ((IPTU_informado − IPTU_esperado) / IPTU_esperado) × 100
```

### Regras implementadas (checagens)

| Regra no código | Onde | Como funciona | Risco ou observação |
|---|---|---|---|
| IPTU vs esperado (diff > 50%) | L. 1428 | Score +3, flag "divergência expressiva" | Alíquota fixa ignora progressividade real |
| IPTU vs esperado (30% < diff ≤ 50%) | L. 1434 | Score +2, flag "inconsistência moderada" | Mesmo risco acima |
| Área cadastrada > área real (+20%) | L. 1448 | Score +3, flag "divergência significativa" | Lógica correta conceitualmente |
| Área cadastrada > área real (+10%) | L. 1454 | Score +1, flag menor | OK |
| Área real > área cadastrada (> 10%) | L. 1460 | Score +1, flag invertida | Seria favorável ao contribuinte, não sobrecobrança |
| Aumento brusco nos últimos 2 anos | L. 1470 | Score +1, flag | Apenas sinaliza; sem quantificação |
| Reforma/ampliação/demolição | L. 1480 | Score +1, flag | Apenas sinaliza; sem direção do impacto |
| Sem documentação (se score > 0) | L. 1489 | Apenas texto informativo | Não afeta score |

### Score → Nível

| Score | Nível | Título exibido |
|---|---|---|
| 0 | 1 | Sem indício claro |
| 1 | 2 | Precisamos de mais dados |
| 2–3 | 3 | Possível inconsistência |
| ≥ 4 | 4 | Alto potencial de revisão |

### Valores hardcoded completos

- Alíquotas: 1,2% / 2,5% / 1,5% / 2,0% / 3,0% / 2,0%
- Threshold de divergência expressiva: **50%**
- Threshold de divergência moderada: **30%**
- Threshold de divergência de área (grande): **20%**
- Threshold de divergência de área (pequena): **10%**

---

## 5. REGRAS LEGAIS QUE A CALCULADORA PARECE TENTAR APLICAR

| Conceito legal | Estado no código | Completo? | Risco |
|---|---|---|---|
| Alíquota diferenciada por uso (residencial vs comercial vs terreno) | Implementado via `RATES` | ❌ Incompleto — usa alíquota única por tipo, sem progressividade | Moderado: gera falsos negativos para imóveis de valor médio-alto |
| Alíquota progressiva por faixa de valor venal | **Ausente** | ❌ Não existe | Alto: o IPTU-RJ tem alíquotas em faixas; usar taxa fixa distorce o cálculo base |
| Divergência de área como fundamento de revisão | Implementado | ✅ Conceitualmente correto | Baixo: lógica faz sentido, mas depende da área declarada ser confiável |
| Fator de vetustez (depreciação pela idade) | **Ausente** | ❌ Não existe | Alto: mencionado no site como parâmetro auditado, mas não avaliado |
| TCL (Taxa de Coleta de Lixo) como item separado do carnê | **Ausente** | ❌ Não existe | Crítico: usuário provavelmente insere total do carnê (IPTU+TCL), inflando artificialmente a divergência |
| Isenção de IPTU (aposentados, baixa renda, etc.) | **Ausente** | ❌ Não existe | Alto para MVP futuro: é objetivo declarado da Vert |
| Alíquota majorada para terreno ocioso (função social) | Parcial — terreno tem 3% | ⚠️ Aproximado | Moderado: a progressividade de terrenos é ainda mais complexa no RJ |
| Dispensa de cobrança por imóvel de baixo valor venal | **Ausente** | ❌ Não existe | Alto para triagem de isenção |
| Padrão construtivo como fator de cálculo | **Ausente** | ❌ Não existe | Alto: é um dos erros mais comuns segundo o próprio site |

---

## 6. RESULTADOS GERADOS PELA CALCULADORA

| Resultado | Condição | Texto principal | Problema potencial |
|---|---|---|---|
| Fora da área | `municipio === 'outro'` | "Atendemos exclusivamente o município do Rio de Janeiro" | Nenhum — correto e adequado |
| Nível 1 — Sem indício | `score === 0` | "Sem indício claro pelos dados informados" | Gera falsa tranquilidade: vetustez, padrão construtivo e TCL não são verificados |
| Nível 2 — Mais dados | `score === 1` | "Precisamos de mais dados para concluir" | Nome enganoso: score=1 pode ser de um flag real (aumento brusco), não de dados faltando |
| Nível 3 — Inconsistência | `score 2–3` | "Possível inconsistência identificada" | Adequado em tom; pode ser falso positivo se IPTU < esperado |
| Nível 4 — Alto potencial | `score ≥ 4` | "Caso com forte potencial para análise técnica" | Adequado em tom; linguagem prudente — não promete resultado |

### Análise dos textos dos resultados

**Pontos positivos:**
- Nenhum resultado afirma categoricamente que há erro
- Nenhum resultado promete redução
- O aviso legal na base de todos os resultados é claro (linhas 607–611)
- O nível 1 explicitamente diz "isso não significa que não existam inconsistências" — honesto

**Problemas:**
- Nível 2 usa o rótulo "precisamos de mais dados" mesmo quando há flags reais (score=1 de aumento brusco)
- O nível 1 direciona para WhatsApp com CTA "Enviar Carnê para Conferência" — converte qualquer resultado (mesmo negativo) em lead
- O sistema flagga IPTU menor que esperado com o mesmo alerta de IPTU maior — a direção da divergência é relevante para o modelo de negócio

---

## 7. TESTES PRÁTICOS SIMULADOS

Executados como cálculo manual a partir da lógica do código (sem alterar nada):

| # | Cenário | Dados inseridos | Resultado calculado | Coerente? | Observação |
|---|---|---|---|---|---|
| 1 | Residencial baixo valor | tipo=residencial, IPTU=R$1.200, venal=R$100.000 | Expected=R$1.200, diff=0%, score=0 → **Nível 1** | ✅ Sim | Correto para os dados; não avalia isenção |
| 2 | Residencial alto valor | tipo=residencial, IPTU=R$15.000, venal=R$1.000.000 | Expected=R$12.000, diff=+25%, score=0 → **Nível 1** | ⚠️ Parcial | Se alíquota real for 1,4% para essa faixa, expected cai para R$14.000 e diff é 7%. Mas se alíquota for 1%, o imóvel paga a mais e o sistema não detecta. **Falso negativo potencial.** |
| 3 | Comercial normal | tipo=comercial, IPTU=R$45.000, venal=R$1.800.000 | Expected=R$45.000, diff=0%, score=0 → **Nível 1** | ✅ Sim | Mas se carnê inclui TCL, real IPTU seria menor e score ficaria errado |
| 4 | Terreno | tipo=terreno, IPTU=R$30.000, venal=R$1.000.000 | Expected=R$30.000, diff=0%, score=0 → **Nível 1** | ⚠️ Parcial | 3% pode não ser a alíquota correta — terrenos têm progressividade em RJ |
| 5 | Venal baixo, IPTU alto | tipo=residencial, IPTU=R$5.000, venal=R$200.000 | Expected=R$2.400, diff=+108%, score=3 → **Nível 3** | ✅ Sim | Detecta corretamente possível supercobrança |
| 6 | Venal alto, IPTU baixo | tipo=residencial, IPTU=R$1.000, venal=R$1.000.000 | Expected=R$12.000, diff=−91,7%, score=3 → **Nível 3** | ❌ Não | IPTU menor que esperado é FAVORÁVEL ao contribuinte. O sistema gera "possível inconsistência" e direciona para análise — mas a análise confirmaria que o contribuinte paga menos do que deveria. **Direcionamento equivocado.** |
| 7 | Dados insuficientes (só obrigatórios, flags opcionais vazios) | tipo=galpao, IPTU=R$30.000, venal=R$2.000.000, sem área nem flags | Expected=R$30.000, diff=0%, score=0 → **Nível 1** | ⚠️ Parcial | O galpão do depoimento (28% de divergência de área) nunca seria detectado aqui |
| 8 | Fora do Rio | municipio=outro | → **Fora da área** | ✅ Sim | Correto e limpo |
| 9 | Galpão com divergência de área | tipo=galpao, IPTU=R$80.000, venal=R$3.000.000, área_cad=1240m², área_real=890m² | Expected=R$45.000, diff=+77,8% (score+3); areaPct=28,2% (score+3); total=6 → **Nível 4** | ✅ Sim | Funciona bem para o caso clássico com dados completos |
| 10 | Possível isenção (residencial, baixo valor) | tipo=residencial, IPTU=R$2.000, venal=R$150.000 | Expected=R$1.800, diff=+11%, score=0 → **Nível 1** | ❌ Não | Um imóvel que talvez devesse ser isento recebe "sem indício claro". **Isenção jamais é avaliada.** |

---

## 8. ERROS, INCONSISTÊNCIAS E LIMITAÇÕES ATUAIS

### A. Erros críticos

**A1. Ausência total de lógica de isenção**

O objetivo declarado da Vert inclui "identificar se o imóvel pode se enquadrar em hipótese objetiva de isenção". A calculadora não pergunta, não avalia e não menciona isenções em nenhum resultado. Um imóvel residencial de baixo valor que deveria ser isento recebe "Nível 1 — sem indício claro".

**A2. TCL embutida no valor do IPTU informado**

O carnê do Rio de Janeiro contém IPTU + TCL (Taxa de Coleta de Lixo) em parcelas que o contribuinte leigo frequentemente soma. O campo "Valor Anual do IPTU no Carnê" não instrui o usuário a separar IPTU de TCL. Se o usuário informa o total do carnê:

- Imóvel residencial com IPTU real de R$12.000 e TCL de R$2.000 → usuário digita R$14.000
- Sistema calcula expected = venal × 1,2%
- A diferença de R$2.000 (TCL) gera divergência artificial de ~16%
- Para valores menores, isso pode cruzar o threshold de 30% e gerar **falso positivo de Nível 3**

**A3. Flagging de IPTU abaixo do esperado como inconsistência**

Quando o IPTU informado é menor que o esperado, o sistema aplica `Math.abs(pctDiff)` (linha 1421) e compara contra os thresholds, sem distinguir a direção da divergência. Mas IPTU abaixo do esperado é **favorável ao contribuinte** e não justifica revisão — ao contrário, uma revisão poderia resultar em aumento do IPTU. Só faz sentido investigar sobrecobrança.

### B. Erros relevantes

**B1. Alíquota residencial única (1,2%) sem progressividade**

O IPTU do Rio de Janeiro aplica alíquotas progressivas por faixa de valor venal para imóveis residenciais. Usar uma taxa fixa de 1,2% para todos os imóveis residenciais gera:
- Falso negativo para imóveis de alto valor (que deveriam pagar alíquota maior)
- Distorção no cálculo esperado como base de comparação

**B2. Formulário de contato não envia dados**

A função `submitForm()` (linhas 1345–1349) apenas esconde o formulário e exibe a tela de sucesso. Nenhum dado é enviado para nenhum lugar. O usuário acredita que enviou uma solicitação, mas não há backend recebendo nada.

```javascript
function submitForm(e) {
    e.preventDefault();
    document.getElementById('contactForm').classList.add('hidden');
    document.getElementById('formSuccess').classList.remove('hidden');
    // ← FIM. Nenhuma chamada de API, fetch, ou envio de e-mail.
}
```

**B3. Nível 2 mal nomeado**

"Precisamos de mais dados para concluir" dispara com score=1, que pode ser causado por `aumento === 'sim'` ou `reforma === 'sim'` — flags genuínas de potencial inconsistência, não falta de dados. O nome sugere ao usuário que o resultado é inconclusivo por culpa dele, quando na verdade há um indício real sendo sinalizado.

**B4. Ausência de fator de vetustez**

O site menciona prominentemente "Idade e Fator de Vetustez" como parâmetro auditado. Mas a calculadora não pede o ano de construção e não aplica nem estima o fator de depreciação. Há incoerência entre o que o site promete auditar e o que a calculadora verifica.

**B5. Ausência de padrão construtivo**

O site descreve "Tipologia e Padrão Construtivo" como parâmetro auditado. A calculadora não pergunta isso. O padrão (popular, normal, médio, alto, luxo) afeta o Valor Venal Unitário e é fonte frequente de erro nas cobranças do Rio.

### C. Limitações de produto (compreensíveis no MVP)

**C1. Sem inscrição imobiliária**
Para um MVP de triagem, é compreensível não ter. Mas é o campo que tornaria possível uma futura integração com sistemas da Prefeitura.

**C2. Sem bairro/zoneamento**
A localização afeta o valor venal unitário. Ausente no MVP.

**C3. Sem histórico de IPTUs anteriores**
Comparar anos anteriores seria útil para detectar aumento súbito, mas exigiria múltiplos inputs.

**C4. Sem captura de lead na calculadora**
A calculadora em si não pede nome, e-mail ou telefone. A única captação é no formulário de contato separado. Isso pode ser uma decisão consciente de produto (baixa fricção na triagem) ou uma lacuna não intencional.

**C5. Sem distinção entre IPTU e TCL nos campos**
O MVP deveria pelo menos orientar o usuário a informar só o IPTU, não o total do carnê.

### D. Problemas de UX ou comunicação

**D1. Placeholder numérico sem máscara de formatação**
Os campos monetários aceitam `type="number"` com placeholder `45.000`. O usuário pode digitar `45000` ou `45.000` (que o browser interpreta como `45` por causa do ponto como separador decimal em locale inglês). Não há máscara de input.

**D2. WhatsApp e e-mail são placeholders**
Todos os links de WhatsApp apontam para `5521999999999` e o e-mail é `contato@verticeiptu.com.br` — claramente dados de teste/template, não reais.

**D3. Scroll automático ao resultado pode confundir**
Quando o formulário é ocultado e o resultado aparece, há um `setTimeout` de 100ms com `scrollIntoView`. Em mobile, esse comportamento pode ser abrupto.

**D4. Campo "Documentação" pouco explicado**
O campo `c_docs` pergunta se o usuário tem "carnê, escritura/matrícula ou planta". A opção "Sim, tenho todos" pode causar confusão: se ele tem o carnê, por que está usando a calculadora para descobrir se o IPTU está errado?

---

## 9. USO DE IA, API OU SERVIÇOS EXTERNOS

**A lógica atual é 100% local e determinística. Não há uso de IA, backend ou serviços externos.**

| Verificação | Resultado |
|---|---|
| Chamadas `fetch()` ou `XMLHttpRequest` | **Nenhuma** |
| Importação de SDK OpenAI ou similar | **Nenhuma** |
| Integração Supabase / Firebase | **Nenhuma** |
| Envio de formulário (`action` HTML) | **Nenhum** — `onsubmit` previne default e só muda CSS |
| Analytics (Google Analytics, Plausible, etc.) | **Nenhum** tag encontrado |
| CDNs externos utilizados | Tailwind CSS CDN, Iconify CDN, Google Fontshare — apenas para interface, sem processamento de dados |
| Pixel de rastreamento | **Nenhum** |

**Conclusão:** Todo o cálculo acontece no browser do usuário. Nenhum dado é transmitido, armazenado ou processado fora do dispositivo do visitante.

---

## 10. CAPTURA E ARMAZENAMENTO DE DADOS

### Calculadora de triagem
- **Não captura nenhum dado pessoal**
- Os valores preenchidos existem apenas em memória durante a sessão
- `resetCalc()` (linha 1549) limpa manualmente os campos
- Nenhum `localStorage`, `sessionStorage` ou cookie é utilizado

### Formulário de contato (`#contato`)
- Captura: Nome completo, telefone/WhatsApp, e-mail, tipo do imóvel, mensagem
- **Destino dos dados: nenhum**
- `submitForm()` apenas alterna visibilidade de elementos HTML
- Nenhuma requisição de rede é feita
- O usuário recebe confirmação visual de "Solicitação enviada!" — mas tecnicamente **nada foi enviado**

### Resumo de dados

| Dado | Capturado? | Armazenado? | Enviado? |
|---|---|---|---|
| Dados da triagem (IPTU, venal, área) | Temporariamente (RAM) | ❌ Não | ❌ Não |
| Nome | Temporariamente (DOM) | ❌ Não | ❌ Não |
| Telefone / WhatsApp | Temporariamente (DOM) | ❌ Não | ❌ Não |
| E-mail | Temporariamente (DOM) | ❌ Não | ❌ Não |
| Resultado da triagem | Exibido na tela | ❌ Não | ❌ Não |

> **Problema crítico de produto:** Toda captação de lead está quebrada. O usuário completa o formulário e clica em "Enviar Solicitação" — e nada acontece no backend. A Vértice não recebe esses contatos.

---

## 11. COMPARAÇÃO COM O OBJETIVO ESPERADO DA VERT

| Critério esperado | Estado atual | Atende? | Comentário |
|---|---|---|---|
| Foco no Rio de Janeiro | Campo município presente; "outro" redireciona corretamente | ✅ Atende | Bem implementado |
| Separação correta entre tipos de imóvel | 6 categorias com alíquotas diferenciadas | ⚠️ Parcial | Alíquotas simplificadas, sem progressividade |
| Presença de lógica de isenções | **Ausente** | ❌ Não atende | Zero lógica de isenção; é objetivo declarado da Vert |
| Fórmula clara | `IPTU_esperado = venal × alíquota` | ⚠️ Parcial | Fórmula simples e legível, mas incompleta (sem vetustez, padrão, TCL) |
| Resultado prudente | Linguagem em todos os níveis é cautelosa | ✅ Atende | Nenhuma promessa de resultado financeiro |
| Sem promessa exagerada | FAQ e disclaimer do resultado reforçam isso | ✅ Atende | Ponto forte do site |
| Captura de lead ou decisão consciente de não capturar | Formulário existe mas não funciona | ❌ Não atende | Pior cenário: parece funcionar mas não funciona |
| Facilidade de manutenção das regras | Alíquotas em tabela `RATES` centralizada, fácil de editar | ✅ Atende | Boa separação das constantes |
| Detecção de indícios de inconsistência | Detecta divergência venal/IPTU e divergência de área | ⚠️ Parcial | Perde casos importantes (vetustez, padrão, TCL) |
| Direcionar para Vert quando houver potencial | CTAs em todos os resultados direcionam para WhatsApp/formulário | ✅ Atende | Bem estruturado, mas o WhatsApp é placeholder |

---

## 12. VEREDITO FINAL

### A. Resumo executivo

A calculadora atual é um **protótipo funcional com identidade visual forte e tom comercial adequado**, construído como página única em HTML/CSS/JavaScript puro. Ela executa 5 verificações independentes, gera um score e direciona o usuário para um dos 4 níveis de resultado, cada um com CTA específico.

**O que ela faz bem:** Tem linguagem prudente e sem promessas abusivas. A lógica de verificação de divergência entre IPTU informado e IPTU esperado é conceitualmente válida como triagem de primeira camada. A separação por tipo de imóvel existe. O resultado para municípios fora do Rio é correto. O disclaimer jurídico está presente.

**Onde ela falha:** A calculadora não avalia isenções — que são o principal objetivo declarado para imóveis residenciais de baixo valor. A alíquota residencial única (1,2%) não reflete a progressividade real do IPTU-RJ. O campo "Valor do IPTU no carnê" provavelmente captura o total do carnê (IPTU+TCL), criando divergências artificiais. O formulário de contato não envia dados a lugar nenhum. O WhatsApp é um número placeholder. A direção da divergência (sobrecobrança vs subcobrança) não é considerada.

**Pode ser aproveitado ou deve ser reconstruído?** A estrutura de apresentação (HTML, CSS, UX) pode ser mantida. A lógica de cálculo precisa ser refatorada com: (a) separação de TCL, (b) alíquotas progressivas por faixa, (c) inclusão de lógica de isenção, (d) tratamento diferente para IPTU < esperado vs IPTU > esperado. O formulário de contato precisa de um backend real. O número de WhatsApp precisa ser o real.

---

### B. Nível de maturidade atual

> **→ Protótipo funcional, mas juridicamente frágil**

**Justificativa:** O código executa e produz resultados sem erros de runtime visíveis. A experiência do usuário é fluida e o design é profissional. Contudo:
- Há falsos positivos previsíveis (IPTU < esperado sendo flagado como inconsistência)
- Há falsos negativos estruturais (isenções não avaliadas; vetustez e padrão construtivo ausentes)
- O formulário de contato é não funcional
- Os dados de contato são placeholders
- Usar como ferramenta pública em produção com imóveis reais pode gerar interpretações erradas com impacto legal ou reputacional para a Vértice

---

### C. Lista objetiva de correções futuras

**Manter:**
- Estrutura de UI e UX (cards, cores, responsividade)
- Tom dos resultados (cauteloso, sem promessa)
- Disclaimer jurídico nos resultados
- Lógica de exclusão para municípios fora do Rio
- Tabela `RATES` como constante centralizada (boa prática)
- Separação de verificações independentes com scores acumulados

**Remover:**
- A função `submitForm()` fake (substituir por integração real)
- O número de WhatsApp `5521999999999` (substituir pelo real)
- A lógica de flagging para IPTU menor que esperado como "inconsistência" — deve distinguir direção da divergência

**Refazer:**
- Alíquotas: substituir por tabela progressiva por faixa de valor venal (residencial no RJ tem faixas)
- Campo "Valor do IPTU no carnê": instruir usuário a informar apenas IPTU, ou criar campo separado para TCL
- Score → Nível: renomear Nível 2 de "precisamos de mais dados" para algo como "indício leve identificado"
- Backend do formulário de contato: integração com e-mail, CRM, n8n, Supabase ou similar
- Lógica de isenção: adicionar verificação básica de hipóteses objetivas (valor venal abaixo de X, uso exclusivamente residencial, etc.)

**Estudar antes de codificar:**
- Quais são as faixas e alíquotas progressivas exatas do IPTU-RJ vigentes?
- Qual o valor venal de referência para início da obrigação tributária (imóveis abaixo de X são isentos de ofício)?
- Quais as hipóteses de isenção objetiva previstas na legislação municipal do Rio?
- O carnê do Rio tem TCL embutida de forma padrão ou depende do tipo de imóvel?
- Como separar IPTU de TCL no input do usuário de forma amigável?
- Qual é o número de WhatsApp real da Vértice?

---

### D. Perguntas pendentes para Gabriel e Bruno

1. **Isenções:** Quais hipóteses de isenção de IPTU no Rio vocês consideram relevantes para triagem? (aposentado proprietário único, imóvel abaixo de valor venal X, deficiente físico, entidade de assistência, outro?) Isso define toda a lógica nova.

2. **TCL:** O que os clientes normalmente informam quando vocês pedem "o valor do IPTU"? Eles costumam dar o total do carnê (com TCL) ou já sabem separar? Isso define se precisamos de dois campos ou de uma instrução muito clara.

3. **Alíquotas progressivas:** Vocês têm a tabela atualizada das faixas progressivas residenciais do IPTU-RJ? Ou preferem manter uma alíquota média e deixar claro que é estimativa?

4. **Captura de lead:** A calculadora deve ou não pedir nome e WhatsApp do usuário antes de mostrar o resultado? Ou a experiência deve ser anônima (sem fricção) e a captação acontece só no CTA pós-resultado?

5. **Formulário de contato:** Qual o destino dos leads do formulário — e-mail direto, WhatsApp, planilha, CRM? Isso define a integração técnica necessária.

6. **WhatsApp real:** Qual o número correto? Há intenção de usar WhatsApp Business API ou link simples mesmo?

7. **Isenção vs revisão:** A calculadora deve atender imóveis residenciais de baixo valor (foco em isenção) junto com galpões e lajes corporativas (foco em divergência de valor)? Ou vocês querem separar os dois fluxos?

8. **Padrão construtivo:** Vocês querem incluir na calculadora uma pergunta sobre padrão (popular/normal/médio/alto/luxo)? O usuário médio consegue responder isso sem orientação?

9. **Veracidade dos stats:** Os números exibidos (R$3,2M+ em divergências, 310 laudos, 98% de precisão, 12 anos) são reais e verificáveis, ou são aspiracionais/estimados? Isso tem implicação em compliance publicitário.

10. **Limite jurídico da triagem:** A Vértice quer que a calculadora fique somente no plano de "há indícios?" ou quer avançar para uma estimativa de quanto o IPTU deveria ser? Isso define se a ferramenta permanece como triagem de sinalização ou evolui para simulação técnica.

---

*Fim do relatório de auditoria. Nenhuma alteração foi feita no código durante este processo.*
