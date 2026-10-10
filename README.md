# Tech Challenge — Fase 2 | POSTECH Data Analytics

**Predição de aprovação de pedidos de cartão de crédito com Aprendizado de Máquina**

---

## 1. Identificação

| Campo | Valor |
|---|---|
| Turma | 2º ano – 2DTATBB – 4ª Entrada |
| Grupo | Grupo 01 |
| Data de entrega | 10/10/2026 |

### Integrantes

| Nome completo | RM | E-mail |
|---|---|---|
| Angelica Fonseca de Freitas | RM377836 | angelicafonsecabb@gmail.com |
| Fernanda de Oliveira Almeida | RM377812 | fernandaalmeida84@yahoo.com.br |
| Gabriela Souza e Silva | RM377814 | gsgabizinha@gmail.com |
| Juliana Oliveira Couto Espindola Vaz de Lima | RM377805 | juoliveiracouto@gmail.com |
| William Vladimir Rosales Merida da Silva | RM377758 | wvmerida@gmail.com |

---

## 2. Links da entrega

Estes três links são **obrigatórios** e devem ser idênticos aos do PDF de submissão.

| Item | Link |
|---|---|
| Repositório | _[inserir URL pública do GitHub]_ |
| Vídeo executivo (≤ 5 min) | _[inserir link do YouTube não listado / Drive com acesso liberado]_ |
| Apresentação | _[inserir link do arquivo `docs/apresentacao_executiva.pdf` no GitHub ou no Drive]_ |

---

## 3. O problema

Conceder crédito é uma das atividades centrais de um banco, mas traz um risco: o cliente que **não paga**. Aprovar um mau pagador costuma custar muito mais do que recusar um bom cliente.
Hoje essa análise depende de regras fixas e de revisão manual, que são lentas e pouco padronizadas. Com Machine Learning, o computador aprende com o histórico de milhares de clientes
e estima a **chance de um novo solicitante atrasar o pagamento**, ajudando a padronizar e acelerar a triagem.

**Objetivo:** classificar solicitantes de cartão de crédito como **bons** ou **maus pagadores** a partir dos dados do cadastro (renda, ocupação, estado civil, histórico de trabalho etc.).
Em termos práticos, a pergunta que respondemos é: **as informações do cadastro permitem identificar clientes com maior chance de atrasar o pagamento?**

> **O que a base permite (e não permite) responder.** Os dados reúnem o cadastro e o histórico de pagamento de clientes que **já receberam** cartão. Eles **não informam se cada pedido foi aprovado ou recusado**.
> Por isso, o modelo estima o **risco de atraso entre quem tem histórico de crédito**; ele não reproduz a decisão original de aprovação. O enunciado fala em "prever a aprovação": aqui interpretamos como *apoiar a decisão de crédito estimando o risco de atraso*.

> **Resultado em uma frase:** com os dados cadastrais disponíveis, os modelos **não separam bons e maus pagadores com confiança** (nota 0,53 a 0,55, onde 0,50 é o acaso, e o intervalo de confiança inclui o acaso).
> Há diferenças pequenas entre grupos (idade, imóvel próprio), mas combinar características também não ajudou. O modelo serve, no máximo, como **apoio à análise**, não como decisor automático.

### Variável alvo

A base não traz uma coluna "bom/mau". Nós a **construímos** a partir do histórico mensal de pagamento (`credit_record.STATUS`):
`0` = atraso de 1–29 dias, `1` = 30–59, `2` = 60–89, `3` = 90–119, `4` = 120–149, `5` = 150+ dias; `C` = quitado no mês; `X` = sem empréstimo no mês.

**Mau pagador (`TARGET = 1`) = cliente com pelo menos um mês de atraso de 30 dias ou mais.** Os demais são bons pagadores (`TARGET = 0`).

**Por que 30 dias? (justificativa pela distribuição das classes)** entre os 36.457 clientes que têm cadastro e histórico:

