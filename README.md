# MVP Engenharia de dados
## Definição do problema: Qual o custo benefício 
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


Esse conjunto de dados foram armazenados na nuvem do Databricks Community.

## Transformação dos dados
- União dos dados: Os dados foram unidos em um único dataframe unindo todas as tabelas
- Em seguida, a coluna de gêneros foi explodida para fazer análises precisas de cada gênero. 

## Catálogos dos dados

### 1. Tabela Fato

| Coluna                   | Tipo     | Descrição                                                                 | Domínio (Exemplos ou Faixas)               | Linhagem                                                                 |
|--------------------------|----------|---------------------------------------------------------------------------|--------------------------------------------|--------------------------------------------------------------------------|
| type                     | string   | Tipo de produção: filme ou série                                          | `movie`, `tv`                              | Extraído da coluna `type` original nos datasets das plataformas          |
| genero                   | string   | Gênero principal da produção analisada                                   | `comedy`, `drama`, `romance`, `action`, ...| Explodido a partir da coluna `genres` de cada plataforma                 |
| plataforma               | string   | Nome da plataforma de streaming                                           | `Netflix`, `HBO`, `Hulu`, `Apple TV`, ...   | Adicionada como coluna fixa durante a ingestão de cada CSV              |
| media_imdb               | float    | Média das avaliações IMDb das obras naquele gênero e plataforma          | 0.0 a 10.0                                 | Média dos valores da coluna `imdbAverageRating`                         |
| ranking                  | integer  | Posição relativa da plataforma dentro do gênero (1 = melhor média)       | 1 a 5 (depende da quantidade de plataformas)| Calculado por ranking decrescente da média IMDb por gênero              |
| p_valor_vs_top1          | float    | Valor de p do teste ANOVA comparando com a melhor plataforma (rank 1)    | `null` ou valor entre 0.0 e 1.0            | Calculado via `scipy.stats.f_oneway()`                                  |
| diferenca_significativa | boolean  | Indica se a diferença entre a plataforma e a top 1 é estatisticamente significativa (p < 0.05) | `true`, `false`            | Derivado do p-valor calculado (se < 0.05, então `true`)                  |



