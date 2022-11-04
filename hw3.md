pm566_hw3
================
Yiping Li
2022-11-04

``` r
knitr::opts_chunk$set(echo = TRUE)
```

``` r
library(rvest)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.7     ✔ dplyr   1.0.9
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.0
    ## ✔ readr   2.1.2     ✔ forcats 0.5.1
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter()         masks stats::filter()
    ## ✖ readr::guess_encoding() masks rvest::guess_encoding()
    ## ✖ dplyr::lag()            masks stats::lag()

``` r
library(httr)
library(stringr)
```

API Question1: using the NCBI API, look for papers that show up under
the term “sars-cov-2 trial vaccine.” Look for the data in the pubmed
database, and then retrieve the details of the paper as shown in lab 7.
How many papers were you able to find?

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2+trial+vaccine")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/div[1]")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "4,009"

``` r
#gives 4010

#Query & Get details about the articles
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(db='pubmed',
               term = 'sars-cov-2 trial vaccine',
               retmax= 5000)
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)

# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, 
"<Id>[[:digit:]]+</Id")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "</?Id>")

head(ids) #above step unable to remove </Id
```

    ## [1] "36328399</Id" "36327352</Id" "36322837</Id" "36320825</Id" "36314847</Id"
    ## [6] "36307830</Id"

``` r
length(ids)
```

    ## [1] 1803

A total of 4010 papers were found on Pubmed, and a total of 1083 papers
were found on Pubmed API.

Question2: using the list of pubmed ids you retrieved, download each
papers’ details using the query parameter rettype = abstract. If you get
more than 250 ids, just keep the first 250.

``` r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db='pubmed',
    id=paste(ids,collapse =','),
    retmax=250,
    rettype='abstract'
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

Questions3: as we did in lab 7 (see Dr. Siegmund’s lab 7). Create a
dataset containing the following: Pubmed ID number, Title of the paper,
Name of the journal where it was published, Publication date, and
Abstract of the paper (if any).

``` r
#We want to build a dataset which includes the title and the abstract of the paper. The title of all records is enclosed by the HTML tag ArticleTitle, and the abstract by Abstract.
#Before applying the functions to extract text directly, it will help to process the XML a bit. We will use the xml2::xml_children() function to keep one element per id. This way, if a paper is missing the abstract, or something else, we will be able to properly match PUBMED IDS with their corresponding records.
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)

#Now, extract the abstract and article title for each one of the elements of pub_char_list. You can either use sapply() as we just did, or simply take advantage of vectorization of stringr::str_extract
abstracts <- str_extract(pub_char_list, "<Abstract>[[:print:][:space:]]+</Abstract>")

abstracts <- str_remove_all(abstracts, "</?[[:alnum:]- =\"]+>") 

abstracts <- str_replace_all(abstracts, "[[:space:]]+"," ")

#Now get the titles:
titles <- str_extract(pub_char_list, "<ArticleTitle>[[:print:][:space:]]+</ArticleTitle>")

titles <- str_remove_all(titles, "</?[[:alnum:]- =\"]+>")

#Now get the journal names:
journals <- str_extract(pub_char_list, "<Title>[[:print:][:space:]]+</Title>")

journals <- str_remove_all(journals, "</?[[:alnum:]- =\"]+>")

#Now get the dates:
dates <- str_extract(pub_char_list, "<PubDate>[[:print:][:space:]]+</PubDate>")

dates <- str_remove_all(dates, "</?[[:alnum:]]+>")

dates <- str_replace_all(dates, "[[:space:]]+"," ")

#Finally the dataset:
length(ids) <- length(abstracts) 
#I added this because if I dont, the next step wont run, giving error: Error in data.frame(PubMedId = ids, Title = titles, Journal = journals,  : arguments imply differing number of rows: 1803, 2
#But still, the output looks wrong

database <- data.frame(
  PubMedId = ids,
  Title    = titles,
  Journal  = journals,
  Date     = dates,
  Abstract = abstracts
)
knitr::kable(database[1:8,], caption = "Some papers about sars-cov-2 trial vaccine")
```

