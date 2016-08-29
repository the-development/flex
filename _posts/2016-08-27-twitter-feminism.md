---
layout: post
title: Feminism Attitude Analysis of Twitter Data
cover: maroc.jpg
date:   2016-08-24 04:00:00 -0800
categories: posts
---

As a freelancer, I come across fun, small projects that are a great way to build my skill set, make a little money, and have some fun work that doesn't feel as drab as 40 hours a week of the same stuff. This past Spring, I got to collaborate with a graduate student at American University on a small project that involved analysis of Twitter data, specifically tweets within a 6-week period from Tanzania. 

What we wanted to do was determine if there was a way to identify attitudes toward women with a machine learning model. This was not an original idea; see [this article](http://www.prooffreader.com/2015/05/most-characteristic-words-in-pro-and.html) to learn about a crowd-sourced approach to attitude analysis with Twitter data. As this was fairly early in my machine learning education, my methods were not too advanced, but it was a really good introduction to many of the problems that we often come across in Text Mining.

Since this was a 2-week project, we had to go fast. Fortunately there was enough data to be a bit quick and dirty. The first thing was separating the data into Swahili and English tweets (other languages would be disregarded). The project lead downloaded the tweets with a third client, and unfortunately language metadata was not available. So we ended up using R's [textcat](https://cran.r-project.org/web/packages/textcat/textcat.pdf) package for language identification. 

Since other similar experiments implemented some sort of crowd-sourcing to tag tweets as pro- or anti-feminist, we decided to do this for our training data set. We had a Swahili speaker available part time to tag the Swahili tweets, and I split up a chunk of the English ones with the project lead. This was tedious, and also it turns out that this is a very subjective tag. Pretty much all of the Swahili tweets were tagged as positive, even though Google Translate on some of the simpler ones indicated otherwise.In a larger scale, more scientific project, I would outline clear guidelines for this sort of task, so that we can maintain the highest level of consistency.

The next hurdle was the disparity in number of pro- and anti-feminist tweets. I tested two methods: two tags (pro, anti) and 3 tags (pro, anti, neutral). The second method was an improvement, but not amazing. I tried a first pass with KNN classification, and it had around a 75% success rate. This was not bad, but there was definitely more work to be done. With more time, I would have tried topic modeling, and attempted to find themes that indicate anti-women attitudes. This or other types of feature selection might have been helpful. I also would have attempted multiple types of machine learning models.

The final task was visualizing the data. I created a measure- feminist score- to record on a heat map of Tanzania. I calculated this score by assigning -1 to anti- tweets, +1 to pro- tweets, 0 to neutral tweets, and calculating the mean within a region (we did fortunately have the province in the metadata). It was actually pretty hard to find a Tanzania map that included regions. I managed to find one via [UC Berkeley](http://gif.berkeley.edu/resources/data_subject.html), and from there I was able to map the data. 

![English Feminism Scores by Region]({{ site.url }}{{ site.baseurl }}/images/english-map.png)

### Lessons Learned

1. Text Mining takes time.
2. If you have time, explore the data as much as you can before you even think about classification. This means finding features in the data, researching for patterns in the metadata, etc.
3. Don't remove skip words in Twitter data. All Words Matter in this sort of Social Media.
4. Attitudes towards women are not cut and dry enough to just trust anyone to tag text. This issue surely exists in all sorts of attitude and sentiment analysis.
5. Sentiment Analysis is (in my opinion) a lot easier than attitude analysis.
6. Sarcasm sucks.