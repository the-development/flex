---
layout: post
title: The Trouble with Stemming in Arabic
cover: morocco-palace.jpg
date:   2016-10-11 01:00:00 -0800
categories: posts
---

Arabic is a really cool language. It is Semitic, which means that it is very closely related to Hebrew, and it consists of three-letter root words. Let me give you an example:

The three-letter root

X--X3-Y or ha-sa-na

is a very common root that is used to mean many related terms:

X-X3Y ('hasan') means 'good' or 'perfect'.

X'X-X3X'Y ('ihsaan') is a spiritual term referring to 'beneficence' or 'charity', although I have also heard it indicate something along the lines of self-betterment.

X-X3YX) ('hasna') is a 'good deed'.

X'X3X*X-X3X'Y ('istihsaan') means 'approval' or 'consent'.

So as you can see, there is a common theme based around the root to indicate some form of positivity. On the surface, this seems potentially very convenient for Computational Linguistics or Natural Language Processing. Unfortunately, the devil is in the details with Arabic Language Processing and Understanding.

In text mining, you want to be able to 'stem' your words. In English this means you cut off suffixes like '-ing' or '-es' and get the barest form possible. For example, 'challenging' would stem to 'challeng', so it can then match both the noun and verb forms as well as the different tenses of the word 'challenge'. Then your term frequencies, etc., can be based on this simplified root and parallels in texts can be more easily identified. 

In Arabic stemming, generally the rule is to narrow each term down to its 3-letter root as best as you can. So 'istihsaan' and 'ihsaan' would both be reduced to ha-sa-na, assuming the stemming algorithm worked. Ok, great, so if both of these terms show up we can see a pattern of positivity. But wait, what if you are trying to classify the topic of the text? What, really, does approval or consent have to do with charity? They could be related, but is it all that safe to lump them together so bluntly like that? In English, the stems 'consent' and 'charit' would not automatically be related unless they appeared in the same documents, so why should they be squashed together in Arabic? This is a problem for sure, and I find most Arabic stemmers (At least open-source ones) to be really sub par in my limited experience.

Moreover, this example above is a nice one. The roots get more and more diverse in their allowable meanings, to the point where it is ridiculously frustrating for an Arabic language learner. Let me give you an example that I was able to find by literally randomly selecting a page in my Arabic dictionary:

X7-X1-X- or ta-ra-ha (bear in mind these are really crass transliterations.)

X7X1X- ('tarh') means expulsion, banishment.

X7X1X-X) ('tarha') is a veil worn by Arab women as a headcloth.

X'X7X1YX-X) ('utruuha') is a disseration.

I think you can catch my drift.

One solution that has been examined is the concept of _light stemming_. This means that just the basic peices at the beginning and end of an Arabic word that have an extremely high likelihood of being an unnecessary extension are removed, such as the definite article and the plural endings. In my opinion, this is potentially a very viable solution. I will provide below, however, an alternate solution that I believe is worth studying as well.

So we have already developed methodologies to identify the roots in Arabic, but there is a second component of a word: the form. There are 10 verb forms of each three-letter root, so a word has a form, a root, and a tense. The form can likely be identified with reasonable accuracy, and so the root plus the form can be the two features for each term. This added dimension would potentially complicate your algorithms, but it would also massively improve your results and predictive capabilities (I think). [This](http://arabic.desert-sky.net/g_vforms.html) is a great introduction to the different verb forms and what they *likely* indicate about the word's meaning. Even if the indication is off sometimes, from a mathematical perspective this would rarely matter. The root + form data for each term will separate out term frequencies for the different words, and this will in turn affect context analyses, etc. for your different types of analysis. The numbers will, in short, be better.

I would love to go ahead and work on this topic further on my own. I believe a lot of contributions have yet to be made to the field of Arabic Text Mining, and I hope to be a contributor to the topic.