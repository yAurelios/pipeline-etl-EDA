# Roteiro do Pipeline ETL (Bronze → Silver → Gold)
 
 
**Integrantes do grupo:** _escreva aqui os nomes_
 
---
 
## Bloco 1 | Escolha do dataset
 
### Passos
- [X] Formem grupos de 3 ou 4 pessoas e escrevam os nomes dos integrantes na primeira célula do notebook.
- [X] Leiam a seção *Datasets sugeridos*, no final do roteiro, e separem 2 candidatos.
- [X] Confiram os critérios de escolha para cada candidato e baixem uma amostra de cada um.
- [X] Escolham 1 dataset. Quem já tem a base do Projeto Aplicado pode usá-la, desde que atenda aos critérios.
### Perguntas de verificação (respondam no notebook)
- [X] **P1.** Qual dataset o grupo escolheu e por que ele foi preferido ao segundo candidato?
- [X] **P2.** Quantas fontes (arquivos, tabelas ou endpoints) ele possui e quantas linhas tem a maior delas?
- [X] **P3.** Que problema de qualidade vocês já enxergaram apenas olhando uma amostra?
---
 
## Bloco 2 | Conhecer a fonte e definir a pergunta
 
### Passos
- [ ] Carreguem cada fonte e executem `shape`, `dtypes`, `head()` e a contagem de nulos por coluna.
- [ ] Identifiquem a chave que liga as fontes (se houver mais de uma) e a coluna de data (se houver).
- [ ] Escrevam, no topo do notebook, UMA pergunta de negócio em uma frase. Ela será respondida pela camada Gold.
- [ ] Desenhem no papel o fluxo fonte → Bronze → Silver → Gold, indicando quais fontes e colunas cada camada usa.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** Que tabela agregada responde à pergunta do grupo? Nessa tabela, uma linha representa o quê?
- [ ] **P2.** Quais fontes ou colunas NÃO são necessárias para a pergunta? Por que deixá-las de fora é uma decisão da etapa de Extract?
- [ ] **P3.** Algum nulo encontrado pode estar carregando informação em vez de indicar erro? Se sim, qual é o nulo e o que ele significa?
---
 
## Bloco 3 | Extract: construir a Bronze
 
### Passos
- [ ] Criem as pastas `data/bronze`, `data/silver` e `data/gold`.
- [ ] Escrevam a função `extract(destino_bronze)`, que lê cada fonte necessária e grava na Bronze sem corrigir nada.
- [ ] Acrescentem a cada arquivo da Bronze os metadados de controle: data e hora da extração e origem do dado.
- [ ] Fonte em arquivo: leiam só as colunas necessárias. Fonte em banco: leiam com `sqlite3` e SQL. Fonte em API: tratem a paginação e façam o código parar com mensagem clara se o status da resposta for diferente de 200.
- [ ] Executem `extract()` duas vezes seguidas e anotem o número de linhas da Bronze após cada execução.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** Quantas linhas e colunas tem cada arquivo da Bronze? Os números batem com os da fonte original?
- [ ] **P2.** O que aconteceu com o número de linhas na segunda execução? Se mudou, por que mudou e como corrigir?
- [ ] **P3.** Por que a Bronze não deve receber nenhuma correção, mesmo que o grupo já conheça os problemas do dado?
- [ ] **P4.** Se a fonte sair do ar amanhã, o que do pipeline continua funcionando?
---
 
## Bloco 4 | Transform: construir a Silver
 
### Passos
- [ ] Escrevam `transform(origem_bronze, destino_silver)`, lendo somente da Bronze e nunca da fonte original.
- [ ] Padronizem tipos (datas como `datetime`, números como numéricos) e nomes de colunas.
- [ ] Tratem duplicados, contando as linhas antes e depois.
- [ ] Tratem os nulos coluna a coluna, decidindo entre remover, preencher ou manter. Registrem cada decisão em uma célula de texto no formato: *decidimos X porque Y*.
- [ ] Tratem valores inválidos (negativos, datas impossíveis, categorias inconsistentes) e contem quantos registros cada regra afetou.
- [ ] Se houver mais de uma fonte, façam a junção e comparem a soma da principal coluna numérica antes e depois do join.
- [ ] Apliquem o filtro de escopo que a pergunta de negócio exigir, contando quantos registros ele descartou.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** Quantas linhas tem a Bronze e quantas tem a Silver? Explique a diferença usando cada uma das decisões tomadas.
- [ ] **P2.** A soma da coluna principal bateu antes e depois da junção? Se não bateu, o que foi duplicado ou perdido?
- [ ] **P3.** Qual decisão de limpeza mais alterou o resultado final? Como o grupo mediu isso?
- [ ] **P4.** Algum nulo foi mantido de propósito? Justifique.
- [ ] **P5.** Se uma regra de limpeza mudar amanhã, qual camada precisa ser reprocessada e qual não precisa? Por quê?
---
 
## Bloco 5 | Load: construir a Gold
 
