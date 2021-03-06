# Separating Spam from Ham
Solutions by John Bobo based on a problem set from MIT’s Analytics Edge MOOC  
May 27, 2016  


Nearly every email user has at some point encountered a "spam" email, which is an unsolicited message often advertising a product, containing links to malware, or attempting to scam the recipient. Roughly 80-90% of more than 100 billion emails sent each day are spam emails, most being sent from botnets of malware-infected computers. The remainder of emails are called "ham" emails.

As a result of the huge number of spam emails being sent across the Internet each day, most email providers offer a spam filter that automatically flags likely spam messages and separates them from the ham. Though these filters use a number of techniques (e.g. looking up the sender in a so-called "Blackhole List" that contains IP addresses of likely spammers), most rely heavily on the analysis of the contents of an email via text analytics.

In this homework problem, we will build and evaluate a spam filter using a publicly available dataset first described in the 2006 conference paper "Spam Filtering with Naive Bayes -- Which Naive Bayes?" by V. Metsis, I. Androutsopoulos, and G. Paliouras. The "ham" messages in this dataset come from the inbox of former Enron Managing Director for Research Vincent Kaminski, one of the inboxes in the Enron Corpus. One source of spam messages in this dataset is the SpamAssassin corpus, which contains hand-labeled spam messages contributed by Internet users. The remaining spam was collected by Project Honey Pot, a project that collects spam messages and identifies spammers by publishing email address that humans would know not to contact but that bots might target with spam. The full dataset we will use was constructed as roughly a 75/25 mix of the ham and spam messages.

The dataset contains just two fields:

- **text**: The text of the email.
- **spam**: A binary variable indicating if the email was spam.

***

#### Problem 1.1 - Loading the Dataset

(1 point possible)

Begin by loading the dataset [emails.csv](https://d37djvu3ytnwxt.cloudfront.net/asset-v1:MITx+15.071x_3+1T2016+type@asset+block/emails.csv) into a data frame called emails. Remember to pass the `stringsAsFactors=FALSE` option when loading the data.

```r
emails <- read.csv("/Users/johnbobo/analytics_edge/data/emails.csv",
                  stringsAsFactors = FALSE)
```

*How many emails are in the dataset?*

```r
answer <- nrow(emails)
```
**Answer:** 5728

***

#### Problem 1.2 - Loading the Dataset

(1 point possible)
*How many of the emails are spam?*

```r
answer <- sum(emails$spam)
```
**Answer:** 1368

***

#### Problem 1.3 - Loading the Dataset

(1 point possible)
*Which word appears at the beginning of every email in the dataset?*


```r
substring(emails$text[1],1,7)
```

```
## [1] "Subject"
```
**Answer:** Subject

***

#### Problem 1.4 - Loading the Dataset

(1 point possible)
*Could a spam classifier potentially benefit from including the frequency of the word that appears in every email?*  

**Answer:** Yes, the number of times the word appears in a single email might help us differentiate spam from ham.  

***

#### Problem 1.5 - Loading the Dataset

(1 point possible)
The nchar() function counts the number of characters in a piece of text. *How many characters are in the longest email in the dataset (where longest is measured in terms of the maximum number of characters)?*

```r
answer <- max(nchar(emails$text))
```
**Answer:** 43952

***

#### Problem 1.6 - Loading the Dataset

(1 point possible)
*Which row contains the shortest email in the dataset? (Just like in the previous problem, shortest is measured in terms of the fewest number of characters.)*

```r
answer <- which.min(nchar(emails$text))
```
**Answer:** 1992

***

#### Problem 2.1 - Preparing the Corpus

(2 points possible)
Follow the standard steps to build and pre-process the corpus:

1) Build a new corpus variable called corpus.

2) Using tm_map, convert the text to lowercase.

3) Using tm_map, remove all punctuation from the corpus.

4) Using tm_map, remove all English stopwords from the corpus.

5) Using tm_map, stem the words in the corpus.

6) Build a document term matrix from the corpus, called dtm.


```r
library(tm)
```

```
## Loading required package: NLP
```

