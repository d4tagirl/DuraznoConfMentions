<p align="center">
<img src="header.png"/>
</p>
Les dejo acá el código de las imágenes que mostré en la
[DuraznoConf](http://duraznoconf.uy/) de los tweets mencionando a la
conferencia, por si se animan a jugar con algo de ésto con sus propias
búsquedas de palabras en Twitter.

Que se diviertan!

Cargo las librerías que uso:

    library(dplyr)
    library(rtweet)
    library(wordcloud)
    library(igraph)
    library(viridis)
    library(tm)
    library(stringr)
    library(widyr)
    library(ggraph)
    library(tidytext)
    library(rcorpora)
    library(purrr)
    library(emo)

Bajo tweets que hablen de la DuraznoConf y miro los primeros

    tweets <- rtweet::search_tweets(q = "@DuraznoConf OR to:DuraznoConf OR DuraznoConf", 
                                    include_rts = FALSE) 

    head(tweets)

Extraigo la info de los usuarios que hablaron de la DuraznoConf:

    users <- toString(tweets$screen_name) %>%
      str_split(pattern = " ", simplify = TRUE)

Hago una nube de palabras con los usuarios que hablaron de la
DuraznoConf:

    set.seed(16995145)
    wordcloud(users, colors = viridis::viridis_pal(end = 0.8)(10),
              min.freq = 800, random.color = TRUE, max.words = 100,
              scale = c(3.5,.8), rot.per=.3)

<p align="center">
<img src="mentions.png" width=50%/>
</p>
Selecciono las palabras comúnes como "el", "es", "la" para que no las
tome en cuenta:

    stopwords <- corpora("words/stopwords/en")$stopWords
    stopwords <- c(stopwords, corpora("words/stopwords/es")$stopWords, "t.co", "https", "ésto")

Veo qué palabras son las más usadas en los Tweets que mencionan a la
DuraznoConf, sacando las menciones y las urls:

    words <- data_frame(text = tweets$text) %>% 
      mutate(text = str_replace_all(text, '@([a-zA-Z0-9\\_\\.]+)', ''),
             text = str_replace_all(text, 'https:([a-zA-Z0-9\\_\\.\\/]+)', '')) %>% 
      unnest_tokens(word, text) %>% 
      filter(!word %in% stopwords)

    words_used <- toString(words$word) %>%
      str_split(pattern = " ", simplify = TRUE)

Hago una nube de palabras con las palabras usadas en los Tweets que
mencionaron a la DuraznoConf:

    set.seed(1699351)
    wordcloud(words_used, colors = viridis::viridis_pal(end = 0.8)(10),
              min.freq = 2, random.color = TRUE, max.words = 40,
              scale = c(3.5,.2), rot.per=.2)

<p align="center">
<img src="words.png" width=50%/>
</p>
Preparo los datos para armar la red de los mencionados en los Tweets:

    mentioned_users <- tweets %>% 
      mutate(mentions = map_chr(.$mentions_screen_name, paste0, collapse = " ")) %>% 
      select(status_id, mentions, screen_name, reply_to_screen_name)

Construyo la red de los usuarios que hablaron de la DuraznoConf y cómo
se vincularon entre ellos:

    set.seed(365464)
    data_frame(users = mentioned_users$mentions, status_id = mentioned_users$status_id)  %>% 
      unnest_tokens(user, users) %>% 
      pairwise_count(user, status_id, sort = TRUE, upper = FALSE) %>% 
      graph_from_data_frame() %>%
      ggraph(layout = "fr") +
      geom_edge_link(aes(edge_alpha = n), show.legend = FALSE,edge_colour = "red", edge_width = 1,
                     end_cap = circle(.07, 'inches')) +
      geom_node_point(color = "red", size = 5, alpha = .5) +
      geom_node_text(aes(label = name), repel = TRUE, 
                     point.padding = unit(0.2, "lines"), vjust = 1, hjust = 1) +
      theme_void()

Y ésto es lo que se ve 🎉

![Red de menciones Duraznoconf](net.png)
