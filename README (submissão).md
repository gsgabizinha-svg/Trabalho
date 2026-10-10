# Tech Challenge — Fase 2 | POSTECH Data Analytics

> REMOVER AO FINAL ESSA PARTE **INSTRUÇÕES:** este README é um template. Substitua **todos** os blocos marcados com
> `<!-- PREENCHER -->` e apague as linhas de instrução antes de submeter.

---

## 1. Identificação

| Campo | Valor |
|---|---|
| Turma | 2º ano – 2DTATBB – 4ª Entrada |
| Grupo | Grupo 33 |
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
| Repositório | <!-- PREENCHER: URL pública do GitHub --> |
| Vídeo executivo (≤ 5 min) | <!-- PREENCHER: YouTube não listado / Drive com acesso liberado --> |
| Apresentação | <!-- PREENCHER: link do arquivo em `docs/` ou Drive --> |

> ⚠️ Repositório privado ou inacessível inviabiliza a avaliação da entrega.
> Confira o acesso em uma janela anônima antes de enviar.

---

## 3. O problema

A concessão de crédito é uma das atividades centrais de instituições financeiras, mas envolve riscos significativos, especialmente quando clientes não honram seus compromissos. Aprovar um mau pagador pode gerar custos elevados, enquanto recusar um bom cliente também representa perda de oportunidade. Tradicionalmente, essa análise depende de regras fixas e revisões manuais, processos que tendem a ser lentos e pouco padronizados. Com o uso de técnicas de machine learning, é possível treinar modelos a partir do histórico de milhares de clientes para estimar a probabilidade de inadimplência em novos solicitantes.

**Objetivo:** desenvolver um modelo de aprendizado de máquina voltado para a predição da aprovação de pedidos de cartão de crédito. A partir de atributos cadastrais como renda, ocupação, estado civil e histórico de trabalho, o modelo busca identificar padrões que permitam classificar os clientes como potenciais "bons" ou "maus" pagadores, oferecendo às instituições financeiras um suporte mais preciso e eficiente na tomada de decisão quanto à concessão de crédito.

### Variável alvo

<!-- PREENCHER: qual é a variável alvo, como foi definida e — se houve binarização —
     qual limiar foi adotado e por quê. Justifique com base na distribuição das classes. -->

### Dataset

| Campo | Valor |
|---|---|
| Fonte | <!-- PREENCHER: URL --> |
| Linhas × colunas | <!-- PREENCHER --> |
| Período / versão | <!-- PREENCHER --> |
| Licença de uso | <!-- PREENCHER --> |

Descrição das variáveis:

| Variável | Tipo | Descrição |
|---|---|---|
| | | |

---

## 4. Como reproduzir

```bash
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_REPOSITORIO>

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
jupyter notebook
```

Baixe o dataset e coloque o arquivo bruto em `data/raw/` (os dados **não** são versionados —
veja `data/README.md`).

Depois execute os notebooks nesta ordem:

| # | Notebook | O que faz |
|---|---|---|
| 1 | `notebooks/01_eda.ipynb` | Análise exploratória |
| 2 | `notebooks/02_preprocessamento.ipynb` | Limpeza, escala e feature engineering |
| 3 | `notebooks/03_modelagem.ipynb` | Treino e comparação dos modelos |
| 4 | `notebooks/04_avaliacao.ipynb` | Métricas, importância de variáveis e conclusões |

**Semente fixa:** `RANDOM_STATE = 42`, declarada na primeira célula de cada notebook.
Rodar os notebooks na ordem acima, a partir de um ambiente limpo, deve reproduzir
exatamente os números da seção 5.

---

## 5. Resultados

| Modelo | Acurácia | Precisão | Recall | F1 | AUC-ROC |
|---|---|---|---|---|---|
| <!-- PREENCHER --> | | | | | |
| | | | | | |

**Modelo escolhido:** <!-- PREENCHER --> — <!-- PREENCHER: por quê. -->

**Métricas priorizadas:** <!-- PREENCHER: justifique a escolha considerando o
     desbalanceamento de classes e o custo de cada tipo de erro no contexto do negócio. -->

---

## 6. Principais conclusões

<!-- PREENCHER: 3 a 5 conclusões em linguagem de negócio.
     Inclua quais variáveis mais influenciam o resultado e o que isso significa
     na prática para quem vai usar o modelo. -->

1.
2.
3.

### Limitações e próximos passos

<!-- PREENCHER -->

---

## 7. Estrutura do repositório

```
.
├── data/          dados brutos (raw) e tratados (processed) — não versionados
├── notebooks/     análise em ordem numerada
└── docs/          apresentação executiva
```

Detalhes e convenções em [`ESTRUTURA.md`](ESTRUTURA.md).
Antes de enviar, percorra o [`CHECKLIST.md`](CHECKLIST.md).

---

## 8. Tecnologias

<!-- PREENCHER: Python 3.11, pandas, scikit-learn, ... -->