```r
vectorToDTM <-function(data){
    corpus = Corpus(VectorSource(data))
    corpus = tm_map(corpus, content_transformer(tolower))
    corpus = tm_map(corpus, PlainTextDocument)
    corpus = tm_map(corpus, removePunctuation)
    corpus = tm_map(corpus, removeWords, stopwords('english'))
    corpus = tm_map(corpus, stemDocument)
    
    dtm = DocumentTermMatrix(corpus)
    return(dtm)
}

dtm <- vectorToDTM(emails$text)
dtm
```

```
## <<DocumentTermMatrix (documents: 5728, terms: 28687)>>
## Non-/sparse entries: 481719/163837417
## Sparsity           : 100%
## Maximal term length: 24
## Weighting          : term frequency (tf)
```

*How many terms are in dtm?*  

**Answer:** 28687

***

#### Problem 2.2 - Preparing the Corpus

(1 point possible)
To obtain a more reasonable number of terms, limit dtm to contain terms appearing in at least 5% of documents, and store this result as spdtm (don't overwrite dtm, because we will use it in a later step of this homework).

```r
spdtm <- removeSparseTerms(dtm, .95)
spdtm
```

```
## <<DocumentTermMatrix (documents: 5728, terms: 330)>>
## Non-/sparse entries: 213551/1676689
## Sparsity           : 89%
## Maximal term length: 10
## Weighting          : term frequency (tf)
```
*How many terms are in spdtm?*
**Answer:** 330

***

#### Problem 2.3 - Preparing the Corpus

(2 points possible)
Build a data frame called emailsSparse from spdtm, and use the make.names function to make the variable names of emailsSparse valid.

```r
emailsSparse <- as.data.frame(as.matrix(spdtm))
colnames(emailsSparse) <- make.names(colnames(emailsSparse))
```

colSums() is an R function that returns the sum of values for each variable in our data frame. Our data frame contains the number of times each word stem (columns) appeared in each email (rows). Therefore, colSums(emailsSparse) returns the number of times a word stem appeared across all the emails in the dataset. *What is the word stem that shows up most frequently across all the emails in the dataset?*

```r
answer <- names(which.max(colSums(emailsSparse)))
```
**Answer:** enron

***

#### Problem 2.4 - Preparing the Corpus

(1 point possible)
Add a variable called "spam" to emailsSparse containing the email spam labels. You can do this by copying over the "spam" variable from the original data frame (remember how we did this in the Twitter lecture).

```r
emailsSparse$spam <- emails$spam
```

*How many word stems appear at least 5000 times in the ham emails in the dataset?*

```r
answer <- sum(colSums(subset(emailsSparse, spam == 0)) > 5000)
```
**Answer:** 6

***

#### Problem 2.5 - Preparing the Corpus

(1 point possible)
*How many word stems appear at least 1000 times in the spam emails in the dataset?*
remember not to count the dependent variable we just added.

```r
answer <- sum(colSums(subset(emailsSparse, spam == 1)) > 1000) - 
                                                (sum(emailsSparse$spam) > 1000)
```
**Answer:** 3

***

#### Problem 2.6 - Preparing the Corpus

(1 point possible)
The lists of most common words are significantly different between the spam and ham emails. *What does this likely imply?*  

**Answer:** The frequencies of these most common words are likely to help differentiate between spam and ham.

***

#### Problem 2.7 - Preparing the Corpus

(1 point possible)
Several of the most common word stems from the ham documents, such as "enron", "hou" (short for Houston), "vinc" (the word stem of "Vince") and "kaminski", are likely specific to Vincent Kaminski's inbox. *What does this mean about the applicability of the text analytics models we will train for the spam filtering problem?*  

**Answer:** The models we build are personalized, and would need to be further tested before being used as a spam filter for another person.

***

#### Problem 3.1 - Building machine learning models

(3 points possible)
First, convert the dependent variable to a factor with "emailsSparse$spam = as.factor(emailsSparse$spam)".

```r
emailsSparse$spam <- as.factor(emailsSparse$spam)
```

Next, set the random seed to 123 and use the sample.split function to split emailsSparse 70/30 into a training set called "train" and a testing set called "test". Make sure to perform this step on emailsSparse instead of emails.

```r
library(caTools)
set.seed(123)

spl <- sample.split(emailsSparse$spam, SplitRatio = 0.7)
train <- subset(emailsSparse, spl == TRUE)
test <- subset(emailsSparse, spl == FALSE)
```

Using the training set, train the following three machine learning models. The models should predict the dependent variable "spam", using all other available variables as independent variables. Please be patient, as these models may take a few minutes to train.

1) A logistic regression model called spamLog. You may see a warning message here - we'll discuss this more later.