### Passos
- [ ] Escrevam `load(origem_silver, destino_gold)`, que agrega a Silver para responder à pergunta do grupo.
- [ ] Calculem pelo menos 2 métricas na agregação. Para contar entidades distintas, usem `nunique` e não `count`.
- [ ] Gravem a Gold em Parquet.
- [ ] Gravem também a Gold em uma tabela SQLite com `if_exists="replace"` e executem uma consulta SQL que responda à pergunta.
- [ ] Comparem o tamanho em disco da Silver e da Gold.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** Qual é a resposta da pergunta de negócio? Mostrem o resultado (o top 5 ou o valor encontrado).
- [ ] **P2.** Quantas linhas tem a Gold e quantas tem a Silver? O que se perde na agregação e por que isso é aceitável?
- [ ] **P3.** O resultado da consulta SQL é igual ao do pandas? Provem com uma comparação em código.
- [ ] **P4.** Por que gravar a Gold em dois formatos? Quem consome cada um deles?
- [ ] **P5.** O que aconteceria na segunda execução se o grupo tivesse usado `if_exists="append"`?
---
 
## Bloco 6 | Load: modelagem dimensional
 
### Passos
- [ ] Escrevam em uma frase o grão da tabela fato do grupo, no formato: *cada linha representa um(a) ____*.
- [ ] Identifiquem na Silver quais colunas descrevem o contexto (quem, o quê, onde, quando) e quais são medidas numéricas.
- [ ] Criem a `dim_tempo`, com uma linha por data e colunas de ano, mês, trimestre e dia da semana.
- [ ] Criem pelo menos mais uma dimensão (por exemplo produto, cliente, zona ou região), com uma linha por membro e uma chave substituta própria, diferente da chave original. Se o dataset tiver poucas dimensões naturais, criem a `dim_tempo` e uma segunda dimensão definida pelo grupo, justificando a escolha.
- [ ] Criem a tabela fato apenas com as chaves substitutas das dimensões e as medidas numéricas, sem repetir texto descritivo.
- [ ] Gravem as dimensões e a fato em Parquet e em SQLite, com chave primária nas dimensões e chaves estrangeiras na fato.
- [ ] Respondam à pergunta de negócio do grupo com um JOIN entre a fato e as dimensões e comparem o resultado com o da Gold agregada.
- [ ] Acrescentem a construção do modelo dimensional à função `load()`, para que `pipeline()` o gere automaticamente.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** Qual é o grão da tabela fato? Como o grupo garantiu que ele foi respeitado, ou seja, que não há duplicidade dentro do grão?
- [ ] **P2.** A soma da principal medida na fato bate com a soma na Silver? Se não bate, o que foi perdido ou duplicado na ligação com as dimensões?
- [ ] **P3.** Por que a dimensão usa uma chave substituta em vez da chave original da fonte?
- [ ] **P4.** O JOIN reproduz o resultado da Gold agregada? Em que situação o modelo dimensional responde algo que a Gold agregada não consegue responder?
- [ ] **P5.** O que acontece com a fato se uma dimensão receber um valor novo ou tiver um valor antigo alterado? Como o grupo trataria cada caso?
---
 
## Bloco 7 | Costurar o pipeline
 
### Passos
- [ ] Escrevam a função `pipeline()`, que chama `extract`, `transform` e `load` nessa ordem.
- [ ] Antes de cada etapa, verifiquem se a entrada existe. Se não existir, parem com uma mensagem que diga qual etapa anterior está faltando.
- [ ] Façam cada etapa gravar uma linha em um CSV de histórico, com etapa, horário, duração em segundos, linhas lidas e linhas escritas. Meçam o tempo com `time.perf_counter()`.
- [ ] Testem a dependência: apaguem a Silver e chamem apenas `load()`.
- [ ] Prova final: apaguem toda a pasta `data`, executem `pipeline()` uma única vez e comparem a Gold nova com a anterior.
- [ ] Executem `pipeline()` duas vezes seguidas e confirmem que o resultado não muda.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** O que aconteceu ao chamar `load()` sem a Silver? A mensagem de erro ajudou a encontrar o problema?
- [ ] **P2.** A Gold reconstruída do zero é idêntica à anterior? Como o grupo verificou isso?
- [ ] **P3.** Qual etapa consumiu mais tempo? Quantas linhas por segundo ela processou?
- [ ] **P4.** Se o Extract falhar e o Transform rodar mesmo assim sobre a Bronze antiga, o que a Gold vai mostrar e como alguém perceberia o problema?
- [ ] **P5.** Desenhem as dependências entre `extract`, `transform` e `load` e escrevam o que deve acontecer com as etapas seguintes quando uma delas falha.
---
 
## Bloco 8 | Carga incremental
 
