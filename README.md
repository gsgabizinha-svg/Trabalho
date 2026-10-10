# Tech Challenge – Fase 2 · Predição de Aprovação de Cartão de Crédito

**POSTECH – FIAP** · Classificação supervisionada de **bons** e **maus** pagadores a partir de dados cadastrais.

> **Resumo executivo (leia primeiro).** Construímos um pipeline completo (limpeza → EDA → 5 modelos → avaliação → interpretação).
> Com validação rigorosa (sem vazamento), os dados cadastrais discriminam **pouco** os maus pagadores:
> o melhor modelo atinge **ROC-AUC ≈ 0,53–0,55** (baseline = 0,50). Mostramos que o ROC-AUC de **~0,76** obtido com um split aleatório é
> **ilusório**, causado por perfis duplicados entre treino e teste. A recomendação é usar o modelo apenas como **triagem complementar**
> e enriquecer a base com dados comportamentais.

## 1. Problema de negócio
Instituições financeiras precisam decidir se concedem cartão de crédito, equilibrando **crescimento** (aprovar bons clientes) e **risco de inadimplência**
(aprovar maus pagadores). Objetivo: apoiar a decisão com um modelo que, a partir de renda, ocupação, estado civil, histórico de trabalho etc.,
estime a probabilidade de o solicitante ser mau pagador.

## 2. Dados
| Arquivo | Conteúdo | Tamanho |
|---|---|---|
| `application_record.csv` | Cadastro (18 colunas) | 438.557 linhas · 438.510 IDs únicos |
| `credit_record.csv` | Histórico mensal de pagamento (`STATUS`) | 1.048.575 linhas · 45.985 IDs |

Apenas **36.457 clientes** existem nas duas tabelas – são a base de modelagem. Coloque os CSVs em `data/raw/` (não versionados, ver `.gitignore`).

**Variável-alvo (construída).** `STATUS`: 0 = 1-29 dias de atraso, 1 = 30-59, 2 = 60-89, 3 = 90-119, 4 = 120-149, 5 = 150+; C = quitado; X = sem empréstimo.
**Mau pagador = algum mês com atraso ≥ 30 dias** → **11,77%** de maus (desbalanceado). É um parâmetro (`BAD_STATUS_THRESHOLD` em `src/config.py`).
> Usamos apenas variáveis do **cadastro** como preditoras. Variáveis derivadas do `credit_record` (ex.: tempo de conta) foram evitadas porque
> não existem no momento da decisão e geram vazamento temporal.

## 3. Pipeline
```
data_prep.py → eda.py → train.py → diagnostics.py
   │              │          │            └─ vazamento + sensibilidade do alvo
   │              │          └─ split agrupado, CV, 5 modelos, threshold, importância, bootstrap
   │              └─ figuras + testes estatísticos
   └─ alvo, limpeza, feature engineering
```
**Limpeza / features:** remoção de IDs duplicados; `DAYS_*` → anos; sentinela `365243` → flag aposentado/desempregado; ocupação nula → "Nao informado";
log da renda; renda por membro; *clip* de outliers (filhos, tamanho da família); remoção de coluna constante (`FLAG_MOBIL`).

**Decisões metodológicas que protegem a validade dos resultados**
1. **Split agrupado por perfil.** 73% das linhas têm cadastro idêntico ao de outra pessoa (só 9.727 perfis únicos em 36.457 clientes).
   Um split aleatório coloca "gêmeos" em treino e teste. Usamos `StratifiedGroupKFold` para que nenhum perfil apareça nos dois lados.
2. **Pré-processamento dentro do `Pipeline`** (sem vazar estatísticas do teste).
3. **Campeão escolhido por CV no treino**, não pelo teste. O corte de decisão também é definido apenas no treino (predições *out-of-fold*).
4. **Intervalo de confiança** do ROC-AUC por bootstrap de perfis.

## 4. Resultados (conjunto de teste: 7.291 clientes, 11,77% maus)
| Modelo | CV ROC-AUC (treino) | ROC-AUC | PR-AUC | KS | Recall (mau) | Precisão (mau) |
|---|---|---|---|---|---|---|
| Baseline (Dummy) | 0,500 | 0,500 | 0,118 | 0,000 | 0,00 | 0,00 |
| Regressão Logística | 0,530 | 0,541 | 0,137 | 0,091 | 0,53 | 0,13 |
| Árvore de Decisão | 0,508 | 0,548 | 0,134 | 0,095 | 0,54 | 0,14 |
| **Random Forest (campeão por CV)** | **0,532** | 0,530 | 0,136 | 0,072 | 0,67 | 0,12 |
| Gradient Boosting (HistGB) | 0,523 | 0,553 | 0,141 | 0,101 | 0,69 | 0,13 |

