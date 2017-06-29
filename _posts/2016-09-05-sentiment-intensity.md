---
layout: post
title: Sentiment Intensity Analysis
cover: lion.jpg
date:   2016-09-05 01:00:00 -0800
categories: posts
---

This summer, I was lucky enough to participate as a fellow at University of Washington's [Data Science for Social Good fellowship](https://uwescience.github.io/DSSG2016). It was a really great, challenging, exciting experience, and I learned so much about Data Science best practices and how to perform advanced analyses on text data specifically. This is because the project I was assigned to was mining Amazon historical review data, joining it  to FDA product reviews, and developing a machine learning algorithm to predict food product recalls, hopefully more quickly than the current methods of research allow.

Due to the dirty nature of most text data, a ton of time is devoted to exploratory analysis from as many angles as possible to extract the most descriptive features. One feature I investigated was sentiment. Sentiment Analysis is a field in Text Mining that still requires a lot of research and is by no means perfected. Therefore, as a researcher this summer it only made sense to contribute to Sentiment Analysis methods. 

First I performed Sentiment Analysis on the text data using NLTK's [Sentiment Analysis](http://www.nltk.org/howto/sentiment.html) package. It has a built in Naive Bayes Classifier that integrates with the generated selected features, and I was able to test out how well that feature [could predict recalls](https://github.com/cvint13/DSSG2016-UnsafeFoods/blob/master/notebooks/text-mining-amazon-reviews.ipynb). It was not a great predictor, which did not surprise me. This is because we were able to determine a pretty strong case of colinearity between sentiment and rating, and we had already determined that rating was also not a good predictor. 

In order to test for colinearity between ratings and sentiment, we needed to extract a sentiment score, something quantifiable, to compare against the ratings. NLTK's Sentiment Analyzer package does not appear to have any sort of generated score for this type of use. However, as seen in the NLTK link above (3rd paragraph), there is a second option built in to the NLTK package- Sentiment Intensity Analyzer. 

This approach, formally known as [VADER](http://comp.social.gatech.edu/papers/icwsm14.vader.hutto.pdf) (Valence Aware Dictionary for sEntiment Reasoning), was developed by researchers at Georgia Tech, and according to their paper it outperforms human classifiers by over 10%! Even cooler about this method is the fact that it is simple- simpler than the more costly, sophisticated machine learning approaches. My main reservation, however, is the fact that it takes into account a 'gold-standard list of lexical features' developed from microblog contexts. This is a problem because the nature of microblogging platforms is that the language evolves extremely rapidly, just like SMS. This makes this type of text data averse to any sort of static 'gold-standard' set of features. The paper does discuss methods of 'learning' this gold standard lexicon, though.

That being said, it works great for now. It accounts for ALL CAPS, repeated characters for emphasisssssss, appearance and repetition of punctuation marks, and even emojis :). Is this ideal for Amazon reviews? Probably not,  because these reviews, as I see them, tend to be somewhere in between a microblog and a blog, maybe a centiblog or a milliblog. But because of this hybrid nature, neither sentiment analyzer is necessarily ideal.

The VADER analysis generates 4 measures for a single document:

1.  Compound- I think this is the overall sentiment score. It is neither mentioned in the NLTK documentation nor in the VADER publication, but in the code it is the normalized sum of the other three scores.
2.  Positive- how positive the document is, normalized.
3.  Negative- how negative the document is- normalized.
4.  Neutral- how neutral it is, normalized.

By normalized, I mean all scores are adjusted to be between -1 and 1. For further research of this feature, I could create a Document-sentiment matrix and determine if that is more successfully predictive than the rating. You could potentially not include the compound score, since the other three scores from which it is calculated are included and therefore it is essentially already included. At the end of the day, however, different types of different dissatisfaction lead to different types and levels of distress in the reviews. Also, it doesn't appear that words that are most indicative of a necessity for recall necessarily lower the sentiment according to existing sentiment analysis packages. We look forward to researching other features that will possibly have more success in improving our model, but in the meantime VADER was quite a gem to come across for anyone interested in Twitter data.