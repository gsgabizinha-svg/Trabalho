# Tech Challenge Fase 2 — estudo de risco de crédito
Pacote de apoio ao grupo: quatro notebooks completos, revisão do código original e resultados reproduzíveis sobre os CSVs recebidos.

## Problema e alvo
Classificar clientes com e sem atraso de 30+ dias em qualquer mês observado, usando apenas atributos cadastrais. O alvo é um proxy histórico de comportamento, não a decisão real de aprovação. 90+ é analisado como alternativa de definição; não se muda de alvo entre etapas.

## Executar
Extraia todo o ZIP, mantendo as pastas. Ambiente validado: Python 3.13.5. As versões fixadas requerem Python 3.12 ou superior.
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```
No Windows use `.venv\Scripts\activate`. Abra notebooks/ e execute 01, 02, 03, 04, cada um com Restart & Run All. Alternativamente: `python executar.py`. Os CSVs do anexo estão em data/raw. Não é necessário baixar dados durante a execução. Notebook 02 gera a base; 03 salva partições e pipelines; 04 salva métricas, gráficos e importância em docs/resultados.

## Método
IDs conflitantes removidos antes do join; uma linha por cliente. Perfis cadastrais idênticos separados por grupos. Pipeline com imputação, padronização e one-hot ajustado dentro dos folds. Baseline, regressão logística e floresta; 3 folds agrupados. Modelo escolhido por AP de validação; limiar por F1 out-of-fold. Teste reservado. A lista de atributos está em src/common.py. O notebook 02 documenta cada derivação; os nomes cadastrais originais são preservados.

## Dados e fonte
Arquivos recebidos: application_record.csv e credit_record.csv. A fonte indicada pelo enunciado é o Google Drive de ID `1z4yEyiCE_CGCWbvAAZQZSz-5-E5T5eYd`. Versão, licença, moeda e periodicidade de renda não foram confirmadas. Consulte data/README.md. O pacote inclui dados para reprodução local; .gitignore impede sua inclusão acidental no Git.

## Resultados e conclusão
Resultados já executados: logística escolhida; ROC-AUC teste 0,5289, AP 0,1409, precisão 0,1180, recall 0,8981 e F1 0,2086. O desempenho é fraco, com 5.732 falsos positivos, e não sustenta concessão ou recusa automática. Consulte docs/resultados_e_conclusoes.md e os CSVs em docs/resultados. A escolha depende da validação de treino, não da maior métrica de teste. Acurácia isolada é inadequada. Importância indica contribuição preditiva e não causalidade. O modelo necessita validação prospectiva, calibração e análise por segmentos antes de orientar concessão.

## Pendências humanas antes da entrega
O grupo deve inserir identificação, turma e RMs; conferir licença/fonte; revisar as conclusões; preparar apresentação em PDF; gravar vídeo de até 5 minutos; criar os links públicos de repositório, vídeo e apresentação; inserir os mesmos três links no PDF de submissão. Esses dados não foram fornecidos e não foram inventados. Consulte docs/roteiro_apresentacao.md e CHECKLIST.md. Nenhum material foi publicado ou enviado à FIAP.