|     | PubMedId                                                                                                                                                                                                                                                                               | Title | Journal | Date | Abstract |
|:----|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------|:--------|:-----|:---------|
| 1   | 36328399</Id |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                         
       |2    |36327352</Id |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                   
       |NA   |NA           |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                   
       |NA.1 |NA           |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                   
       |NA.2 |NA           |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                   
       |NA.3 |NA           |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                   
       |NA.4 |NA           |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                   
       |NA.5 |NA           |NA    |NA      |NA   |NA       |                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                               
       Test Mining:                                                                                                                                                                                                                                                                            
       A new dataset has been added to the data science data repository https://github.com/USCbiostats/data-science-data/tree/master/03_pubmed. The dataset contains 3241 abstracts from articles across 5 search terms. Your job is to analyse these abstracts to find interesting insights.  
                                                                                                                                                                                                                                                                                               
       Question1: tokenize the abstracts and count the number of each token. Do you see anything interesting? Does removing stop words change what tokens appear as the most frequent? What are the 5 most common tokens for each search term after removing stopwords?                        
                                                                                                                                                                                                                                                                                               
       ```r                                                                                                                                                                                                                                                                                    
       library(readr)                                                                                                                                                                                                                                                                          
       urlfile1="https://raw.githubusercontent.com/USCbiostats/data-science-data/master/03_pubmed/pubmed.csv"                                                                                                                                                                                  
       pub <- read_csv(url(urlfile1))                                                                                                                                                                                                                                                          
       ```                                                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                               
       ```                                                                                                                                                                                                                                                                                     
       ## Rows: 3241 Columns: 2                                                                                                                                                                                                                                                                
       ## ── Column specification ────────────────────────────────────────────────────────                                                                                                                                                                                                     
       ## Delimiter: ","                                                                                                                                                                                                                                                                       
       ## chr (2): abstract, term                                                                                                                                                                                                                                                              
       ##                                                                                                                                                                                                                                                                                      
       ## ℹ Use `spec()` to retrieve the full column specification for this data.                                                                                                                                                                                                              
       ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.                                                                                                                                                                                                    
       ```                                                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                                               
       ```r                                                                                                                                                                                                                                                                                    
       library(tidytext)                                                                                                                                                                                                                                                                       
       pub %>%                                                                                                                                                                                                                                                                                 |       |         |      |          |

Some papers about sars-cov-2 trial vaccine

unnest_tokens(token, abstract) %\>% count(token, sort = TRUE) %\>%
top_n(20, n) %\>% knitr::kable()




    |token    |     n|
    |:--------|-----:|
    |the      | 28126|
    |of       | 24760|
    |and      | 19993|
    |in       | 14653|
    |to       | 10920|
    |a        |  8245|
    |with     |  8038|
    |covid    |  7275|
    |19       |  7080|
    |is       |  5649|
    |for      |  5492|
    |patients |  4674|
    |cancer   |  3999|
    |prostate |  3832|
    |was      |  3315|
    |that     |  3226|
    |were     |  3226|
    |as       |  3159|
    |this     |  3158|
    |are      |  2833|

    The 5 most common tokens are stop words: "the", "of", "and", "in", and "to". 


    ```r
    pub %>%
      unnest_tokens(output = word, input = abstract) %>%
      anti_join(stop_words, by = "word") %>%
      count(word, sort=TRUE)%>%
      top_n(20, n) %>%
      knitr::kable()

| word         |    n |
|:-------------|-----:|
| covid        | 7275 |
| 19           | 7080 |
| patients     | 4674 |
| cancer       | 3999 |
| prostate     | 3832 |
| disease      | 2574 |
| pre          | 2165 |
| eclampsia    | 2005 |
| preeclampsia | 1863 |
| treatment    | 1841 |
| clinical     | 1682 |
| risk         | 1588 |
| women        | 1327 |
| study        | 1299 |
| results      | 1281 |
| severe       | 1063 |
| diagnosis    | 1015 |
| pregnancy    | 1011 |
| data         |  945 |
| health       |  922 |

Removing stop changes what tokens appear as the most frequent. After
removing stop words, the 5 most common disease-relevant tokens are
“covid”, “patients”, “cancer”, “prostate”, and “disease”

``` r
#"term" is one column from pub
pub %>%
  filter(term == "covid") %>%
  unnest_tokens(output = token, input = abstract) %>%
  count(token, sort = TRUE) %>%
  anti_join(stop_words, by = c("token" = "word")) %>%
  top_n(5,n) %>%
  knitr::kable(caption = "5 Most Frequent Tokens for covid")
```