| Definição de "mau" | Maus | % |
|---|---|---|
| Atraso ≥ 30 dias (**adotado**) | 4.291 | 11,77% |
| Atraso ≥ 60 dias | 616 | 1,69% |
| Atraso ≥ 90 dias | 302 | 0,83% |

Com limiares mais severos sobram poucos casos de mau pagador, o que torna o aprendizado instável. Com 30 dias há exemplos suficientes.
A base é **desbalanceada**: ≈ 88% bons e ≈ 12% maus (para cada mau pagador há ~7,5 bons).

> **Limitação do alvo (viés de exposição).** Basta **um** mês de atraso em qualquer época do histórico para o cliente ser "mau". Como os históricos vão de 1 a 61 meses, quem foi observado por mais tempo teve **mais chances de atrasar**:
> a taxa de maus é de 4,0% para quem tem até 6 meses de histórico e 16,4% para quem tem mais de 3 anos. Não é erro do critério, mas precisa ser reconhecido. Testamos o efeito refazendo o alvo com a **mesma janela para todos** (ver seção 5): as conclusões se mantêm.

> Usamos como preditoras apenas informações do **cadastro**. Informações tiradas do histórico de pagamento (como o tempo de conta) foram deixadas de fora:
> elas só existem depois da concessão do crédito e fariam o modelo "colar" a resposta.

### Dataset

| Campo | Valor |
|---|---|
| Fonte | <https://drive.google.com/file/d/1z4yEyiCE_CGCWbvAAZQZSz-5-E5T5eYd/view?usp=sharing> (indicada no enunciado do Tech Challenge) |
| Linhas × colunas | `application_record.csv`: 438.557 × 18 (438.510 IDs únicos) · `credit_record.csv`: 1.048.575 × 3 (45.985 IDs). Base de modelagem (clientes presentes nas duas tabelas, após o tratamento): **36.457 × 22** |
| Período / versão | Versão única do enunciado. `MONTHS_BALANCE` vai de 0 (mês de referência) a −60 (cinco anos antes); não há datas absolutas |
| Licença de uso | Não especificada na fonte fornecida; uso estritamente acadêmico (Tech Challenge POSTECH/FIAP) |

Descrição das variáveis:

**`application_record.csv` (cadastro)**

| Variável | Tipo | Descrição |
|---|---|---|
| `ID` | Identificador | Código do cliente (liga as duas tabelas) |
| `CODE_GENDER` | Categórica | Gênero (M/F) |
| `FLAG_OWN_CAR` | Binária | Possui carro (Y/N) |
| `FLAG_OWN_REALTY` | Binária | Possui imóvel (Y/N) |
| `CNT_CHILDREN` | Numérica | Número de filhos |
| `AMT_INCOME_TOTAL` | Numérica | Renda anual |
| `NAME_INCOME_TYPE` | Categórica | Tipo de renda (Working, Commercial associate, Pensioner, State servant, Student) |
| `NAME_EDUCATION_TYPE` | Categórica | Escolaridade |
| `NAME_FAMILY_STATUS` | Categórica | Estado civil |
| `NAME_HOUSING_TYPE` | Categórica | Tipo de moradia |
| `DAYS_BIRTH` | Numérica | Dias desde o nascimento, contados para trás (valor negativo) |
| `DAYS_EMPLOYED` | Numérica | Dias no emprego atual (negativo); **365243** = não trabalha (aposentado/desempregado) |
| `FLAG_MOBIL` | Binária | Possui celular (sempre 1; descartada) |
| `FLAG_WORK_PHONE` | Binária | Possui telefone comercial |
| `FLAG_PHONE` | Binária | Possui telefone residencial |
| `FLAG_EMAIL` | Binária | Possui e-mail |
| `OCCUPATION_TYPE` | Categórica | Ocupação (30,6% nulos) |
| `CNT_FAM_MEMBERS` | Numérica | Pessoas na família |

**`credit_record.csv` (histórico)**

| Variável | Tipo | Descrição |
|---|---|---|
| `ID` | Identificador | Código do cliente |
| `MONTHS_BALANCE` | Numérica | Mês do registro em relação ao de referência (0 = atual, −1 = mês anterior…) |
| `STATUS` | Categórica | Situação de pagamento do mês (ver variável alvo) |