- **IC 95% do ROC-AUC do campeão:** [0,496 ; 0,565] – inclui 0,50. As diferenças entre algoritmos **não são estatisticamente distinguíveis**.
- Métricas escolhidas: ROC-AUC, PR-AUC e KS (robustas ao desbalanceamento), recall/precisão da classe "mau". **Acurácia foi evitada** (um modelo que aprova todos teria 88%).
- **Decis de risco:** os decis 9-10 têm ~15% de maus vs. 11,8% médio (lift ≈ 1,27). O ordenamento é fraco e não monotônico.
- **Variáveis mais relevantes (permutação):** `AGE_YEARS`, `FLAG_OWN_REALTY`, `FLAG_OWN_CAR`, `YEARS_EMPLOYED`, `IS_MALE`. Renda tem importância ≈ 0.
- **Teste estatístico:** várias variáveis são significativas (p < 0,05), mas com V de Cramér < 0,05 (efeitos minúsculos).

### Diagnóstico de vazamento (resultado-chave)
| Cenário | ROC-AUC | PR-AUC | % do teste com perfil visto no treino |
|---|---|---|---|
| Split **aleatório** (errado) | 0,760 | 0,371 | 89,4% |
| Split **agrupado** (correto) | 0,505 | 0,122 | 0,0% |

### Sensibilidade à definição de "mau"
| Definição | % maus | ROC-AUC |
|---|---|---|
| ≥ 30 dias | 11,8% | 0,505 |
| ≥ 60 dias | 1,7% | 0,563 |
| ≥ 90 dias | 0,8% | 0,625 |
Atrasos mais graves são ligeiramente mais previsíveis, porém com pouquíssimos casos (instável).

### Análise de custo (`reports/sensibilidade_custo.csv`)
Para custo de aprovar um mau (FN) até **5×** o de recusar um bom (FP), "aprovar todos" custa menos que usar o modelo; só aparece ganho marginal (~1%) com razão ≈ 8:1.
Ou seja: **com estes dados, o modelo ainda não justifica automação da decisão.**

## 5. Conclusões e recomendações
1. Entregamos um pipeline reprodutível e validado corretamente; o baixo desempenho reflete **o limite informacional do cadastro**, não falha de modelagem.
2. Usar como **score de triagem complementar** (ex.: priorizar análise manual dos decis de maior risco), nunca como decisor único.
3. **Enriquecer a base:** histórico de bureau, comportamento transacional, relacionamento prévio com o banco.
4. **Governança:** `IS_MALE` e `AGE_YEARS` entram no modelo – avaliar impacto regulatório/ético (discriminação) antes de produção; definir custos reais FN/FP; monitorar *drift*.

## 6. Como reproduzir
```bash
pip install -r requirements.txt
# coloque application_record.csv e credit_record.csv em data/raw/
python -m src.data_prep      # gera data/processed/dataset_model.csv
python -m src.eda            # figuras + testes estatísticos em reports/
python -m src.train          # treina, avalia, salva models/modelo_campeao.joblib e reports/*
python -m src.diagnostics    # vazamento e sensibilidade do alvo
```
Notebook narrado e já executado: `notebooks/tech_challenge_fase2.ipynb`.

**Usar o modelo salvo**
```python
import joblib, pandas as pd
m = joblib.load("models/modelo_campeao.joblib")
score = m["pipeline"].predict_proba(novos_clientes[m["features"]])[:, 1]   # prob. de mau pagador
recusar = score >= m["threshold"]
```

## 7. Estrutura
```
├── data/raw/            # CSVs originais (não versionados)
├── notebooks/           # notebook narrado
├── src/                 # config, data_prep, eda, train, diagnostics
├── models/              # modelo campeão (.joblib)
├── reports/             # métricas (.csv/.json) e figuras
└── requirements.txt
```
> Observação: o enunciado indica o template `drlima/postech-techchallenge-fase2-template`; se o grupo preferir, basta mover estes arquivos para a estrutura dele.