| token    |    n |
|:---------|-----:|
| covid    | 7275 |
| 19       | 7035 |
| patients | 2293 |
| disease  |  943 |
| pandemic |  800 |

5 Most Frequent Tokens for covid

``` r
pub %>%
  filter(term == "cystic fibrosis") %>%
  unnest_tokens(output = token, input = abstract) %>%
  count(token, sort = TRUE) %>%
  anti_join(stop_words, by = c("token" = "word")) %>%
  top_n(5,n) %>%
  knitr::kable(caption = "5 Most Frequent Tokens for cystic fibrosis")
```

| token    |   n |
|:---------|----:|
| fibrosis | 867 |
| cystic   | 862 |
| cf       | 625 |
| patients | 586 |
| disease  | 400 |

5 Most Frequent Tokens for cystic fibrosis

``` r
pub %>%
  filter(term == "meningitis    ") %>%
  unnest_tokens(output = token, input = abstract) %>%
  count(token, sort = TRUE) %>%
  anti_join(stop_words, by = c("token" = "word")) %>%
  top_n(5,n) %>%
  knitr::kable(caption = "5 Most Frequent Tokens for meningitis ")
```

| token |   n |
|:------|----:|

5 Most Frequent Tokens for meningitis

``` r
pub %>%
  filter(term == "preeclampsia") %>%
  unnest_tokens(output = token, input = abstract) %>%
  count(token, sort = TRUE) %>%
  anti_join(stop_words, by = c("token" = "word")) %>%
  top_n(5,n) %>%
  knitr::kable(caption = "5 Most Frequent Tokens for preeclampsia")
```

| token        |    n |
|:-------------|-----:|
| pre          | 2038 |
| eclampsia    | 2005 |
| preeclampsia | 1863 |
| women        | 1196 |
| pregnancy    |  969 |

5 Most Frequent Tokens for preeclampsia

``` r
pub %>%
  filter(term == "prostate cancer") %>%
  unnest_tokens(output = token, input = abstract) %>%
  count(token, sort = TRUE) %>%
  anti_join(stop_words, by = c("token" = "word")) %>%
  top_n(5,n) %>%
  knitr::kable(caption = "5 Most Frequent Tokens for prostate cancer")
```

| token     |    n |
|:----------|-----:|
| cancer    | 3840 |
| prostate  | 3832 |
| patients  |  934 |
| treatment |  926 |
| disease   |  652 |

5 Most Frequent Tokens for prostate cancer

``` r
pub %>%
  unnest_tokens(word, abstract) %>%
  anti_join(stop_words, by = "word") %>%
  group_by(term) %>% 
  count(word, term, sort = TRUE) %>%
  top_n(5, n) %>%
  arrange(term, desc(n)) %>%
  knitr::kable()
```

| term            | word         |    n |
|:----------------|:-------------|-----:|
| covid           | covid        | 7275 |
| covid           | 19           | 7035 |
| covid           | patients     | 2293 |
| covid           | disease      |  943 |
| covid           | pandemic     |  800 |
| cystic fibrosis | fibrosis     |  867 |
| cystic fibrosis | cystic       |  862 |
| cystic fibrosis | cf           |  625 |
| cystic fibrosis | patients     |  586 |
| cystic fibrosis | disease      |  400 |
| meningitis      | patients     |  446 |
| meningitis      | meningitis   |  429 |
| meningitis      | meningeal    |  219 |
| meningitis      | csf          |  206 |
| meningitis      | clinical     |  187 |
| preeclampsia    | pre          | 2038 |
| preeclampsia    | eclampsia    | 2005 |
| preeclampsia    | preeclampsia | 1863 |
| preeclampsia    | women        | 1196 |
| preeclampsia    | pregnancy    |  969 |
| prostate cancer | cancer       | 3840 |
| prostate cancer | prostate     | 3832 |
| prostate cancer | patients     |  934 |
| prostate cancer | treatment    |  926 |
| prostate cancer | disease      |  652 |

Question2: tokenize the abstracts into bigrams. Find the 10 most common
bigram and visualize them with ggplot2.