**Variáveis criadas (feature engineering)**

| Variável | O que é |
|---|---|
| `TARGET` | 1 = mau pagador, 0 = bom pagador |
| `AGE_YEARS`, `YEARS_EMPLOYED` | Idade e tempo de emprego em anos |
| `IS_UNEMPLOYED_OR_RETIRED` | Marca quem não trabalha (código 365243) |
| `LOG_INCOME`, `LOG_INCOME_PER_MEMBER` | Renda e renda por pessoa da família, em escala logarítmica |
| `IS_MALE` | Gênero em 0/1 |
| `OCCUPATION_TYPE = "Nao informado"` | Categoria própria para ocupação em branco |
| `PROFILE_GROUP` | Identifica perfis idênticos (usado só para separar treino e teste; **não é preditora**) |

---

## 4. Como reproduzir

```bash
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_REPOSITORIO>

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
pip install notebook             # para abrir os notebooks
jupyter notebook
```

Baixe o dataset e coloque os arquivos brutos `application_record.csv` e `credit_record.csv` em `data/raw/` (os dados **não** são versionados —
veja `data/README.md`).

Depois execute os notebooks nesta ordem:

| # | Notebook | O que faz |
|---|---|---|
| 1 | `notebooks/01_eda.ipynb` | Análise exploratória: nulos, alvo, duração do histórico, outliers, correlações, testes estatísticos e **todos os cruzamentos de variáveis** |
| 2 | `notebooks/02_preprocessamento.ipynb` | Limpeza, nulos, outliers, normalização e feature engineering |
| 3 | `notebooks/03_modelagem.ipynb` | Treino e comparação de 5 modelos com validação cruzada |
| 4 | `notebooks/04_avaliacao.ipynb` | Métricas, importância das variáveis, custo (simulação), robustez (vazamento, cruzamentos em clientes novos, janela uniforme) e conclusões |

**Semente fixa:** `RANDOM_STATE = 42`, declarada na primeira célula de cada notebook.
Rodar os notebooks na ordem acima, a partir de um ambiente limpo, deve reproduzir
exatamente os números da seção 5. (Os notebooks 03 e 04 levam alguns minutos: treinam vários modelos e testam as 120 combinações de variáveis.)

---

## 5. Resultados

Os números abaixo vêm do **conjunto de teste**: 7.291 clientes (11,77% maus) com perfis que o modelo **nunca viu** no treino.
Precisão, recall e F1 referem-se à classe "mau pagador", no ponto de corte que melhor separa os dois grupos (definido só com dados de treino).

| Modelo | Acurácia | Precisão | Recall | F1 | AUC-ROC |
|---|---|---|---|---|---|
| Referência (aprova todos) | 0,882 | 0,000 | 0,000 | 0,000 | 0,500 |
| Regressão Logística | 0,518 | 0,128 | 0,534 | 0,207 | 0,541 |
| Árvore de Decisão | 0,551 | 0,139 | 0,541 | 0,221 | 0,548 |
| **Random Forest** | 0,401 | 0,123 | 0,670 | 0,208 | 0,530 |
| Gradient Boosting | 0,406 | 0,127 | 0,685 | 0,214 | 0,553 |

**Modelo escolhido:** **Random Forest**, porque teve a maior nota (AUC-ROC) na validação cruzada do treino (0,532). Escolhemos assim, sem olhar o teste, para que o teste continue imparcial.
Ressalva honesta: a nota no teste tem margem de erro de 0,496 a 0,565 (95% de confiança). Como o intervalo inclui 0,50, **a evidência de que o modelo supera o acaso é insuficiente** (isso não prova que seja igual ao acaso). As diferenças entre os quatro algoritmos **não são estatisticamente relevantes**.

