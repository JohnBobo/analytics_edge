# Popularity of Music Records
By John Bobo based on a problem set from MIT’s Analytics Edge MOOC  
May 13, 2016  


The music industry has a well-developed market with a global annual revenue around $15 billion. The recording industry is highly competitive and is dominated by three big production companies which make up nearly 82% of the total annual album sales.   
  
Artists are at the core of the music industry and record labels provide them with the necessary resources to sell their music on a large scale. A record label incurs numerous costs (studio recording, marketing, distribution, and touring) in exchange for a percentage of the profits from album sales, singles and concert tickets.  
  
Unfortunately, the success of an artist's release is highly uncertain: a single may be extremely popular, resulting in widespread radio play and digital downloads, while another single may turn out quite unpopular, and therefore unprofitable.   
  
Knowing the competitive nature of the recording industry, record labels face the fundamental decision problem of which musical releases to support to maximize their financial success.   
  
How can we use analytics to predict the popularity of a song? In this assignment, we challenge ourselves to predict whether a song will reach a spot in the Top 10 of the Billboard Hot 100 Chart.

Taking an analytics approach, we aim to use information about a song's properties to predict its popularity. The dataset [songs.csv](https://d37djvu3ytnwxt.cloudfront.net/asset-v1:MITx+15.071x_3+1T2016+type@asset+block/songs.csv) consists of all songs which made it to the Top 10 of the Billboard Hot 100 Chart from 1990-2010 plus a sample of additional songs that didn't make the Top 10. This data comes from three sources:[Wikipedia](http://en.wikipedia.org/wiki/Billboard_Hot_100), [Billboard.com](http://www.billboard.com/), and [EchoNest](http://echonest.com/).

The variables included in the dataset either describe the artist or the song, or they are associated with the following song attributes: time signature, loudness, key, pitch, tempo, and timbre.

Here's a detailed description of the variables:

- **year** = the year the song was released
- **songtitle** = the title of the song
- **artistname** = the name of the artist of the song
- **songID** and **artistID** = identifying variables for the song and artist
- **timesignature** and **timesignature_confidence** = a variable estimating the time signature of the song, and the confidence in the estimate
-  **loudness** = a continuous variable indicating the average amplitude of the audio in decibels
-  **tempo** and **tempo_confidence** = a variable indicating the estimated beats per minute of the song, and the confidence in the estimate
-  **key** and **key_confidence** = a variable with twelve levels indicating the estimated key of the song (C, C#, . . ., B), and the confidence in the estimate
-  **energy** = a variable that represents the overall acoustic energy of the song, using a mix of features such as loudness
- **pitch** = a continuous variable that indicates the pitch of the song
-  **timbre_0_min**, **timbre_0_max**, **timbre_1_min**, **timbre_1_max**, . . . , **timbre_11_min**, and **timbre_11_max** = variables that indicate the minimum/maximum values over all segments for each of the twelve values in the timbre vector (resulting in 24 continuous variables)
- **Top10** = a binary variable indicating whether or not the song made it to the Top 10 of the Billboard Hot 100 Chart (1 if it was in the top 10, and 0 if it was not)

***

#### Problem 1.1 - Understanding the Data


Use the read.csv function to load the dataset "songs.csv" into R.

```r
songs <- read.csv("/Users/johnbobo/analytics_edge/data/songs.csv")
```

**How many observations (songs) are from the year 2010?**

```r
nrow(songs[songs$year==2010,])
```

```
## [1] 373
```
**Answer:** 373

***

#### Problem 1.2 - Understanding the Data


**How many songs does the dataset include for which the artist name is "Michael Jackson"?**

```r
mj <- subset(songs, artistname == "Michael Jackson")
nrow(mj)
```

```
## [1] 18
```
**Answer:** 18

***

#### Problem 1.3 - Understanding the Data


**Which songs by Michael Jackson made it to the Top 10?**

```r
subset(mj, Top10 == 1)$songtitle
```

```
## [1] You Rock My World You Are Not Alone Black or White    Remember the Time
## [5] In The Closet    
## 7141 Levels: ̈́ l'or_e des bois _\x84_ _\x84\x8d ... Zumbi
```
**Answer:** You Rock My World, You Are Not Alone, Black or White, Remember the Time, In The Closet.

***

#### Problem 1.4 - Understanding the Data


The variable corresponding to the estimated time signature (timesignature) is discrete, meaning that it only takes integer values (0, 1, 2, 3, . . . ). **What are the values of this variable that occur in our dataset?**

```r
unique(songs$timesignature)
```

```
## [1] 3 4 5 7 1 0
```
**Answer:** 3, 4, 5, 7, 1, 0

**Which timesignature value is the most frequent among songs in our dataset?**  

```r
sort(table(songs$timesignature), decreasing=TRUE)[1]
```

```
##    4 
## 6787
```

**Answer**: 4

***

#### Problem 1.5 - Understanding the Data


**What is a song with the highest tempo**

```r
songs$songtitle[which.max(songs$tempo)]
```

```
## [1] Wanna Be Startin' Somethin'
## 7141 Levels: ̈́ l'or_e des bois _\x84_ _\x84\x8d ... Zumbi
```
**Answer:** Wanna Be Startin' Somethin'

***

#### Problem 2.1 - Creating Our Prediction Model


We wish to predict whether or not a song will make it to the Top 10. To do this, first use the subset function to split the data into a training set "songsTrain" consisting of all the observations up to and including 2009 song releases, and a testing set "songsTest", consisting of the 2010 song releases.

```r
songsTrain <- subset(songs, year <= 2009)
songsTest <- subset(songs, year == 2010)
```

**How many observations (songs) are in the training set?**

```r
nrow(songsTrain)
```

```
## [1] 7201
```
**Answer:** 7201

***

#### Problem 2.2 - Creating our Prediction Model


In this problem, our outcome variable is "Top10" - we are trying to predict whether or not a song will make it to the Top 10 of the Billboard Hot 100 Chart. Since the outcome variable is binary, we will build a logistic regression model. We'll start by using all song attributes as our independent variables, which we'll call Model 1.

We will only use the variables in our dataset that describe the numerical attributes of the song in our logistic regression model. So we won't use the variables "year", "songtitle", "artistname", "songID" or "artistID".

```r
non_vars <- c("year", "songtitle", "artistname", "songID", "artistID")
songsTrain <- songsTrain[,!(names(songsTrain) %in% non_vars)]
songsTest <- songsTest[,!(names(songsTest) %in% non_vars)]
```
Now, use the glm function to build a logistic regression model to predict Top10 using all of the other variables as the independent variables. You should use SongsTrain to build the model.

```r
songs_log <- glm(Top10 ~ ., data=songsTrain, family=binomial)
summary(songs_log)
```

```
## 
## Call:
## glm(formula = Top10 ~ ., family = binomial, data = songsTrain)
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -1.922  -0.540  -0.346  -0.184   3.077  
## 
## Coefficients:
##                           Estimate Std. Error z value Pr(>|z|)    
## (Intercept)               1.47e+01   1.81e+00    8.14  4.0e-16 ***
## timesignature             1.26e-01   8.67e-02    1.46  0.14505    
## timesignature_confidence  7.45e-01   1.95e-01    3.81  0.00014 ***
## loudness                  3.00e-01   2.92e-02   10.28  < 2e-16 ***
## tempo                     3.63e-04   1.69e-03    0.21  0.82989    
## tempo_confidence          4.73e-01   1.42e-01    3.33  0.00087 ***
## key                       1.59e-02   1.04e-02    1.53  0.12635    
## key_confidence            3.09e-01   1.41e-01    2.19  0.02876 *  
## energy                   -1.50e+00   3.10e-01   -4.85  1.3e-06 ***
## pitch                    -4.49e+01   6.83e+00   -6.57  5.0e-11 ***
## timbre_0_min              2.32e-02   4.26e-03    5.44  5.3e-08 ***
## timbre_0_max             -3.31e-01   2.57e-02  -12.88  < 2e-16 ***
## timbre_1_min              5.88e-03   7.80e-04    7.54  4.6e-14 ***
## timbre_1_max             -2.45e-04   7.15e-04   -0.34  0.73209    
## timbre_2_min             -2.13e-03   1.13e-03   -1.89  0.05884 .  
## timbre_2_max              6.59e-04   9.07e-04    0.73  0.46757    
## timbre_3_min              6.92e-04   5.98e-04    1.16  0.24758    
## timbre_3_max             -2.97e-03   5.81e-04   -5.10  3.3e-07 ***
## timbre_4_min              1.04e-02   1.98e-03    5.24  1.6e-07 ***
## timbre_4_max              6.11e-03   1.55e-03    3.94  8.1e-05 ***
## timbre_5_min             -5.60e-03   1.28e-03   -4.38  1.2e-05 ***
## timbre_5_max              7.74e-05   7.94e-04    0.10  0.92234    
## timbre_6_min             -1.69e-02   2.26e-03   -7.45  9.7e-14 ***
## timbre_6_max              3.67e-03   2.19e-03    1.68  0.09388 .  
## timbre_7_min             -4.55e-03   1.78e-03   -2.55  0.01066 *  
## timbre_7_max             -3.77e-03   1.83e-03   -2.06  0.03941 *  
## timbre_8_min              3.91e-03   2.85e-03    1.37  0.17012    
## timbre_8_max              4.01e-03   3.00e-03    1.34  0.18162    
## timbre_9_min              1.37e-03   3.00e-03    0.46  0.64836    
## timbre_9_max              1.60e-03   2.43e-03    0.66  0.51019    
## timbre_10_min             4.13e-03   1.84e-03    2.24  0.02485 *  
## timbre_10_max             5.83e-03   1.77e-03    3.29  0.00099 ***
## timbre_11_min            -2.63e-02   3.69e-03   -7.11  1.2e-12 ***
## timbre_11_max             1.97e-02   3.39e-03    5.81  6.2e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 6017.5  on 7200  degrees of freedom
## Residual deviance: 4759.2  on 7167  degrees of freedom
## AIC: 4827
## 
## Number of Fisher Scoring iterations: 6
```

**Looking at the summary of your model, what is the value of the Akaike Information Criterion (AIC)?**  
**Answer:** 4827.1541

***

#### Problem 2.3 - Creating Our Prediction Model


Let's now think about the variables in our dataset related to the confidence of the time signature, key and tempo (timesignature_confidence, key_confidence, and tempo_confidence). Our model seems to indicate that these confidence variables are significant (rather than the variables timesignature, key and tempo themselves). **What does the model suggest?**  
**Answer:** The higher our confidence about time signature, key and tempo, the more likely the song is to be in the Top 10.

***

#### Problem 2.4 - Creating Our Prediction Model


In general, if the confidence is low for the time signature, tempo, and key, then the song is more likely to be complex. **What does our model suggest in terms of complexity?**  
**Answer:** Mainstream listeners tend to prefer less complex songs.

***

#### Problem 2.5 - Creating Our Prediction Model


Songs with heavier instrumentation tend to be louder (have higher values in the variable "loudness") and more energetic (have higher values in the variable "energy").

**By inspecting the coefficient of the variable "loudness", what does our model suggest?**  
**Answer:** The coefficient estimate for loudness is positive, implying mainstream listeners prefer louder songs, which are those with heavier instrumentation. However, the coefficient estimate for energy is negative, meaning that mainstream listeners prefer songs that are less energetic, which are those with light instrumentation. These coefficients lead us to different conclusions!

***

#### Problem 3.1 - Beware of Multicollinearity Issues!


**What is the correlation between the variables "loudness" and "energy" in the training set?**  

```r
cor(songsTrain$loudness, songsTrain$energy)
```

```
## [1] 0.7399
```
**Answer:** 0.7399 --- This means we have an issue with multicollinearity and should drop one of these variables in our model.  

***

#### Problem 3.2 - Beware of Multicollinearity Issues!


Create Model 2, which is Model 1 without the independent variable "loudness".

```r
songs_log_2 <- glm(Top10 ~ . -loudness, data=songsTrain, family=binomial)
summary(songs_log_2)
```

```
## 
## Call:
## glm(formula = Top10 ~ . - loudness, family = binomial, data = songsTrain)
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -2.098  -0.561  -0.360  -0.190   3.311  
## 
## Coefficients:
##                           Estimate Std. Error z value Pr(>|z|)    
## (Intercept)              -2.24e+00   7.46e-01   -3.00  0.00269 ** 
## timesignature             1.62e-01   8.73e-02    1.86  0.06287 .  
## timesignature_confidence  6.88e-01   1.92e-01    3.58  0.00035 ***
## tempo                     5.52e-04   1.67e-03    0.33  0.74023    
## tempo_confidence          5.50e-01   1.41e-01    3.91  9.4e-05 ***
## key                       1.74e-02   1.03e-02    1.70  0.08974 .  
## key_confidence            2.95e-01   1.39e-01    2.12  0.03416 *  
## energy                    1.81e-01   2.61e-01    0.70  0.48699    
## pitch                    -5.15e+01   6.86e+00   -7.51  5.9e-14 ***
## timbre_0_min              2.48e-02   4.24e-03    5.85  5.0e-09 ***
## timbre_0_max             -1.01e-01   1.18e-02   -8.55  < 2e-16 ***
## timbre_1_min              7.14e-03   7.71e-04    9.27  < 2e-16 ***
## timbre_1_max             -7.83e-04   7.06e-04   -1.11  0.26765    
## timbre_2_min             -1.58e-03   1.11e-03   -1.42  0.15453    
## timbre_2_max              3.89e-04   8.96e-04    0.43  0.66443    
## timbre_3_min              6.50e-04   5.95e-04    1.09  0.27452    
## timbre_3_max             -2.46e-03   5.67e-04   -4.34  1.4e-05 ***
## timbre_4_min              9.11e-03   1.95e-03    4.67  3.0e-06 ***
## timbre_4_max              6.31e-03   1.53e-03    4.12  3.9e-05 ***
## timbre_5_min             -5.64e-03   1.26e-03   -4.50  7.0e-06 ***
## timbre_5_max              6.94e-04   7.81e-04    0.89  0.37426    
## timbre_6_min             -1.61e-02   2.24e-03   -7.21  5.5e-13 ***
## timbre_6_max              3.81e-03   2.16e-03    1.77  0.07698 .  
## timbre_7_min             -5.10e-03   1.76e-03   -2.91  0.00364 ** 
## timbre_7_max             -3.16e-03   1.81e-03   -1.74  0.08109 .  
## timbre_8_min              4.49e-03   2.81e-03    1.60  0.11025    
## timbre_8_max              6.42e-03   2.95e-03    2.18  0.02950 *  
## timbre_9_min             -4.28e-04   2.96e-03   -0.14  0.88479    
## timbre_9_max              3.52e-03   2.38e-03    1.48  0.13802    
## timbre_10_min             2.99e-03   1.80e-03    1.66  0.09700 .  
## timbre_10_max             7.37e-03   1.73e-03    4.25  2.1e-05 ***
## timbre_11_min            -2.84e-02   3.63e-03   -7.82  5.5e-15 ***
## timbre_11_max             1.83e-02   3.34e-03    5.48  4.3e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 6017.5  on 7200  degrees of freedom
## Residual deviance: 4871.8  on 7168  degrees of freedom
## AIC: 4938
## 
## Number of Fisher Scoring iterations: 6
```
**Inspect the coefficient of the variable "energy". What do you observe?**  
**Answer:** The coefficient estimate for energy is positive implying mainstream listeners prefer heavier instrumentation. However, the variable energy is not significant in this model.

***

#### Problem 3.3 - Beware of Multicollinearity Issues!


Now, create Model 3, which should be exactly like Model 1, but without the variable "energy".

```r
songs_log_3 <- glm(Top10 ~ . -energy, data=songsTrain, family=binomial)
summary(songs_log_3)
```

```
## 
## Call:
## glm(formula = Top10 ~ . - energy, family = binomial, data = songsTrain)
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -1.918  -0.542  -0.348  -0.187   3.417  
## 
## Coefficients:
##                           Estimate Std. Error z value Pr(>|z|)    
## (Intercept)               1.20e+01   1.71e+00    6.98  3.0e-12 ***
## timesignature             1.15e-01   8.73e-02    1.32  0.18718    
## timesignature_confidence  7.14e-01   1.95e-01    3.67  0.00024 ***
## loudness                  2.31e-01   2.53e-02    9.12  < 2e-16 ***
## tempo                    -6.46e-04   1.67e-03   -0.39  0.69811    
## tempo_confidence          3.84e-01   1.40e-01    2.75  0.00602 ** 
## key                       1.65e-02   1.04e-02    1.59  0.11106    
## key_confidence            3.39e-01   1.41e-01    2.41  0.01598 *  
## pitch                    -5.33e+01   6.73e+00   -7.91  2.5e-15 ***
## timbre_0_min              2.20e-02   4.24e-03    5.20  2.0e-07 ***
## timbre_0_max             -3.10e-01   2.54e-02  -12.24  < 2e-16 ***
## timbre_1_min              5.42e-03   7.64e-04    7.09  1.4e-12 ***
## timbre_1_max             -5.12e-04   7.11e-04   -0.72  0.47193    
## timbre_2_min             -2.25e-03   1.12e-03   -2.01  0.04419 *  
## timbre_2_max              4.12e-04   9.02e-04    0.46  0.64791    
## timbre_3_min              3.18e-04   5.87e-04    0.54  0.58808    
## timbre_3_max             -2.96e-03   5.76e-04   -5.15  2.6e-07 ***
## timbre_4_min              1.10e-02   1.98e-03    5.58  2.3e-08 ***
## timbre_4_max              6.47e-03   1.54e-03    4.20  2.7e-05 ***
## timbre_5_min             -5.13e-03   1.27e-03   -4.05  5.2e-05 ***
## timbre_5_max              2.98e-04   7.86e-04    0.38  0.70453    
## timbre_6_min             -1.78e-02   2.25e-03   -7.94  1.9e-15 ***
## timbre_6_max              3.45e-03   2.18e-03    1.58  0.11420    
## timbre_7_min             -5.13e-03   1.77e-03   -2.90  0.00373 ** 
## timbre_7_max             -3.39e-03   1.82e-03   -1.86  0.06221 .  
## timbre_8_min              3.69e-03   2.83e-03    1.30  0.19323    
## timbre_8_max              4.66e-03   2.99e-03    1.56  0.11902    
## timbre_9_min             -9.32e-05   2.96e-03   -0.03  0.97486    
## timbre_9_max              1.34e-03   2.42e-03    0.55  0.57990    
## timbre_10_min             4.05e-03   1.83e-03    2.22  0.02664 *  
## timbre_10_max             5.79e-03   1.76e-03    3.29  0.00099 ***
## timbre_11_min            -2.64e-02   3.68e-03   -7.16  8.0e-13 ***
## timbre_11_max             1.98e-02   3.36e-03    5.90  3.7e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 6017.5  on 7200  degrees of freedom
## Residual deviance: 4782.7  on 7168  degrees of freedom
## AIC: 4849
## 
## Number of Fisher Scoring iterations: 6
```

Look at the summary of Model 3 and inspect the coefficient of the variable "loudness". Remembering that higher loudness and energy both occur in songs with heavier instrumentation, **do we make the same observation about the popularity of heavy instrumentation as we did with Model 2?**  
**Answer:** Yes because the coefficient estimate is positive for loudness. Additionally this variable is significant in Model 3.

***

#### Problem 4.1 - Validating Our Model


Make predictions on the test set using Model 3. What is the accuracy of Model 3 on the test set, using a threshold of 0.45? **(Compute the accuracy as a number between 0 and 1.)**

```r
predTest = predict(songs_log_3, newdata=songsTest, type="response")
table(songsTest$Top10, predTest >= 0.45)
```

```
##    
##     FALSE TRUE
##   0   309    5
##   1    40   19
```
**Answer:** 0.8794

***

#### Problem 4.2 - Validating Our Model


Let's check if there's any incremental benefit in using Model 3 instead of a baseline model. Given the difficulty of guessing which song is going to be a hit, an easier model would be to pick the most frequent outcome (a song is not a Top 10 hit) for all songs. What would the accuracy of the baseline model be on the test set? (Give your answer as a number between 0 and 1.)

```r
table(songsTest$Top10)
```

```
## 
##   0   1 
## 314  59
```
**Answer:** 0.8418

***

#### Problem 4.3 - Validating Our Model


It seems that Model 3 gives us a small improvement over the baseline model. Still, does it create an edge?

Let's view the two models from an investment perspective. A production company is interested in investing in songs that are highly likely to make it to the Top 10. The company's objective is to minimize its risk of financial losses attributed to investing in songs that end up unpopular.

A competitive edge can therefore be achieved if we can provide the production company a list of songs that are highly likely to end up in the Top 10. We note that the baseline model does not prove useful, as it simply does not label any song as a hit. Let us see what our model has to offer.

**How many songs does Model 3 correctly predict as Top 10 hits in 2010 (remember that all songs in 2010 went into our test set), using a threshold of 0.45?**    
**Answer:** 19

**How many non-hit songs does Model 3 predict will be Top 10 hits (again, looking at the test set), using a threshold of 0.45?**  
**Answer:** 5

***

#### Problem 4.4 - Validating Our Model


**What is the sensitivity of Model 3 on the test set, using a threshold of 0.45?**  
**Answer:** 0.322

**What is the specificity of Model 3 on the test set, using a threshold of 0.45?**  
**Answer:** 0.9841

***

#### Problem 4.5 - Validating Our Model


**What conclusions can you make about our model?**  
**Answer:** Model 3 favors specificity over sensitivity.  It gives us conservative predictions and rarely predicts a song will make it to the Top 10. How ever when it does predict a song being in the Top 10, we can be pretty confident it will make it to the Top 10.