``` r
library(ggplot2)
pub %>%
  unnest_ngrams(bigram, abstract, n=2) %>%
  count(bigram, sort = TRUE) %>%
  top_n(10, n) %>%
  ggplot(aes(n, fct_reorder(bigram,n))) +
  geom_col()
```

![](hw3_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Question3: Calculate the TF-IDF value for each word-search term
combination. (here you want the search term to be the “document”) What
are the 5 tokens from each search term with the highest TF-IDF value?
How are the results different from the answers you got in question 1?

``` r
pub %>%
  unnest_tokens(word, abstract) %>%
  count(word, term) %>%
  bind_tf_idf(word, term, n) %>%
  group_by(term) %>%
  arrange(desc(tf_idf)) %>% #or combine the next line with this: arrange(desc(tf_idf), .by_group=TRUE)
  arrange(term) %>%
  top_n(5,tf_idf) %>%
  select(term,word,n,tf,idf,tf_idf) %>% #rearrange per term instead of word
  knitr::kable()
```

| term            | word            |    n |        tf |       idf |    tf_idf |
|:----------------|:----------------|-----:|----------:|----------:|----------:|
| covid           | covid           | 7275 | 0.0371050 | 1.6094379 | 0.0597183 |
| covid           | pandemic        |  800 | 0.0040803 | 1.6094379 | 0.0065670 |
| covid           | coronavirus     |  647 | 0.0032999 | 1.6094379 | 0.0053110 |
| covid           | sars            |  372 | 0.0018973 | 1.6094379 | 0.0030536 |
| covid           | cov             |  334 | 0.0017035 | 1.6094379 | 0.0027417 |
| cystic fibrosis | cf              |  625 | 0.0127188 | 0.9162907 | 0.0116541 |
| cystic fibrosis | fibrosis        |  867 | 0.0176435 | 0.5108256 | 0.0090127 |
| cystic fibrosis | cystic          |  862 | 0.0175417 | 0.5108256 | 0.0089608 |
| cystic fibrosis | cftr            |   86 | 0.0017501 | 1.6094379 | 0.0028167 |
| cystic fibrosis | sweat           |   83 | 0.0016891 | 1.6094379 | 0.0027184 |
| meningitis      | meningitis      |  429 | 0.0091942 | 1.6094379 | 0.0147974 |
| meningitis      | meningeal       |  219 | 0.0046935 | 1.6094379 | 0.0075539 |
| meningitis      | pachymeningitis |  149 | 0.0031933 | 1.6094379 | 0.0051394 |
| meningitis      | csf             |  206 | 0.0044149 | 0.9162907 | 0.0040453 |
| meningitis      | meninges        |  106 | 0.0022718 | 1.6094379 | 0.0036562 |
| preeclampsia    | eclampsia       | 2005 | 0.0142784 | 1.6094379 | 0.0229802 |
| preeclampsia    | preeclampsia    | 1863 | 0.0132672 | 1.6094379 | 0.0213527 |
| preeclampsia    | pregnancy       |  969 | 0.0069006 | 0.5108256 | 0.0035250 |
| preeclampsia    | maternal        |  797 | 0.0056757 | 0.5108256 | 0.0028993 |
| preeclampsia    | gestational     |  191 | 0.0013602 | 1.6094379 | 0.0021891 |
| prostate cancer | prostate        | 3832 | 0.0311890 | 1.6094379 | 0.0501967 |
| prostate cancer | androgen        |  305 | 0.0024824 | 1.6094379 | 0.0039953 |
| prostate cancer | psa             |  282 | 0.0022952 | 1.6094379 | 0.0036940 |
| prostate cancer | prostatectomy   |  215 | 0.0017499 | 1.6094379 | 0.0028164 |
| prostate cancer | castration      |  148 | 0.0012046 | 1.6094379 | 0.0019387 |

In question 1: the most frequent searching words for terms covid, cystic
fibrosis, meningitis, preeclampsia, and prostate cancer are covid,
fibrosis, patients, pre, and cancer correspondingly. While in question
3: the most frequent searching words for terms covid, cystic fibrosis,
meningitis, preeclampsia, and prostate cancer are covid, cf, meningitis,
eclampsia, and prostate correspondingly, which are different from those
in question 1.
