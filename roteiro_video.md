# Roteiro do vídeo executivo (máx. 5 min)

Acompanha os 7 slides de `apresentacao_executiva.pdf`. Linguagem sem jargão; cada integrante pode narrar um bloco.

| Slide | Tempo | Fala sugerida |
|---|---|---|
| 1. Capa | 0:00–0:20 | "Somos o Grupo 01. Neste Tech Challenge construímos um modelo para prever se quem pede um cartão de crédito tende a ser um bom ou um mau pagador." |
| 2. Problema | 0:20–1:00 | "Aprovar um mau pagador custa mais do que recusar um bom cliente. Analisamos 36 mil clientes com cadastro e histórico de pagamento. Consideramos mau pagador quem teve pelo menos um mês com 30 dias ou mais de atraso: cerca de 12% dos clientes." |
| 3. Como fizemos | 1:00–2:00 | "Organizamos os dados, comparamos o perfil de bons e maus pagadores, testamos quatro modelos e os comparamos com uma regra simples de aprovar todo mundo. Um cuidado importante: os 36 mil clientes formam só 9,7 mil perfis diferentes. Por isso testamos o modelo apenas em perfis que ele nunca tinha visto." |
| 4. Resultado | 2:00–3:00 | "A nota dos modelos ficou entre 0,53 e 0,55, em que 0,50 é um sorteio. Para detectar 2 de cada 3 maus pagadores, o modelo precisaria recusar 2 de cada 3 bons clientes. Ou seja, separa só um pouco melhor que o acaso." |
| 5. Cuidado | 3:00–3:50 | "Se tivéssemos testado com perfis repetidos, a nota seria 0,76, um resultado que parece excelente. Mas o modelo estaria só reconhecendo perfis que já viu, como dar a prova com as respostas. Com o teste correto, a nota real é 0,51." |
| 6. O que pesa | 3:50–4:20 | "As características que mais pesam são idade e ter imóvel próprio: quem é mais jovem e não tem imóvel atrasa um pouco mais. A renda quase não faz diferença. Mas os efeitos são pequenos." |
| 7. Recomendações | 4:20–5:00 | "Recomendamos usar o modelo só como apoio ao analista, buscar mais informação, como histórico em bureaus de crédito, e revisar o uso de gênero e idade por questões éticas. O repositório traz todo o passo a passo para reproduzir o trabalho." |