**Métricas priorizadas:** AUC-ROC, PR-AUC e o recall dos maus pagadores. Como só ~12% dos clientes são maus, a acurácia engana: um "modelo" que aprova todos acerta 88% sem detectar um único mau pagador.
No negócio, aprovar um mau pagador (falso negativo) tende a custar mais do que recusar um bom cliente (falso positivo); por isso o recall dos maus importa, mas olhamos também o quanto de bons clientes seria recusado.

### Como ler essas métricas (sem jargão)

| Métrica | Pergunta que responde | Resultado do modelo |
|---|---|---|
| **Acurácia** | De todos os clientes, quantos o modelo classificou certo? | Baixa (40–55%), pois o modelo aceita errar em bons para tentar pegar maus |
| **Recall** | Dos maus pagadores reais, quantos o modelo pegou? | ~67 em cada 100 (Random Forest) |
| **Precisão** | Dos clientes que o modelo recusou, quantos eram mesmo maus? | ~12 em cada 100, quase igual à taxa geral de maus (11,8%) |
| **F1** | Equilíbrio entre precisão e recall | ~0,21 (baixo) |
| **AUC-ROC** | O quanto o modelo ordena os clientes por risco (0,50 = sorteio, 1,00 = perfeito) | 0,53 a 0,55, perto do sorteio |

### Teste de robustez: o perigo dos perfis repetidos

Os 36.457 clientes formam só 9.727 perfis únicos (73% das linhas repetem um perfil já existente). Por isso, treino e teste foram separados **por perfil**.
Se a divisão fosse aleatória, o modelo veria no treino "gêmeos" dos clientes do teste e pareceria muito melhor:

| Tipo de divisão | AUC-ROC | % do teste com perfil já visto no treino |
|---|---|---|
| Aleatória (incorreta) | 0,760 | 89,4% |
| Agrupada por perfil (adotada) | 0,505 | 0,0% |

### Sensibilidade à definição de "mau pagador"

| Definição | % maus | AUC-ROC |
|---|---|---|
| ≥ 30 dias (adotada) | 11,8% | 0,505 |
| ≥ 60 dias | 1,7% | 0,563 |
| ≥ 90 dias | 0,8% | 0,625 |

Atrasos mais graves ficam um pouco mais "previsíveis", mas com pouquíssimos casos (resultado instável).

### Teste de robustez: janela de observação uniforme (viés de exposição)

Para neutralizar o efeito do tempo de histórico, refizemos o alvo como "atraso de 30 dias ou mais **nos primeiros N meses**", usando só clientes com pelo menos N meses de histórico. AUC médio de 5 divisões por perfil:

| Definição do alvo | Clientes | % maus | Reg. Logística | Random Forest | Gradient Boosting |
|---|---|---|---|---|---|
| Histórico inteiro (adotado) | 36.457 | 11,8% | 0,525 | 0,527 | 0,532 |
| Primeiros 6 meses | 31.791 | 6,3% | 0,522 | 0,534 | 0,536 |
| Primeiros 12 meses | 24.566 | 11,2% | 0,541 | 0,537 | 0,530 |
| Primeiros 24 meses | 13.828 | 13,6% | 0,520 | 0,517 | 0,512 |

O desempenho **não muda** (0,51 a 0,54), e as diferenças entre grupos permanecem (ex.: até 30 anos 13,6% × 60+ 10,1%; sem imóvel 12,3% × com imóvel 10,6%).
Um dado revelador: só o **tempo de histórico** (que não existe na hora da decisão) já teria AUC 0,60, mais do que qualquer modelo com dados cadastrais.

### Quão confiáveis são as diferenças entre grupos?

Os 36.457 clientes formam 9.728 perfis únicos (9.727 depois do tratamento de outliers), e clientes do mesmo perfil costumam ter o mesmo resultado (correlação interna de 0,37). Na prática, os 36 mil clientes valem cerca de **18 mil independentes**.
Corrigindo os testes estatísticos por isso (e pelo fato de testarmos muitas coisas ao mesmo tempo):