### Passos
- [ ] Dividam a Bronze em duas partes usando uma coluna de data (ou faixas de linhas, se não houver data): uma carga inicial com cerca de 80% dos registros e uma carga nova com o restante.
- [ ] Reconstruam o pipeline apenas com a carga inicial e anotem o número de linhas de cada camada e o resultado da Gold.
- [ ] Acrescentem a carga nova à Bronze sem apagar a anterior e executem o pipeline de novo.
- [ ] Garantam que registros que já estavam na Silver não sejam duplicados: definam uma chave única e removam a duplicidade ao combinar o dado novo com o antigo.
- [ ] Executem o pipeline mais uma vez, sem nenhuma carga nova, e confirmem que nada muda.
- [ ] Comparem a Gold final com a Gold gerada pelo pipeline sobre a base completa, de uma só vez.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** Quantas linhas cada camada tinha após a carga inicial e após a carga nova?
- [ ] **P2.** A Gold gerada em duas cargas é igual à gerada de uma só vez? Se houver diferença, de onde ela vem?
- [ ] **P3.** Qual chave o grupo usou para impedir duplicidade? O que aconteceria se ela não fosse única de verdade?
- [ ] **P4.** O que deveria acontecer na Gold se o dado novo corrigisse um registro que já existia? O pipeline do grupo trata esse caso?
---
 
## Bloco 9 | Segunda Gold e visualização
 
### Passos
- [ ] Escrevam uma segunda pergunta de negócio, diferente da primeira, que possa ser respondida pela mesma Silver.
- [ ] Criem a segunda tabela Gold, gravem em Parquet e em SQLite e acrescentem-na à função `load()`.
- [ ] Construam um gráfico para cada Gold com `matplotlib`, usando o tipo adequado, com título, rótulos de eixo e unidades.
- [ ] Salvem os gráficos como PNG na pasta `data/gold/graficos`.
- [ ] Executem `pipeline()` do zero e confirmem que as duas Golds e os dois gráficos são gerados novamente.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** A segunda pergunta exigiu algo que a Silver não tinha? Se sim, em qual camada a correção deve ser feita e por quê?
- [ ] **P2.** Por que manter duas tabelas Gold em vez de uma única tabela com tudo?
- [ ] **P3.** O gráfico escolhido permite que alguém sem conhecimento técnico responda à pergunta em poucos segundos? O que mudaria para melhorá-lo?
- [ ] **P4.** Qual das duas Golds é mais cara de recalcular e por quê?
---
 
## Bloco 10 | Parametrizar o pipeline
 
### Passos
- [ ] Reúnam todos os caminhos, nomes de arquivo e parâmetros (período, filtro de escopo, tamanho de lote, se houver) em um único dicionário de configuração, no topo do notebook.
- [ ] Troquem os valores fixos dentro das funções por valores desse dicionário.
- [ ] Troquem os `print` por `logging`, com nível INFO para o andamento e ERROR para falhas, gravando em um arquivo `pipeline.log`.
- [ ] Executem `pipeline()` com outro valor de parâmetro (outro período, outro filtro ou outra categoria) sem alterar nenhuma linha das funções.
- [ ] Confirmem no log e no histórico de execuções quais parâmetros foram usados em cada execução.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** Quantos pontos do código precisaram ser alterados para trocar o parâmetro? Se foi mais de um, o que ficou fixo dentro das funções?
- [ ] **P2.** O que o grupo consegue descobrir lendo apenas o `pipeline.log` de uma execução que falhou?
- [ ] **P3.** Que parâmetro, se alterado por engano, produziria uma Gold errada sem gerar nenhum erro?
- [ ] **P4.** Por que registrar os parâmetros junto do histórico de execuções?
---
 
## Bloco 11 | Documentação e reprodutibilidade
 
### Passos
- [ ] Escrevam um `README.md` na raiz do projeto com: pergunta de negócio, fontes usadas, descrição de cada camada, decisões de transformação, ordem de execução e como rodar o pipeline.
- [ ] Criem o `requirements.txt` com as bibliotecas e as versões usadas.
- [ ] Troquem a pasta do projeto, sem a pasta `data`, com outro grupo.
- [ ] Executem o pipeline do outro grupo seguindo apenas o README e anotem cada obstáculo encontrado.
- [ ] Devolvam ao grupo dono uma lista de tudo o que impediu ou atrapalhou a execução.
### Perguntas de verificação (respondam no notebook)
- [ ] **P1.** O pipeline do outro grupo rodou seguindo apenas o README? Que informação faltou?
- [ ] **P2.** A Gold gerada no computador de vocês é igual à do grupo dono? Se não for, o que explica a diferença?
- [ ] **P3.** Que parte do pipeline do seu grupo só funciona na sua máquina?
- [ ] **P4.** O que havia no README do outro grupo que vocês incluiriam no de vocês?
---
 
## Checklist de entrega
 
- [ ] Pergunta de negócio escrita no topo do notebook.
- [ ] Bronze com todas as fontes, sem alteração, com metadados de extração.
- [ ] Silver com as decisões documentadas no formato: *decidimos X porque Y*.
- [ ] Duas tabelas Gold, em Parquet e em SQLite, e um gráfico para cada uma.
- [ ] Modelo dimensional (fato e dimensões) gravado, com a soma da fato batendo com a da Silver.
- [ ] `pipeline()` reexecutável: apaga a pasta `data`, executa e a Gold volta idêntica.
- [ ] Carga incremental sem duplicidade de registros.
- [ ] Configuração em um único dicionário, `pipeline.log` e histórico de execuções em CSV.
- [ ] `README.md` e `requirements.txt` na raiz do projeto.
- [ ] Respostas das perguntas de verificação em células de texto do notebook.
 