WordClouds
================

The `wordcount_df` function need a `df` that contains a **tweet**
column.

It generates a new df with a *word* and a *freq* column.

``` r
wordcount_df = function(df) {
  
  library(tm)

  if (!is.data.frame(df)) {
    stop("Input must be a dataframe")
  }
  
  df_text = 
    df %>% 
    select(tweet) %>%
    mutate(
       tweet = gsub("#[[:alpha:]]*", "", tweet),
       tweet = gsub("@[[:alpha:]]*", "", tweet),
       tweet = gsub("https\\S*", "", tweet),
       tweet = gsub("http\\S*", "", tweet),
       tweet = gsub("@\\S*", "", tweet),
       tweet = gsub("amp", "", tweet),
       tweet = gsub("[\r\n]", "", tweet),
       tweet = gsub("[0-9]", "", tweet),
       tweet = gsub("[[:punct:]]", "", tweet)
    ) %>% 
    slice_head(n = 10000)
  
  docs_df =
    Corpus(VectorSource(df_text)) %>% 
    tm_map(removeNumbers) %>%
    tm_map(removePunctuation) %>%
    tm_map(stripWhitespace) %>% 
    tm_map(content_transformer(tolower)) %>% 
    tm_map(removeWords, stopwords("english"))
  
  words_df = 
    TermDocumentMatrix(docs_df) %>% 
    as.matrix() %>% 
    rowSums() %>% 
    sort(decreasing = TRUE)
  
  word_count_df = data.frame(word = names(words_df),freq = words_df)

  return(word_count_df)
  
}
```

Creating word and freq dfs to create **wordclouds** later

``` r
biden_text = 
  main_tweets_usa %>% 
  filter(hashtag == "Biden") %>% 
  wordcount_df()
```

    ## Warning in tm_map.SimpleCorpus(., removeNumbers): transformation drops documents

    ## Warning in tm_map.SimpleCorpus(., removePunctuation): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., stripWhitespace): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., content_transformer(tolower)): transformation
    ## drops documents

    ## Warning in tm_map.SimpleCorpus(., removeWords, stopwords("english")):
    ## transformation drops documents

``` r
trump_text =
  main_tweets_usa %>% 
  filter(hashtag == "Trump") %>% 
  wordcount_df()
```

    ## Warning in tm_map.SimpleCorpus(., removeNumbers): transformation drops documents

    ## Warning in tm_map.SimpleCorpus(., removePunctuation): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., stripWhitespace): transformation drops
    ## documents

    ## Warning in tm_map.SimpleCorpus(., content_transformer(tolower)): transformation
    ## drops documents

    ## Warning in tm_map.SimpleCorpus(., removeWords, stopwords("english")):
    ## transformation drops documents

## WordClouds

\[<http://www.clc-ent.com/TBDE/Docs/wordCloud.pdf>\]
\[<https://towardsdatascience.com/create-a-word-cloud-with-r-bde3e7422e8a>\]
\[<http://www.sthda.com/english/wiki/word-cloud-generator-in-r-one-killer-function-to-do-everything-you-need>\]

``` r
#Option1
wordcloud(words = biden_text$word, freq = biden_text$freq, min.freq = 2,           
          max.words = 250, random.order = FALSE, rot.per = 0.45,            
          colors = brewer.pal(8, "Dark2"), scale = c(3.5,0.25))
```

![](wordcloud_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
wordcloud(words = trump_text$word, freq = trump_text$freq, min.freq = 2,           
          max.words = 250, random.order = FALSE, rot.per = 0.45,            
          colors = brewer.pal(8, "RdBu"), scale = c(3.5,0.25))
```

![](wordcloud_files/figure-gfm/unnamed-chunk-3-2.png)<!-- -->

``` r
#Option2
wordcloud2(data = biden_text, size = 1.5, color = 'random-dark', ellipticity = 1)
```

![](wordcloud_files/figure-gfm/unnamed-chunk-3-3.png)<!-- -->

``` r
wordcloud2(data = trump_text, size = 1.5, color = 'random-dark', ellipticity = 1)
```

![](wordcloud_files/figure-gfm/unnamed-chunk-3-4.png)<!-- -->

Changing shape

``` r
#USA shape
wordcloud2(biden_text, size = 1.2, figPath = "usa.jpg", ellipticity = 8, minSize = 6)
```

![](wordcloud_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
wordcloud2(trump_text, size = 1.2, figPath = "usa.jpg", ellipticity = 8, minSize = 6)
```

![](wordcloud_files/figure-gfm/unnamed-chunk-4-2.png)<!-- -->

Useful stuff:

tidytext::unnest\_tokens() -\> unique words

<https://www.earthdatascience.org/courses/earth-analytics/get-data-using-apis/text-mining-twitter-data-intro-r/>

## WordClouds

\[<http://www.clc-ent.com/TBDE/Docs/wordCloud.pdf>\]
\[<https://towardsdatascience.com/create-a-word-cloud-with-r-bde3e7422e8a>\]
\[<http://www.sthda.com/english/wiki/word-cloud-generator-in-r-one-killer-function-to-do-everything-you-need>\]

## Exporting as pdf or png… **not working**

\[<https://www.r-graph-gallery.com/196-the-wordcloud2-library.html#export>\]

``` r
library("htmlwidgets")
#install.packages("webshot")
library(webshot)
webshot::install_phantomjs()

# Make the cloud
biden_usa = 
  wordcloud2(word_count_biden, size = 1.2, figPath = "usa.jpg", ellipticity = 8, minSize = 6)

# save it in html
saveWidget(biden_usa,"tmp.html",selfcontained = FALSE)

# and in png or pdf
webshot("tmp.html","biden_usa.png", delay =5, vwidth = 480, vheight = 480)
```