- Das 16 variáveis analisadas, **13 pareciam relevantes** sem correção; **só 5 continuam**: imóvel próprio, gênero, e-mail informado, idade e tempo de emprego. Escolaridade, estado civil, moradia, ocupação, renda e carro **só pareciam** diferentes.
- Mesmo as 5 que restam têm efeito minúsculo (V de Cramér < 0,05).

### Cruzamentos de variáveis (todas as 120 combinações de duas variáveis)

Cruzamos **todas as combinações de 2 entre as 16 variáveis** (idade, tempo de emprego, renda, filhos, família, gênero, carro, imóvel, telefones, e-mail, tipo de renda, escolaridade, estado civil, moradia e ocupação), mostrando sempre o tamanho de cada grupo.

| Pergunta | Resultado |
|---|---|
| Quantos grupos de clientes os cruzamentos formam? | 1.871 (1.497 com ao menos 150 clientes) |
| Quantos **parecem** de alto ou baixo risco (taxa ≥ 15% ou ≤ 8%)? | 138 |
| Quantos seriam "significativos" tratando os clientes como independentes? | 248 |
| Quantos **continuam** após corrigir por perfis repetidos e múltiplos testes? | **6** |
| Quantos cruzamentos têm interação confirmada (o efeito de uma variável muda conforme a outra)? | 55 sem a correção, **0** com a correção |
| As combinações escolhidas no treino melhoram a previsão em clientes novos? | **Não** (ganho médio −0,009 de AUC no teste; correlação treino × teste = −0,16) |
| Adicionar 10 combinações a um modelo com todos os efeitos isolados | Piora (0,547 → 0,523 de AUC no teste) |

Os 6 grupos que se destacam repetem, em sua maioria, o que as variáveis isoladas já indicavam (ex.: até 30 anos sem carro: 15,7%, 2.773 clientes; sem imóvel com família de 4+ pessoas: 17,5%, 1.185 clientes; sem imóvel com 2+ filhos: 16,9%, 1.246 clientes).
Os dois restantes são grupos pequenos (381 e 484 clientes) com margem de erro grande. **São pistas para investigação, não regras de aprovação ou recusa.**
Cruzamentos detalhados no notebook 01: Idade × Imóvel, Renda × Ocupação, Escolaridade × Tempo de emprego e Estado civil × Moradia.

---

## 6. Principais conclusões

### Em uma frase
Com os dados do cadastro, **não conseguimos identificar com confiança quais clientes vão atrasar**: encontramos diferenças pequenas entre perfis, mas nenhum modelo, e nenhuma combinação de características, separou bons e maus pagadores de forma convincente.

### O que este estudo mostra (e o que não mostra)

- A base reúne clientes que **já receberam crédito** e **não diz se cada pedido foi aprovado ou recusado**. Estimamos o **risco de atraso** entre quem tem histórico, e não a decisão original do banco.
- "Mau pagador" = pelo menos um mês com 30 dias ou mais de atraso (11,8% dos clientes). Como os históricos têm durações diferentes, testamos uma janela igual para todos e **as conclusões se mantiveram**.

### O que os resultados significam na prática

1. **Diferenças pequenas entre perfis.** Clientes mais jovens atrasam mais (13,4% até 30 anos, 10,6% acima de 60). Quem tem imóvel atrasa menos (11,1% × 13,1%). Gênero mostra pequena diferença (12,7% homens, 11,3% mulheres), que **não deve virar critério de crédito** sem análise de justiça e adequação.
2. **A renda não organiza o risco.** Ganhar mais não significou atrasar menos (em algumas ocupações, como tecnologia, a taxa até sobe com a renda).
3. **Várias diferenças "sumiram" ao sermos mais rigorosos.** Escolaridade, estado civil, moradia, ocupação e carro pareciam relevantes, mas, considerando que clientes de mesmo perfil não são independentes e que testamos muitas coisas ao mesmo tempo, **não há evidência suficiente** de que sejam diferenças reais.
4. **Combinar características não ajuda.** Das 120 combinações de duas variáveis, só 6 grupos se destacaram após a correção, nenhuma interação foi confirmada e, em clientes novos, as combinações **não melhoraram a previsão**.
5. **Os modelos ficam perto do acaso.** Nota de 0,53 a 0,55 (0,50 é um sorteio). O Random Forest foi escolhido pela validação, mas não é comprovadamente melhor, e o intervalo de confiança inclui o acaso.
6. **Acertar mais maus pagadores custa recusar muitos bons.** Para detectar ~2 em cada 3 maus, o modelo recusaria ~2 em cada 3 bons; entre os recusados, só ~12% seriam realmente maus (quase a taxa geral).
7. **Cuidado com resultados "bons demais".** Com divisão aleatória o modelo parece chegar a 0,76, mas é ilusório (perfis repetidos entre treino e teste).

