# Tech Challenge — Fase 2
## Predição de risco de inadimplência em solicitações de cartão de crédito

**Turma:** 2º ano — 2DTATBB — 4ª Entrada  
**Grupo:** 01  
**Data de entrega:** 10/10/2026

## Integrantes

| Integrante | RM | E-mail |
|---|---|---|
| Angelica Fonseca de Freitas | RM377836 | angelicafonsecabb@gmail.com |
| Fernanda de Oliveira Almeida | RM377812 | fernandaalmeida84@yahoo.com.br |
| Gabriela Souza e Silva | RM377814 | gsgabizinha@gmail.com |
| Juliana Oliveira Couto Espindola Vaz de Lima | RM377805 | juoliveiracouto@gmail.com |
| William Vladimir Rosales Merida da Silva | RM377758 | wvmerida@gmail.com |

## 1. Contexto e objetivo de negócio

A concessão de crédito exige avaliar a capacidade e o comportamento de pagamento dos solicitantes. O objetivo deste projeto é construir e comparar modelos de classificação capazes de identificar perfis com maior risco de inadimplência, usando dados cadastrais e registros históricos de crédito.

A solução é um apoio analítico à tomada de decisão, e não deve ser usada isoladamente para aprovar ou negar crédito. Decisões reais exigem validação adicional, análise de vieses, explicabilidade, governança e conformidade com as regras aplicáveis.

## 2. Bases utilizadas

O projeto utiliza duas bases CSV fornecidas para o desafio:

- `application_record.csv`: informações cadastrais e socioeconômicas, incluindo renda, ocupação, escolaridade, situação familiar, moradia, idade em dias, tempo de emprego e indicadores de contato.
- `credit_record.csv`: histórico mensal de crédito. `ID` identifica o cliente, `MONTHS_BALANCE` indica o mês relativo ao registro e `STATUS` representa o estado de pagamento naquele mês.

Dimensões observadas nos arquivos recebidos:
- Cadastro: 438.557 linhas e 18 colunas.
- Histórico: 1.048.575 linhas e 3 colunas.

As bases não têm correspondência para todos os clientes. A análise e a modelagem usam os clientes que aparecem nas duas bases após a agregação do histórico.

### Interpretação de `STATUS` e definição do alvo

Os códigos do histórico são interpretados da seguinte forma: `0` indica atraso de 1 a 29 dias; `1`, atraso de 30 a 59 dias; `2`, atraso de 60 a 89 dias; `3`, atraso de 90 a 119 dias; `4`, atraso de 120 a 149 dias; `5`, atraso de 150 dias ou mais; `C` indica crédito quitado naquele mês; e `X` indica ausência de empréstimo/registro ativo naquele mês.

Para a primeira versão do projeto, o alvo `MAU_PAGADOR` será definido como 1 quando o cliente tiver pelo menos um registro de status `2`, `3`, `4` ou `5`; caso contrário, será 0. O limiar foi escolhido para focar atrasos de 60 dias ou mais, mais severos do que atrasos curtos. Essa é uma regra operacional do trabalho, não uma verdade universal sobre risco de crédito. O notebook de preparação documenta a regra e a proporção de cada classe.

## 3. Metodologia

1. Análise exploratória: dimensões, tipos, valores ausentes, distribuições, outliers, balanceamento e relações entre variáveis.
2. Integração das bases por `ID` e criação do rótulo a partir do histórico mensal.
3. Pré-processamento: remoção de identificadores como preditores, tratamento de nulos, codificação de variáveis categóricas e padronização de variáveis numéricas quando aplicável.
4. Treinamento e comparação de pelo menos dois classificadores, com divisão estratificada entre treino e teste e `RANDOM_STATE = 42`.
5. Avaliação com acurácia, precisão, recall, F1-score, AUC-ROC e matriz de confusão.
6. Interpretação das variáveis mais relevantes e discussão das limitações de negócio.

O conjunto é desbalanceado. Por isso, a acurácia não será usada sozinha: recall, F1 e AUC-ROC ajudam a avaliar a identificação da classe minoritária e a capacidade de distinguir os grupos. Os resultados finais devem ser interpretados junto à matriz de confusão e aos custos de falsos positivos e falsos negativos.

## 4. Estrutura do repositório

