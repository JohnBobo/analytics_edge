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