### Como usar na prática

- **Não** como único responsável pela aprovação ou recusa.
- Como **apoio à triagem**: os 20% de clientes com maior risco previsto têm cerca de 15% de maus, contra 11,8% da média. Isso ajuda a priorizar a análise humana.
- A análise de custo é uma **simulação com custos hipotéticos**: para as razões simuladas (até 5 vezes), aprovar todos custa menos do que usar o modelo. Com os custos reais da instituição a conta deve ser refeita.

### O que mais pesa na previsão (e o que cada variável significa)

| Variável | O que representa | O que observamos |
|---|---|---|
| **Idade** (a mais importante) | Idade do solicitante | Mais jovem, um pouco mais de atraso: 13,4% de maus até 30 anos, contra 10,6% acima de 60 |
| **Imóvel próprio** | Possui casa/apartamento | Quem tem: 11,1% de maus. Quem não tem: 13,1% |
| **Carro próprio** | Possui carro | Mesma direção, efeito menor e pouco confiável (11,3% × 12,0%) |
| **Tempo de emprego** | Anos no emprego atual | Efeito pequeno e sem padrão simples |
| **Gênero** | Masculino/feminino | Pequena diferença (12,7% × 11,3%). **Atenção ética e regulatória** |
| **Renda** | Renda anual | **Sem relação clara**: ganhar mais não significou pagar melhor nesta base |

Todas essas diferenças são **associações** de 1 a 3 pontos percentuais, e não causas: ter imóvel não "faz" ninguém pagar em dia; pode refletir idade, estabilidade ou outras características.

### Limitações e próximos passos

- **Falta informação:** histórico em bureaus de crédito, movimentação financeira e relacionamento com o banco são os dados mais promissores.
- **Alvo e janela:** definir com a área de risco o critério de "mau pagador" e um período de observação igual para todos.
- **Viés de seleção:** só observamos quem foi aprovado; não sabemos como se comportariam os recusados.
- **Custos reais** de aprovar um mau pagador e de recusar um bom devem substituir as razões de exemplo.
- **Ética e regulação:** avaliar o uso de gênero e idade, validar em outro período e monitorar o modelo antes de qualquer uso em produção.

---

## 7. Estrutura do repositório

```
.
├── .gitignore
├── CHECKLIST.md
├── ESTRUTURA.md
├── README.md
├── requirements.txt
├── data
│   ├── README.md          instruções para baixar o dataset
│   ├── raw/               dados brutos (não versionados)
│   └── processed/         gerados pelos notebooks (não versionados)
├── docs
│   ├── README.md
│   ├── apresentacao_executiva.pdf
│   └── roteiro_video.md
└── notebooks
    ├── 01_eda.ipynb
    ├── 02_preprocessamento.ipynb
    ├── 03_modelagem.ipynb
    ├── 04_avaliacao.ipynb
    └── README.md
```

Detalhes e convenções em [`ESTRUTURA.md`](ESTRUTURA.md).
Antes de enviar, percorra o [`CHECKLIST.md`](CHECKLIST.md).

---

## 8. Tecnologias

Python 3.11+ (executado em 3.12), pandas 3.0, NumPy 2.4, scikit-learn 1.8 (Pipeline, ColumnTransformer, GridSearchCV, StratifiedGroupKFold),
SciPy, Matplotlib, Seaborn, joblib e Jupyter Notebook.
