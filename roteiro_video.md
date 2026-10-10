# Roteiro do vídeo (máx. 5 min) – acompanha os 7 slides

| Slide | Tempo | Fala sugerida |
|---|---|---|
| 1. Capa | 0:00–0:20 | "Somos o grupo [__]. Neste Tech Challenge desenvolvemos um modelo para prever se um solicitante de cartão de crédito será bom ou mau pagador." |
| 2. Problema | 0:20–1:00 | "Aprovar um mau pagador custa mais do que recusar um bom. Trabalhamos com 36 mil clientes que têm cadastro e histórico de pagamento. Como não havia rótulo pronto, definimos mau pagador como quem teve atraso de 30 dias ou mais: 11,8% da base." |
| 3. Abordagem | 1:00–2:00 | "Seguimos quatro etapas: limpeza, exploração, cinco modelos contra um baseline e avaliação rigorosa. Um ponto crítico: 73% dos clientes têm cadastro idêntico ao de outro. Por isso separamos treino e teste por perfil, para o modelo ser testado em perfis que nunca viu." |
| 4. Resultados | 2:00–3:00 | "Todos os algoritmos ficaram com ROC-AUC entre 0,53 e 0,55, contra 0,50 do acaso. O campeão pela validação cruzada foi o Random Forest, mas as diferenças entre modelos não são estatisticamente relevantes. O limite está na informação, não no algoritmo." |
| 5. Vazamento | 3:00–3:50 | "Se tivéssemos dividido aleatoriamente, o ROC-AUC seria 0,76, resultado que parece excelente. Mas 89% do teste tinha um perfil idêntico no treino: o modelo só memorizava. Com a validação correta, o valor real é 0,51." |
| 6. Variáveis | 3:50–4:20 | "Idade, imóvel, veículo e tempo de emprego são as mais relevantes. A renda quase não discrimina. Os 20% de maior score têm cerca de 15% de maus contra 11,8% da média." |
| 7. Recomendações | 4:20–5:00 | "Recomendamos usar o modelo como triagem complementar, enriquecer a base com bureau e dados comportamentais, e revisar o uso de gênero e idade por questões éticas. O repositório traz o pipeline reprodutível e todas as métricas."|
