# AUTOMATING REVIEWS IN MEDICINE
Solutions by John Bobo based on a problem set from MIT’s Analytics Edge MOOC  
May 26, 2016  



The medical literature is enormous. Pubmed, a database of medical publications maintained by the U.S. National Library of Medicine, has indexed over 23 million medical publications. Further, the rate of medical publication has increased over time, and now there are nearly 1 million new publications in the field each year, or more than one per minute.

The large size and fast-changing nature of the medical literature has increased the need for reviews, which search databases like Pubmed for papers on a particular topic and then report results from the papers found. While such reviews are often performed manually, with multiple people reviewing each search result, this is tedious and time consuming. In this problem, we will see how text analytics can be used to automate the process of information retrieval.

The dataset consists of the titles (variable _title_) and abstracts (variable _abstract_) of papers retrieved in a [Pubmed](http://www.ncbi.nlm.nih.gov/pubmed) search. Each search result is labeled with whether the paper is a clinical trial testing a drug therapy for cancer (variable _trial_). These labels were obtained by two people reviewing each search result and accessing the actual paper if necessary, as part of a literature review of clinical trials testing drug therapies for advanced and metastatic breast cancer.

***

#### Problem 1.1 - Loading the Data

(1 point possible)

Load [clinical_trial.csv](https://d37djvu3ytnwxt.cloudfront.net/asset-v1:MITx+15.071x_3+1T2016+type@asset+block/clinical_trial.csv) into a data frame called trials (remembering to add the argument stringsAsFactors=FALSE).

```r
trials <- read.csv("/Users/johnbobo/analytics_edge/data/clinical_trial.csv",
                   stringsAsFactors = FALSE)
```

We can use R's string functions to learn more about the titles and abstracts of the located papers. The nchar() function counts the number of characters in a piece of text. Using the nchar() function on the variables in the data frame, answer the following questions:

*How many characters are there in the longest abstract? (Longest here is defined as the abstract with the largest number of characters.)*  

```r
answer <- max(nchar(trials$abstract))
```
**Answer:** 3708

***

#### Problem 1.2 - Loading the Data

(1 point possible)
*How many search results provided no abstract?*  

```r
answer <- sum(nchar(trials$abstract) == 0)
```
**Answer:** 112

***

#### Problem 1.3 - Loading the Data

(1 point possible)
Find the observation with the minimum number of characters in the title (the variable "title") out of all of the observations in this dataset. What is the text of the title of this article? Include capitalization and punctuation in your response, but don't include the quotes.

```r
answer <- trials$title[which.min(nchar(trials$title))]
```
**Answer:** A decade of letrozole: FACE.

***

#### Problem 2.1 - Preparing the Corpus

(4 points possible)
Because we have both title and abstract information for trials, we need to build two corpera instead of one. Name them corpusTitle and corpusAbstract.

Following the commands from lecture, perform the following tasks (you might need to load the "tm" package first if it isn't already loaded). Make sure to perform them in this order.

1) Convert the title variable to corpusTitle and the abstract variable to corpusAbstract.

2) Convert corpusTitle and corpusAbstract to lowercase. After performing this step, remember to run the lines:

`corpusTitle = tm_map(corpusTitle, PlainTextDocument)`

`corpusAbstract = tm_map(corpusAbstract, PlainTextDocument)`

3) Remove the punctuation in corpusTitle and corpusAbstract.

4) Remove the English language stop words from corpusTitle and corpusAbstract.

5) Stem the words in corpusTitle and corpusAbstract (each stemming might take a few minutes).

6) Build a document term matrix called dtmTitle from corpusTitle and dtmAbstract from corpusAbstract.

7) Limit dtmTitle and dtmAbstract to terms with sparseness of at most 95% (aka terms that appear in at least 5% of documents).

8) Convert dtmTitle and dtmAbstract to data frames (keep the names dtmTitle and dtmAbstract).


```r
library(tm)
```

```
## Loading required package: NLP
```

```r
vectorToDTM <-function(data, sparse){
    corpus = Corpus(VectorSource(data))
    corpus = tm_map(corpus, content_transformer(tolower))
    corpus = tm_map(corpus, PlainTextDocument)
    corpus = tm_map(corpus, removePunctuation)
    corpus = tm_map(corpus, removeWords, stopwords('english'))
    corpus = tm_map(corpus, stemDocument)
    
    dtm = DocumentTermMatrix(corpus)
    dtmSparse = removeSparseTerms(dtm, sparse)
    df = as.data.frame(as.matrix(dtmSparse))
    return(df)
}
dtmTitle <- vectorToDTM(trials$title, .95)
dtmAbstract <- vectorToDTM(trials$abstract, .95)
```
*How many terms remain in `dtmTitle` after removing sparse terms (aka how many columns does it have)?*  

**Answer:** 31

*How many terms remain in `dtmAbstract`?*  

**Answer:** 335

***

####  Problem 2.2 - Preparing the Corpus

(1 point possible)
*What is the most likely reason why dtmAbstract has so many more terms than dtmTitle?*  

**Answer:** Abstracts tend to have many more words than titles.

***

#### Problem 2.3 - Preparing the Corpus

(1 point possible)
*What is the most frequent word stem across all the abstracts?*

```r
sort(colSums(dtmAbstract), decreasing = TRUE)[1]
```

```
## patient 
##    8381
```
**Answer:** patient.  This shouldn't be too surprising.

***


