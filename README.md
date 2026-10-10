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

> **Resultado em uma frase:** com os dados cadastrais disponíveis, os modelos separam bons e maus pagadores **só um pouco melhor que um sorteio**
> (nota 0,53 a 0,55, onde 0,50 é o acaso). Por isso o modelo serve como **apoio à análise**, não como decisor automático.

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
| 1 | `notebooks/01_eda.ipynb` | Análise exploratória: nulos, alvo, distribuições, outliers, correlações e testes estatísticos |
| 2 | `notebooks/02_preprocessamento.ipynb` | Limpeza, nulos, outliers, normalização e feature engineering |
| 3 | `notebooks/03_modelagem.ipynb` | Treino e comparação de 5 modelos com validação cruzada |
| 4 | `notebooks/04_avaliacao.ipynb` | Métricas, importância das variáveis, custo, teste de robustez e conclusões |

**Semente fixa:** `RANDOM_STATE = 42`, declarada na primeira célula de cada notebook.
Rodar os notebooks na ordem acima, a partir de um ambiente limpo, deve reproduzir
exatamente os números da seção 5.

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
Ressalva honesta: a nota no teste tem margem de erro de 0,496 a 0,565 (95% de confiança), e as diferenças entre os quatro algoritmos **não são estatisticamente relevantes**.

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

---

## 6. Principais conclusões

### Em uma frase
Com os dados do cadastro, **não conseguimos prever com confiança quem será mau pagador**: o melhor modelo ordena os clientes só um pouco melhor que um sorteio.

### O que os resultados significam na prática

1. **O modelo não deve decidir sozinho.** Para detectar cerca de 2 em cada 3 maus pagadores, ele recusaria cerca de 2 em cada 3 bons clientes. Entre os recusados, só ~12% seriam realmente maus, praticamente a taxa geral.
2. **Pode ajudar a priorizar a análise.** Os 20% de clientes com maior risco previsto têm cerca de **15% de maus pagadores**, contra 11,8% da média. Há algum sinal, mas fraco: serve para **organizar a fila de análise**, não para aprovar ou recusar automaticamente.
3. **Olhando só pelo custo, ainda não compensa automatizar.** Se aprovar um mau pagador custar até 5 vezes mais do que recusar um bom, é mais barato aprovar todos do que usar o modelo. Só perto de 8 vezes o ganho aparece, e é mínimo.
4. **Nenhuma característica isolada define o risco.** O atraso depende mais do **comportamento de pagamento** do que do perfil de cadastro.
5. **Cuidado com resultados "bons demais".** Quem testar com perfis repetidos obtém nota 0,76, mas ela é ilusória (é como dar a prova com as respostas).

### O que mais pesa na previsão (e o que cada variável significa)

| Variável | O que representa | O que observamos |
|---|---|---|
| **Idade** (a mais importante) | Idade do solicitante | Mais jovem, um pouco mais de risco: 13,4% de maus entre 20 e 30 anos, contra 10,6% entre 60 e 70 |
| **Imóvel próprio** | Possui casa/apartamento | Quem tem: 11,1% de maus. Quem não tem: 13,1% |
| **Carro próprio** | Possui carro | Mesma direção, efeito menor (11,3% × 12,0%) |
| **Tempo de emprego** | Anos no emprego atual | Efeito pequeno e sem padrão simples |
| **Gênero** | Masculino/feminino | Pequena diferença (12,7% de maus entre homens, 11,3% entre mulheres). **Atenção ética e regulatória** |
| **Renda** | Renda anual | **Quase nenhum efeito**: ganhar mais não significou pagar melhor nesta base |

Todas essas diferenças são **pequenas** (1 a 3 pontos percentuais). Elas existem estatisticamente, mas, sozinhas, não bastam para identificar um mau pagador.

### Limitações e próximos passos

- **Falta informação:** incluir histórico em bureaus de crédito, movimentação financeira e relacionamento com o banco é o caminho mais promissor.
- **O rótulo foi construído por nós** (atraso ≥ 30 dias) e deve ser alinhado à política de risco real da instituição.
- **Só há resposta para quem já recebeu crédito** (viés de seleção): o modelo não conhece o comportamento de quem foi recusado.
- **Custos reais** de aprovar um mau pagador e de recusar um bom devem substituir as razões de exemplo usadas na análise de custo.
- **Ética e regulação:** avaliar o uso de gênero e idade, calibrar as probabilidades, validar em outro período e monitorar o modelo antes de qualquer uso em produção.

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
