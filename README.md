# MVP Engenharia de dados
## Definição do problema: Qual o custo benefício de plataformas de streaming baseado no gosto pessoal por produtos audiovisuais?
Em 2007, a netflix se tornou pioneira e dominante no mercado de streaming global. Hoje existem diversas plataformas de streaming que oferecem o mesmo serviço de diferentes maneiras. Uma das formas de diferenciação entre essas plataformas é o catálogo de filmes e séries que elas oferecem para o público. Consumidores diferenciam-se nas preferências de gênero de filmes e séries que consomem, o que pode influenciar na escolha de assinatura. Por exemplo, se uma pessoa assiste apenas filmes e séries de comédia, qual é a melhor plataforma para ela assinar? E se duas plataformas forem muito semelhantes oferecendo conteúdo de comédia, mas uma for muito mais barata que a outra?  
## Objetivo
Considerando essa variação de gosto pessoal pelo consumo de conteúdo audiovisual e a variação de preço que existe entre plataformas, esse trabalho visa identificar quais plataformas oferecem os melhores serviços filmes e séries para determinada preferência. Serão analizados apenas os planos sem anúncios para fins de padronização das comparações.
## Coleta dos dados
Os dados foram coletados na base de dados do Kaggle (https://www.kaggle.com/datasets) nos seguintes links:

- Prime video =  https://www.kaggle.com/datasets/octopusteam/full-amazon-prime-dataset
- Netflix = https://www.kaggle.com/datasets/octopusteam/full-netflix-dataset
- HBO =  https://www.kaggle.com/datasets/octopusteam/full-hbo-max-dataset
- Hulu =  https://www.kaggle.com/datasets/octopusteam/full-hulu-dataset
- Apple TV = https://www.kaggle.com/datasets/octopusteam/full-apple-tv-dataset


Esse conjunto de dados foram armazenados na nuvem do Databricks Community. Segue um print abaixo como prova de que todas as operações foram feitas no Databricks:

![Ambiente Databricks](https://github.com/pedro1999-wolf/MVP---Engenharia-de-dados/blob/main/Print_Ambiente_Databricks.png)

## Transformação dos dados
- **União dos dados:** Os dados foram unidos em um único dataframe.
- **Explosão da coluna gênero:** A coluna de gêneros foi explodida para fazer análises precisas de cada gênero.
- **Limpeza dos dados** Os dados foram analisados minuciosamente à procura de inconsistências e devidamente corrigidos.Nesse processo também foram excluídos gêneros de filmes e séries muito específicos, que tornariam as análises extensas. Detalharei mais abaixo em análise da qualidade dos dados.
- **Tranformação final dos dados:** Os dataframes foram processados para gerar um novo conjunto de dados baseado no ranking de filmes e séries por cada gênero nas plataformas baseadas nas notas do imdb. Foram tabém realizadas análises de ANOVA para testar se há diferença significativa na média das notas das plataformas em relação a plataforma que ficou em primeiro no ranking. Por exemplo, a média de nota do imdb de filmes de comédia da HBO é significativamente diferente da Hulu? 
- **Criação da tabela dimensão das plataformas**: Foi criada em seguida a tabela de dimensão das plataformas, onde criei um id_plataforma, associei ao nome de uma plataforma e coloquei o preço da mensalidade sem anúncios (obtido por pesquisa na rede).
- **Criação da tabela fato**: Por fim, foi criada a tabela fato a partir da tabela de ranking apenas substituindo a coluna com nome de plataformas por id_plataforma, tornando-a ligada à tabela dimensão da plataformas.

  
## Modelagem dos dados
Os dados foram modelados segundo um esquema de estrela, como mostra o diagrama abaixo:

![Esquema Estrela](https://github.com/pedro1999-wolf/MVP---Engenharia-de-dados/blob/main/Esquema%20estrela.png)

## Catálogos dos dados

### 1. Tabela Fato
| Coluna                   | Tipo     | Descrição                                                                 | Domínio (Exemplos ou Faixas)               | Linhagem                                                                 |
|--------------------------|----------|---------------------------------------------------------------------------|--------------------------------------------|--------------------------------------------------------------------------|
| type                     | string   | Tipo de produção audiovisual                                              | `movie`, `tv`                              | Derivado da coluna `type` original dos CSVs por plataforma               |
| genero                   | string   | Gênero principal da obra analisada                                        | `comedy`, `action`, `drama`, `romance`, ...| Extraído da coluna `genres` após explosão e limpeza                     |
| media_imdb               | float    | Média das notas IMDb da plataforma para aquele gênero                     | de 0.0 a 10.0                               | Agregado a partir de `imdbAverageRating` por grupo                      |
| ranking                  | integer  | Posição da plataforma em comparação às demais no mesmo gênero             | 1 a N (tipicamente 1 a 5)                   | Calculado por média IMDb decrescente                                    |
| p_valor_vs_top1          | float    | p-valor do teste ANOVA comparando com a plataforma de ranking 1           | `null` ou valor entre 0.0 e 1.0             | Gerado com `scipy.stats.f_oneway()` entre Top1 e atual                  |
| diferenca_significativa | boolean  | Se a diferença é estatisticamente significativa (p < 0.05)                | `true`, `false`                            | Derivado do `p_valor_vs_top1`                                           |
| id_plataforma            | bigint   | ID da plataforma de streaming correspondente (FK para `dim_plataforma`)   | 1 a N                                      | Criado com `monotonically_increasing_id()` e referenciado por JOIN      |

### 2. Dimensão plataforma
| Coluna                   | Tipo     | Descrição                                                        | Domínio (Exemplos ou Faixas)         | Linhagem                                                      |
|--------------------------|----------|------------------------------------------------------------------|--------------------------------------|---------------------------------------------------------------|
| id_plataforma            | bigint   | Identificador único da plataforma (chave primária)              | Ex: `1`, `2`, `3`, `4`, `5`           | Gerado com `monotonically_increasing_id()` no PySpark         |
| plataforma               | string   | Nome da plataforma de streaming                                 | `Netflix`, `HBO`, `Prime Video`, ... | Adicionado manualmente durante ingestão dos CSVs              |
| preco_mensal_sem_anuncios| float    | Valor mensal da assinatura sem anúncios (em R$)                 | Ex: `18.99`, `21.90`, `59.90`         | Coletado manualmente de fontes oficiais no site de cada plano |

## Carga dos dados
Os dataframes finais (df_final e dim_plataforma) foram carregados para o Delta Lake do Databricks como tabela_fato e dim_plataforma para a realização das consultas em SQL. Destaco abaixo o código utilizado para tal:

- df_final.write.format("delta").mode("overwrite").saveAsTable("tabela_fato")
- dim_plataforma.write.format("delta").mode("overwrite").saveAsTable("dim_plataforma")

Seguem as imagens deles carregados:

![Carga na nuvem](https://github.com/pedro1999-wolf/MVP---Engenharia-de-dados/blob/main/Carga%20na%20nuvem.png))

## Análises
### Qualidade dos dados
Os dados brutos vieram com algumas inconssitências que foram corrigidos durante o processamento:

- 1. A coluna de gêneros veio com uma lista, acumulando vários gêneros em uma linha, o que dificultava análises. A explosão do gênero resolveu o problema.
- 2. As colunas title e imdbAverageRating, essenciais para as posteriores tranformações, estavam com vários valores nulos. Todas as linhas dessas colunas com valores nulos foram excluídas.
- 3. A coluna de gêneros veio com valores redundantes. Depois de excluir os gêneros que não seriam utilizados, eliminei as redundâncias dando um nome só: por exemplo, "sci fi" e "science fiction se tornaram apenas "sci fi".
- 4. A coluna de tipos veio com um valor ""    Princess", que não faz sentido algum, e portanto foram eliminadas suas linhas.
 
Fora essas inconsistências, o resto dos dados e colunas eliminados foram feitos arbitrariamente para tornar a tabela de análise mais limpa. Por exemplo, as colunas "releaseYear", "imdbId", "imdbNumVotes" e "availableCountries" foram eliminadas logo no início.

### Solução do problema
Seguem abaixo todos os insights retirados das consultas para a solução do problema.
- **Gosto pessoal para filmes**:
- 1. A HBO liderou o pódio de melhor plataforma de filmes na maioria dos gêneros, com excessão de filmes de comédia.
- 2. Para quem gosta de filmes de comédia, a plataforma Hulu é grande custo-benefício, pois seu catálogo possuí a mesma qualidade que o catálogo da HBO e é 37 reais mais barata.
- 3. O Prime Video é a pior plataforma de filmes para todos os gêneros, apesar de sua versão sem anúncios ser mais cara que outras opções com catálogos melhores (ex. Apple e Hulu).

- **Gosto pessoal para séries de tv:**
- 1. HBO seguiu sendo uma das melhores plataformas de streaming para todos os gêneros.
- 2. Em comédia, todas as plataformas são equivalentes em qualidade, exceto prime video. Vale o cliente avaliar quanto quer gastar e os exclusivos oferecidos.
- 3. Em ação, a HBO é a melhor plataforma de séries. Game of thrones!
- 4. Em drama, todas as plataformas são equivalentes, cabe ao cliente decidir.
- 5. Em romance, a HBO e a Netlix são equivalentes em qualidade.
- 6. Em horror todas as plataformas são equivalentes.
- 7. Em suspense, a HBO e a Hulu são equivalentes.
- 8. Em ficção científica, a Apple TV e a HBO são equivalentes.
 
**Todas as tabelas de consulta podem ser vistas no notebook python "MVP engenharia de dados.ipynb" do trabalho**

## Conclusão
As consultas com SQL permitiram responder adequadamente qual plataforma tem o maior custo benefício baseado no gosto pessoal dos 7 gêneros abordados. Entretanto, outros fatores podem ser considerados para refinar ainda mais as análises:

- 1. Aumentar a amostragem de séries, pois há muito menos séries do que filmes nos dados.
  2. Muitas dessas plataformas produzem conteúdo exclusivo, o que poderia ser uma informação importante a ser levada em consideração na hora de decidir o quanto pagar por uma plataforma.
  3. Ela poderia ser feita com todos os gêneros mais específicos que foram excluídos no início, tornando os resultados ainda mais ricos em informação.

## Autoavaliação
Comecei essa Sprint completamente desesperado com os conceitos apresentados. Comparado com as anteriores, ela é cheia de nomenclaturas e conceitos distintos que ainda me causam confusão. Sentia que eu não seria capaz de concluir esse trabalho de uma forma que eu sentisse satisfeito com o produto. Entretanto, com o passar do tempo e mais estudo, fui mergulhando na tarefa e consegui terminar o trabalho de forma satisfatória. Ainda sim, sinto que tenho que organizar melhor todo o conteúdo teórico apresentado nessa Sprint, pois com toda certeza meu notebook não está montado com o *flow* adequado de um pipeline de ETL. Saí do conteúdo teórico com dúvidas, e saio desse trabalho com ainda mais dúvidas, e é assim que um aluno se fortalece.
