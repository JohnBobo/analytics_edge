# Detecting Vandalism on Wikipedia
By John Bobo based on a problem set from MIT’s Analytics Edge MOOC  
May 25, 2016  



[Wikipedia](http://en.wikipedia.org/wiki/Wikipedia) is a free online encyclopedia that anyone can edit and contribute to. It is available in many languages and is growing all the time. On the English language version of Wikipedia:

- There are currently [4.7 million pages](http://en.wikipedia.org/wiki/Wikipedia:Size_of_Wikipedia).
- There have been a total over [760 million edits](http://en.wikipedia.org/wiki/Wikipedia:Pruning_article_revisions) (also called revisions) over its lifetime.
- There are approximately [130,000 edits per day](http://en.wikipedia.org/wiki/Wikipedia:WikiProject_Editing_trends/Raw_data/Revisions_per_day).

One of the consequences of being editable by anyone is that some people _vandalize_ pages. This can take the form of removing content, adding promotional or inappropriate content, or more subtle shifts that change the meaning of the article. With this many articles and edits per day it is difficult for humans to detect all instances of vandalism and _revert_ (undo) them. As a result, Wikipedia uses _bots_ - computer programs that automatically revert edits that look like vandalism. In this assignment we will attempt to develop a vandalism detector that uses machine learning to distinguish between a valid edit and vandalism.

The data for this problem is based on the revision history of the page [Language](http://en.wikipedia.org/wiki/Language). Wikipedia provides a history for each page that consists of the state of the page at each revision. Rather than manually considering each revision, a script was run that checked whether edits stayed or were reverted. If a change was eventually reverted then that revision is marked as vandalism. This may result in some misclassifications, but the script performs well enough for our needs.

As a result of this preprocessing, some common processing tasks have already been done, including lower-casing and punctuation removal. The columns in the dataset are:

- Vandal = 1 if this edit was vandalism, 0 if not.
- Minor = 1 if the user marked this edit as a "minor edit", 0 if not.
- Loggedin = 1 if the user made this edit while using a Wikipedia account, 0 if they did not.
- Added = The _unique_ words added.
- Removed = The _unique_ words removed.

Notice the repeated use of _unique_. The data we have available is not the traditional bag of words - rather it is the set of words that were removed or added. For example, if a word was removed multiple times in a revision it will only appear one time in the "Removed" column.

#### Problem 1.1 - Bags of Words

(1 point possible)
Load the data wiki.csv with the option stringsAsFactors=FALSE, calling the data frame "wiki". Convert the "Vandal" column to a factor using the command wiki$Vandal = as.factor(wiki$Vandal).

```r
wiki <- read.csv("/Users/johnbobo/analytics_edge/data/wiki.csv", stringsAsFactors = FALSE)

wiki$Vandal <- as.factor(wiki$Vandal)
```

*How many cases of vandalism were detected in the history of this page?*

```r
table(wiki$Vandal)
```

```
## 
##    0    1 
## 2061 1815
```
**Answer:** 1815.

***

#### Problem 1.2 - Bags of Words

(2 points possible)
We will now use the bag of words approach to build a model. We have two columns of textual data, with different meanings. For example, adding rude words has a different meaning to removing rude words. We'll start like we did in class by building a document term matrix from the Added column. The text already is lowercase and stripped of punctuation. So to pre-process the data, just complete the following four steps:

1) Create the corpus for the Added column, and call it "corpusAdded".

```r
library(tm)
```

```
## Loading required package: NLP
```

```r
corpusAdded <- Corpus(VectorSource(wiki$Added))
```

2) Remove the English-language stopwords.

```r
corpusAdded <- tm_map(corpusAdded, removeWords, stopwords('english'))
```

3) Stem the words.

```r
corpusAdded <- tm_map(corpusAdded, stemDocument)
```

4) Build the DocumentTermMatrix, and call it dtmAdded.

```r
dtmAdded <- DocumentTermMatrix(corpusAdded)
```

*How many terms appear in dtmAdded?*

```r
dtmAdded
```

```
## <<DocumentTermMatrix (documents: 3876, terms: 6675)>>
## Non-/sparse entries: 15368/25856932
## Sparsity           : 100%
## Maximal term length: 784
## Weighting          : term frequency (tf)
```
**Answer:** 6675

***

#### Problem 1.3 - Bags of Words

(1 point possible)
Filter out sparse terms by keeping only terms that appear in 0.3% or more of the revisions, and call the new matrix sparseAdded. *How many terms appear in sparseAdded?*

```r
sparseAdded <- removeSparseTerms(dtmAdded, .997)
sparseAdded
```

```
## <<DocumentTermMatrix (documents: 3876, terms: 166)>>
## Non-/sparse entries: 2681/640735
## Sparsity           : 100%
## Maximal term length: 28
## Weighting          : term frequency (tf)
```
**Answer:** 166

***

#### Problem 1.4 - Bags of Words

(2 points possible)
Convert sparseAdded to a data frame called wordsAdded, and then prepend all the words with the letter A.

```r
wordsAdded <- as.data.frame(as.matrix(sparseAdded))

colnames(wordsAdded) = paste("A", colnames(wordsAdded))
```
Now repeat all of the steps we've done so far (create a corpus, remove stop words, stem the document, create a sparse document term matrix, and convert it to a data frame) to create a Removed bag-of-words dataframe, called wordsRemoved, except this time, prepend all of the words with the letter R:

```r
vectorToCorpusDF <-function(data, sparse){
    corpus = Corpus(VectorSource(data))
    corpus = tm_map(corpus, content_transformer(tolower))
    corpus = tm_map(corpus, removePunctuation)
    corpus = tm_map(corpus, removeWords, stopwords('english'))
    corpus = tm_map(corpus, stemDocument)
    
    dtm = DocumentTermMatrix(corpus)
    dtmSparse = removeSparseTerms(dtm, sparse)
    df = as.data.frame(as.matrix(dtmSparse))
    
    return(df)
}
vectorToDTM <-function(data, sparse){
    corpus = Corpus(VectorSource(data))
    corpus = tm_map(corpus, content_transformer(tolower))
    corpus = tm_map(corpus, removePunctuation)
    corpus = tm_map(corpus, removeWords, stopwords('english'))
    corpus = tm_map(corpus, stemDocument)
    
    dtm = DocumentTermMatrix(corpus)
    return(dtm)
}

dtmRemoved <- vectorToDTM(wiki$Removed)
wordsRemoved <- vectorToCorpusDF(wiki$Removed, .997)

colnames(wordsRemoved) = paste("R", colnames(wordsRemoved))
```
*How many words are in the wordsRemoved data frame?*  

**Answer:** 162

***

#### Problem 1.5 - Bags of Words

(2 points possible)
Combine the two data frames into a data frame called wikiWords.

```r
wikiWords <- cbind(wordsAdded, wordsRemoved)
```
Then add the Vandal column. 

```r
wikiWords$Vandal <- wiki$Vandal
```
Set the random seed to 123 and then split the data set using sample.split from the "caTools" package to put 70% in the training set.

```r
library(caTools)
set.seed(123)

spl <- sample.split(wikiWords$Vandal, SplitRatio = 0.7)
train <- subset(wikiWords, spl == TRUE)
test <- subset(wikiWords, spl == FALSE)
```

*What is the accuracy on the test set of a baseline method that always predicts "not vandalism" (the most frequent outcome)?*

```r
baseline_accuracy <- mean(test$Vandal == '0')
```
**Answer:** 0.531

***

#### Problem 1.6 - Bags of Words

(2 points possible)
Build a CART model to predict Vandal, using all of the other variables as independent variables. Use the training set to build the model and the default parameters (don't set values for minbucket or cp).

```r
library(rpart)
library(rpart.plot)

wikiCART <- rpart(Vandal ~ ., data=train, method='class')
```

*What is the accuracy of the model on the test set, using a threshold of 0.5? (Remember that if you add the argument type="class" when making predictions, the output of predict will automatically use a threshold of 0.5.)*

```r
predCART <- predict(wikiCART, newdata=test, type='class')
table(test$Vandal, predCART)
```

```
##    predCART
##       0   1
##   0 618   0
##   1 533  12
```
**Answer:** 0.542

***

#### Problem 1.7 - Bags of Words

(1 point possible)
Plot the CART tree. *How many word stems does the CART model use?*

```r
prp(wikiCART)
```

![](detecting_vandalism_on_wikipedia_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

**Answer:** 2 words.

***

#### Problem 1.8 - Bags of Words

(1 point possible)
*Given the performance of the CART model relative to the baseline, what is the best explanation of these results?*

**Answer:** Although it beats the baseline, bag of words is not very predictive for this problem.

***

#### Problem 2.1 - Problem-specific Knowledge

(1 point possible)
We weren't able to improve on the baseline using the raw textual information. More specifically, the words themselves were not useful. There are other options though, and in this section we will try two techniques - identifying a key class of words, and counting words.

The key class of words we will use are website addresses. "Website addresses" (also known as URLs - Uniform Resource Locators) are comprised of two main parts. An example would be "http://www.google.com". The first part is the protocol, which is usually "http" (HyperText Transfer Protocol). The second part is the address of the site, e.g. "www.google.com". We have stripped all punctuation so links to websites appear in the data as one word, e.g. "httpwwwgooglecom". We hypothesize that given that a lot of vandalism seems to be adding links to promotional or irrelevant websites, the presence of a web address is a sign of vandalism.

We can search for the presence of a web address in the words added by searching for "http" in the Added column. The grepl function returns TRUE if a string is found in another string, e.g.

`grepl("cat","dogs and cats",fixed=TRUE) # TRUE`

`grepl("cat","dogs and rats",fixed=TRUE) # FALSE`

Create a copy of your dataframe from the previous question:

```r
wikiWords2 <- wikiWords
```
Make a new column in wikiWords2 that is 1 if "http" was in Added:

```r
wikiWords2$HTTP <- ifelse(grepl("http",wiki$Added,fixed=TRUE), 1, 0)
```
*Based on this new column, how many revisions added a link?*

```r
answer <- sum(wikiWords2$HTTP)
```
**Answer:** 217

***

#### Problem 2.2 - Problem-Specific Knowledge

(2 points possible)
In problem 1.5, you computed a vector called `spl` that identified the observations to put in the training and testing sets. Use that variable (do not recompute it with sample.split) to make new training and testing sets:

```r
wikiTrain2 <- subset(wikiWords2, spl==TRUE)
wikiTest2 <- subset(wikiWords2, spl==FALSE)
```
Then create a new CART model using this new variable as one of the independent variables.

```r
wikiCART2 <- rpart(Vandal ~ ., data=wikiTrain2, method='class')
testPred2 <- predict(wikiCART2, newdata=wikiTest2, type='class')
prp(wikiCART2)
```

![](detecting_vandalism_on_wikipedia_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

*What is the new accuracy of the CART model on the test set, using a threshold of 0.5?*

```r
table(wikiTest2$Vandal, testPred2)
```

```
##    testPred2
##       0   1
##   0 609   9
##   1 488  57
```
**Answer:** 0.573

***

#### Problem 2.3 - Problem-Specific Knowledge

(1 point possible)
Another possibility is that the number of words added and removed is predictive, perhaps more so than the actual words themselves. We already have a word count available in the form of the document-term matrices (DTMs).

Sum the rows of dtmAdded and dtmRemoved and add them as new variables in your data frame wikiWords2 (called NumWordsAdded and NumWordsRemoved) by using the following commands:

```r
wikiWords2$NumWordsAdded <- rowSums(as.matrix(dtmAdded))
wikiWords2$NumWordsRemoved <- rowSums(as.matrix(dtmRemoved))
```
*What is the average number of words added?*

```r
answer <- mean(wikiWords2$NumWordsAdded)
```
**Answer:** 4.05

***

#### Problem 2.4 - Problem-Specific Knowledge

(2 points possible)
In problem 1.5, you computed a vector called "spl" that identified the observations to put in the training and testing sets. Use that variable (do not recompute it with sample.split) to make new training and testing sets with wikiWords2. Create the CART model again (using the training set and the default parameters).

```r
train <- subset(wikiWords2, spl == TRUE)
test <- subset(wikiWords2, spl == FALSE)

cart <- rpart(Vandal ~ ., data=train, method='class')
predTest <- predict(cart, newdata=test, type='class')
prp(cart)
```

![](detecting_vandalism_on_wikipedia_files/figure-html/unnamed-chunk-26-1.png)<!-- -->

*What is the new accuracy of the CART model on the test set?*

```r
table(test$Vandal, predTest)
```

```
##    predTest
##       0   1
##   0 514 104
##   1 297 248
```
**Answer:** 0.655


***

#### Problem 3.1 - Using Non-Textual Data

(2 points possible)
We have two pieces of "metadata" (data about data) that we haven't yet used. Make a copy of wikiWords2, and call it wikiWords3:

```r
wikiWords3 <- wikiWords2
```
Then add the two original variables Minor and Loggedin to this new data frame:

```r
wikiWords3$Minor <- wiki$Minor
wikiWords3$Loggedin <- wiki$Loggedin
```
In problem 1.5, you computed a vector called "spl" that identified the observations to put in the training and testing sets. Use that variable (do not recompute it with sample.split) to make new training and testing sets with wikiWords3.

```r
train <- subset(wikiWords3, spl == TRUE)
test <- subset(wikiWords3, spl == FALSE)
```

Build a CART model using all the training data.

```r
cart <- rpart(Vandal ~ ., data=train, method='class')
predTest <- predict(cart, newdata=test, type='class')
prp(cart)
```

![](detecting_vandalism_on_wikipedia_files/figure-html/unnamed-chunk-31-1.png)<!-- -->

*What is the new accuracy of the CART model on the test set?*

```r
table(test$Vandal, predTest)
```

```
##    predTest
##       0   1
##   0 595  23
##   1 304 241
```
**Answer:** 0.719

***

#### Problem 3.2 - Using Non-Textual Data

(1 point possible)
There is a substantial difference in the accuracy of the model using the meta data. Is this because we made a more complicated model? *How many splits are there in the tree?*  

**Answer:** There are three splits.  Adding the metadata significantly improved the accuracy of our model without making it more complicated.