```r
spamLog <- glm(spam ~ ., data=train, family='binomial')
```

```
## Warning: glm.fit: algorithm did not converge
```

```
## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred
```

2) A CART model called spamCART, using the default parameters to train the model (don't worry about adding minbucket or cp). Remember to add the argument method="class" since this is a binary classification problem.

```r
library(rpart)
library(rpart.plot)

spamCART <- rpart(spam ~ ., data=train, method='class')
```

3) A random forest model called spamRF, using the default parameters to train the model (don't worry about specifying ntree or nodesize). Directly before training the random forest model, set the random seed to 123 (even though we've already done this earlier in the problem, it's important to set the seed right before training the model so we all obtain the same results. Keep in mind though that on certain operating systems, your results might still be slightly different).

```r
library(randomForest)
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```r
set.seed(123)
spamRF <- randomForest(spam ~ ., data=train)
```

For each model, obtain the predicted spam probabilities for the training set. Be careful to obtain probabilities instead of predicted classes, because we will be using these values to compute training set AUC values. Recall that you can obtain probabilities for CART models by not passing any type parameter to the predict() function, and you can obtain probabilities from a random forest by adding the argument type="prob". For CART and random forest, you need to select the second column of the output of the predict() function, corresponding to the probability of a message being spam.

```r
predTrainLog <- predict(spamLog, type='response')
predTrainCART <- predict(spamCART)[,2]
predTrainRF <- predict(spamRF, type='prob')[,2]
```

You may have noticed that training the logistic regression model yielded the messages "algorithm did not converge" and "fitted probabilities numerically 0 or 1 occurred". Both of these messages often indicate overfitting and the first indicates particularly severe overfitting, often to the point that the training set observations are fit perfectly by the model. Let's investigate the predicted probabilities from the logistic regression model.

*How many of the training set predicted probabilities from spamLog are less than 0.00001?*


```r
low <- sum(predTrainLog < 0.00001)
```
**Answer:** 3046
 
*How many of the training set predicted probabilities from spamLog are more than 0.99999?*

```r
high <- sum(predTrainLog > .99999)
```

**Answer:** 954


*How many of the training set predicted probabilities from spamLog are between 0.00001 and 0.99999?*

```r
answer <- length(predTrainLog) - low - high
```
**Answer:** 10

***

#### Problem 3.2 - Building Machine Learning Models

(1 point possible)
*How many variables are labeled as significant (at the p=0.05 level) in the logistic regression summary output?*  

```r
summary(spamLog)
```

**Answer**: 0. This can be seen from the summary of spamLog (supressed due to length).

***

#### Problem 3.3 - Building Machine Learning Models

(1 point possible)
*How many of the word stems "enron", "hou", "vinc", and "kaminski" appear in the CART tree?* Recall that we suspect these word stems are specific to Vincent Kaminski and might affect the generalizability of a spam filter built with his ham data.

```r
prp(spamCART)
```

![](separating_spam_from_ham_files/figure-html/unnamed-chunk-24-1.png)<!-- -->

**Answer:** 2

***

#### Problem 3.4 - Building Machine Learning Models

(1 point possible)
*What is the training set accuracy of spamLog, using a threshold of 0.5 for predictions?*

```r
table(train$spam, predTrainLog >= 0.5)
```

```
##    
##     FALSE TRUE
##   0  3052    0
##   1     4  954
```
**Answer:** 0.999

#### Problem 3.5 - Building Machine Learning Models

(1 point possible)
*What is the training set AUC of spamLog?*  

```r
library(ROCR)
```

```
## Loading required package: gplots
```

```
## 
## Attaching package: 'gplots'
```

```
## The following object is masked from 'package:stats':
## 
##     lowess
```

```r
predROCR <- prediction(predTrainLog, train$spam)
auc <- performance(predROCR, 'auc')@y.values
```
**Answer:** 1

***

#### Problem 3.6 - Building Machine Learning Models

(1 point possible)
*What is the training set accuracy of spamCART, using a threshold of 0.5 for predictions?*

```r
table(train$spam, predTrainCART >= 0.5)
```

```
##    
##     FALSE TRUE
##   0  2885  167
##   1    64  894
```
**Answer:** 0.942

***

#### Problem 3.7 - Building Machine Learning Models

(1 point possible)
*What is the training set AUC of spamCART?* 

```r
library(ROCR)

predROCR <- prediction(predTrainCART, train$spam)
auc <- performance(predROCR, 'auc')@y.values
```
**Answer:** 0.97

***

#### Problem 3.8 - Building Machine Learning Models

(1 point possible)
*What is the training set accuracy of spamRF, using a threshold of 0.5 for predictions?*

```r
table(train$spam, predTrainRF >= 0.5)
```

```
##    
##     FALSE TRUE
##   0  3013   39
##   1    44  914
```
**Answer:** 0.979

***

#### Problem 3.9 - Building Machine Learning Models

(2 points possible)
*What is the training set AUC of spamRF?*


```r
library(ROCR)

predROCR <- prediction(predTrainRF, train$spam)
auc <- performance(predROCR, 'auc')@y.values
```
**Answer:** 0.998

***

#### Problem 3.10 - Building Machine Learning Models

(1 point possible)
*Which model had the best training set performance, in terms of accuracy and AUC?*

**Answer:** Logistic Regression.

***

#### Problem 4.1 - Evaluating on the Test Set

(1 point possible)
Obtain predicted probabilities for the testing set for each of the models, again ensuring that probabilities instead of classes are obtained.

```r
predTestLog <- predict(spamLog, newdata=test, type='response')
predTestCART <- predict(spamCART, newdata=test)[,2]
predTestRF <- predict(spamRF, newdata=test, type='prob')[,2]
```

*What is the testing set accuracy of spamLog, using a threshold of 0.5 for predictions?*

```r
table(test$spam, predTestLog >= 0.5)
```

```
##    
##     FALSE TRUE
##   0  1257   51
##   1    34  376
```
**Answer:** 0.951

***

#### Problem 4.2 - Evaluating on the Test Set

(1 point possible)
*What is the testing set AUC of spamLog?*

```r
predROCR <- prediction(predTestLog, test$spam)
auc <- performance(predROCR, 'auc')@y.values
```
**Answer:** 0.963

***

#### Problem 4.3 - Evaluating on the Test Set

(1 point possible)
*What is the testing set accuracy of spamCART, using a threshold of 0.5 for predictions?*

```r
table(test$spam, predTestCART >= 0.5)
```

```
##    
##     FALSE TRUE
##   0  1228   80
##   1    24  386
```
**Answer:** 0.939

***

#### Problem 4.4 - Evaluating on the Test Set

(1 point possible)
*What is the testing set AUC of spamCART?*

```r
predROCR <- prediction(predTestCART, test$spam)
auc <- performance(predROCR, 'auc')@y.values
```
**Answer:** 0.963

***

#### Problem 4.5 - Evaluating on the Test Set

(1 point possible)
*What is the testing set accuracy of spamRF, using a threshold of 0.5 for predictions?*

```r
table(test$spam, predTestRF >= 0.5)
```

```
##    
##     FALSE TRUE
##   0  1290   18
##   1    24  386
```
**Answer:** 0.976

***

#### Problem 4.6 - Evaluating on the Test Set

(1 point possible)
*What is the testing set AUC of spamRF?*

```r
predROCR <- prediction(predTestRF, test$spam)
auc <- performance(predROCR, 'auc')@y.values
```
**Answer:** 0.998

***

#### Problem 4.7 - Evaluating on the Test Set

(1 point possible)
*Which model had the best testing set performance, in terms of accuracy and AUC?*

**Answer:** The random forest model.

#### Problem 4.8 - Evaluating on the Test Set

(1 point possible)
*Which model demonstrated the greatest degree of overfitting?*  

**Answer:** The logistic regression model.  It had the highest accuracy and auc on the training set and the lowest on the test set.