```text
.
├── README.md
├── requirements.txt
├── .gitignore
├── CHECKLIST.md
├── data/
│   ├── README.md
│   ├── raw/
│   └── processed/
├── docs/
│   ├── README.md
│   └── apresentacao_executiva.pdf   # a produzir pelo grupo
└── notebooks/
    ├── 01_eda.ipynb
    ├── 02_preprocessamento.ipynb
    ├── 03_modelagem.ipynb
    ├── 04_avaliacao.ipynb
    └── README.md
```

## 5. Como reproduzir

1. Instale Python 3.10 ou 3.11.
2. Crie e ative um ambiente virtual.
3. Instale as dependências: `pip install -r requirements.txt`.
4. Baixe as duas bases a partir da fonte indicada no enunciado e coloque os CSVs em `data/raw/` com os nomes `application_record.csv` e `credit_record.csv`.
5. Abra os notebooks no Jupyter Notebook, JupyterLab ou Google Colab e execute na ordem `01` a `04`.
6. Execute todas as células e salve os notebooks com as saídas dos gráficos e das métricas.

Os arquivos de dados não devem ser enviados ao GitHub. O repositório deve conter o código e instruções para obter os dados, respeitando a fonte e suas condições de uso.

## 6. Resultados e conclusões preliminares

Na execução inicial com os arquivos fornecidos, a integração resultou em **36.457 clientes**. Pela regra adotada, aproximadamente **1,69%** foram classificados como `MAU_PAGADOR = 1`. Portanto, a base é bastante desbalanceada: há muito mais clientes sem registro de atraso severo do que clientes com pelo menos um atraso de 60 dias ou mais.

| Modelo | Acurácia | Precisão | Recall | F1-score | AUC-ROC |
|---|---:|---:|---:|---:|---:|
| Regressão Logística | 59,22% | 2,08% | 50,41% | 0,040 | 0,557 |
| Random Forest | 96,11% | 17,93% | 36,59% | 0,241 | 0,754 |

**Interpretação em linguagem simples:** a Regressão Logística identificou cerca de metade dos clientes da classe positiva no teste, mas gerou muitos alertas incorretos. Sua precisão foi de aproximadamente 2%, ou seja, entre os clientes que ela sinalizou como positivos, poucos pertenciam à classe definida como mau pagador. O Random Forest teve melhor F1-score e AUC-ROC e maior precisão, mas identificou aproximadamente 37% dos casos positivos e deixou passar os demais. Sua acurácia alta, sozinha, seria enganosa por causa do desbalanceamento.

Na matriz de confusão do conjunto de teste, a Regressão Logística identificou 62 dos 123 casos positivos e sinalizou incorretamente 2.913 casos negativos. O Random Forest identificou 45 casos positivos, deixou passar 78 e sinalizou incorretamente 206 casos negativos. Esses números ilustram a troca entre encontrar mais casos de risco e evitar alertas indevidos.

**Conclusão preliminar:** entre os dois modelos avaliados, o Random Forest apresentou melhor equilíbrio geral segundo F1-score e AUC-ROC. Ainda assim, os resultados não são suficientes para recomendar uso operacional. O desempenho na classe positiva continua limitado, e o rótulo é uma aproximação baseada nos registros disponíveis, não uma confirmação de comportamento futuro. É importante revisar a definição do alvo, testar outros limiares e estratégias para lidar com o desbalanceamento, validar em períodos futuros e analisar possíveis vieses antes de qualquer uso real.

A importância das variáveis deve ser interpretada a partir do gráfico produzido no notebook `04_avaliacao.ipynb`. Ela mostra quais atributos o modelo utilizou mais, mas não prova causalidade. Antes da entrega, executem os quatro notebooks, confiram os resultados e atualizem esta seção caso os valores mudem devido a alterações no código ou no ambiente.

## 7. Entregáveis finais

- Repositório GitHub público com código e notebooks executados.
- Vídeo executivo de até 5 minutos, com ao menos um integrante aparecendo ou narrando.
- Apresentação gerencial em `docs/apresentacao_executiva.pdf`.
- PDF de submissão com os links do repositório, vídeo e apresentação.

**Links para completar na entrega**
- Repositório: `PREENCHER`
- Vídeo: `PREENCHER`
- Apresentação: `PREENCHER`
