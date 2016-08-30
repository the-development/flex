---
layout: post
title: My First Pass at a Resume Matcher
cover: books.jpg
date:   2016-08-29 01:00:00 -0800
categories: posts
---

I was recently approached to develop a 'Semantic Search' tool, in which you could input a list of resumes and a job description, and the resumes would be ranked by relevance. I immediately answered, 'Sure, no problem!' This seemed like such a simple task, because rather than a typical search engine that takes in a single phrase and has to make huge assumptions about what the user is actually looking for, this program would take in a lengthy, concise 'search query' of sorts that is full of key words essential to a successful resume. 

I explained to my colleague the basics of matching texts, quantifying term frequencies, how you calculate distances between documents, etc., and he responded, 'But do you know about anything that can capture the *meaning* of a sentence, more of a *semantic search*? I hadn't heard this phrase before, and I was intrigued. After looking into it, it turns out that there are entire companies out there that provide this Semantic Search software to HR departments, and it is becoming an essential part of finding good candidates.

There are a lot of companies out there now providing a matching tool, in which you input a job description and it returns and ranks matches from your database of candidate resumes. Companies like Monster are now offering 'Semantic Search' as a service, supposedly using Artificial Intelligence to make the best matches on your job descriptions. I found a great [article](http://booleanblackbelt.com/2012/01/semantic-search-explained-for-sourcing-and-recruiting/) on what 'Semantic Search' really means, and if it's really worth your money.

Below are the major claims about these types of services:

1. They go beyond exact matches. This includes:
	- Fuzzy String Matching
	- Synonyms and related terms
	- Contextual Analysis
2. Machine Learning and Pattern Matching
	- Clustering of Resumes

This all sounds really awesome. However, I am not sure that it provides a huge payoff. First of all, all of these methods you see above are already prepackaged and ready to use in a package in just about every respectable programming language out there. If you assigned the task of developing this tool to a Summer intern, you already got your payoff. Second of all, Machine Learning methods in the arena of text data are hard for all of the reasons that you don't find in a corpus of clean, edited, professional, non-flowery documents. Job descriptions and resumes don't have sarcasm or emojis. And you can pretty well guarantee that if you cluster your resumes, they will probably nicely align with the corresponding positions that the candidates are applying for. It's nice to have a repository of synonyms, but I've certainly noticed that job descriptions call for very specific technical skills and background. If you want a Python developer, you want to see 'Python' in their resume. As for the few situations where this isn't the case, it might not be worth a monthly service fee to push the rank of that occasional, unlikely applicant by 2 slots.

Nevertheless, I was investigating this topic to see about this newly found project, and decided to look into a simple method for how it could be done. Who needs to pay for a monthly service if they can write up a quick program? I wouldn't recommend a cheap solution like this for a large company with thousands of resumes to sift through, but you don't need anything super advanced for a small corpus. 

# Exploring Data and Methods

This notebook consists of an exploratory analysis of expected data to be input into the final resume search system. While I did not include code for fuzzy string matching, synonyms and taxonomies, and contextual analysis beyond bigrams, I still believe that a toll this simple will do 80-90% of the sifting through resumes for you. The rest can be added within another week.

## Reading in the Data

For this project, we will read the resume text and metadata into a Pandas dataframe. See [the pandas website](http://pandas.pydata.org/) for more information.


```python
import pandas as pd

#read in the excel file
resume_data = pd.read_excel('../test-files/test_batch.xls')
resume_data
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>First Name</th>
      <th>Last Name</th>
      <th>Email</th>
      <th>Resume</th>
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>Postal Code</th>
      <th>Date Joined</th>
      <th>Interest Level:</th>
      <th>Desired Job Title:</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>George</td>
      <td>Doe</td>
      <td>GD@Yahoo.com</td>
      <td>George Doe\n1001 N. 1st st  Henderson, NV. 890...</td>
      <td>Henderson</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 14:20:21</td>
      <td>NaN</td>
      <td>Sr. Systems Administrator</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Ellen</td>
      <td>Does</td>
      <td>ED@gmail.com</td>
      <td>Ellen Does\n\n3221 15th Ave. Las Vegas, NV 891...</td>
      <td>Las Vegas</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 13:33:14</td>
      <td>NaN</td>
      <td>Engineering Assistant</td>
    </tr>
    <tr>
      <th>2</th>
      <td>David</td>
      <td>Test</td>
      <td>DTEST@gmail.com</td>
      <td>DAVID TEST\n210 W. Elm Street, Henderson, NV 8...</td>
      <td>Henderson</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 12:44:40</td>
      <td>NaN</td>
      <td>Account Manager</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jane</td>
      <td>Doe</td>
      <td>JaneDoe@yahoo.com</td>
      <td>Jane Doe\n1122 E. Elm, North Las Vegas, NV 890...</td>
      <td>North Las Vegas</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 09:47:19</td>
      <td>NaN</td>
      <td>Payment Operations</td>
    </tr>
  </tbody>
</table>
</div>



## Read in the Job Description

This job decription is a word document, so we will use the python-docx package in Python to read in the text. it will then be appended as the last document in the vector of documents.


```python
import docx

#method to fetch text from all paragraphs in the job description
def getText(filename):
    doc = docx.Document(filename)
    fullText = []
    for para in doc.paragraphs:
        fullText.append(para.text)
    return '\n'.join(fullText)

#run method on path to description file
description = getText('../test-files/TEST_JD.docx')

#get just resume text
text_data = list(resume_data.Resume)
text_data.append(description)
```

## Processing the Text

In this analysis we will utilize the [NLTK](nltk.org) package in python to process the text, including stemming, removal of stop words and numbers, and creation of the Document Term Matrix.


```python
from nltk import word_tokenize
from nltk.corpus import stopwords
from nltk.stem.lancaster import LancasterStemmer
import re

#get stopwords
stops = stopwords.words('english')

#get stemmer
st = LancasterStemmer()

#remove numbers
for i in range(len(text_data)):
    text_data[i] = re.sub('\d','',text_data[i])

#tokenize each resume
resume_tokens = [word_tokenize(text) for text in text_data]

#make it all lowercase
for i in range(len(resume_tokens)):
    
    tokens = resume_tokens[i]
    
    #make lowercase
    tokens = [token.lower() for token in tokens]
    
    #remove stopwords/skipwords
    tokens = [token for token in tokens if token not in stops]
    
    #stem tokens
    tokens = [st.stem(token) for token in tokens]
    
    resume_tokens[i] = tokens
```




    5



After we have preprocessed the text, we want to merge it back together in preparation for creation of the Count Vectorizer Matrix, which in this case is our term document matrix. We have to do this because the sklearn package's Count Vectorizer Matrix creation method requires this format.


```python
##Manipulate stemmed text to be string instead of list (needed for count vectorizer)
final_resume_text = []
for text in resume_tokens:
    string = ''
    
    for word in text:
        string += ' ' +word
        
    final_resume_text.append(string)
```

## Create the Document Term Matrix

We will use numpy, sklearn and SciPy to create the Document Term Matrix. It will account for unigrams and bigrams- that is, single word occruences and occurences of pairs of words.


```python
##Count Vectorizer Matrix
import numpy as np
import scipy
from scipy.sparse import coo_matrix, vstack
from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer(binary=False, ngram_range=(1, 2)) ##Removed stopwords before stemming so don't apply here

resume_text = vectorizer.fit_transform(final_resume_text)

counts = scipy.sparse.coo_matrix.sum(resume_text, axis=0)
resume_text = np.transpose(vstack([resume_text]))
resume_text = pd.DataFrame(resume_text.todense(), index = vectorizer.get_feature_names())
```


```python
resume_text
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>abl</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>abl driv</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>abl us</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>absolv</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>absolv wel</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>academ</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>academ educ</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>acc</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>acc report</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>acceiv</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>acceiv rat</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>access</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>access control</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>access stor</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>accord</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>accord fdic</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account</th>
      <td>1</td>
      <td>0</td>
      <td>12</td>
      <td>11</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account appl</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account attorney</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account credit</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account day</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account execut</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account front</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>account man</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
  </tbody>
</table>
<p>2611 rows × 5 columns</p>
</div>



## Find Distance Between Resumes and Job Description

For this exercise, we will calculate the Euclidean distance between the n-gram frequency vectors for each resume and the job description. We will then rank them from lowest to highest score using this distance measure.


```python
#create list to store distances
distances = []

#get vector representing job description
description = resume_text.iloc[:,resume_text.shape[1]-1]

#iterate through the columns, since each column represents a single document
for col in range(resume_text.shape[1]-1):
    
    #get resume vector
    resume = resume_text.iloc[:,col]
    
    #calculate distance
    dist = np.linalg.norm(description-resume)
    
    #add to list of distances
    distances.append(dist)
```


```python
distances
```




    [155.74979935781619,
     36.715119501371639,
     43.588989435406738,
     39.899874686520008]



## Alternative Measure- Limit Terms

We can also limit the measurement to only terms pertinent to the job description. This way, the distance won't be thrown off by all of the extra terms in the resumes that might show a ton of relevant experience but aren't necessarily mentioned in the job description.


```python
#subset only job-description-specific terms
description_terms = resume_text.iloc[[index for index in range(resume_text.shape[0]) \
                                     if resume_text.iloc[index,resume_text.shape[1]-1] > 0],:]
```


```python
description_terms
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>abl</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>abl driv</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>across</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>across property</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>act</th>
      <td>17</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>act job</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adequ</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adequ meas</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adv</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adv excel</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>also</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>also requir</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys</th>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>7</td>
    </tr>
    <tr>
      <th>analys across</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys adv</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys in</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys inform</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys lik</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys skil</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys team</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
  </tbody>
</table>
<p>323 rows × 5 columns</p>
</div>



## Change Term Frequency to Term Existence

If a job description calls for a business analyst, and only uses the term 'business analyst' once, while a very relevant resume uses the term 'business analyst' 5 times throughout their job history and summary, this will cause the distance to greatly increase. We want to fix this. One way is to make each cell either a 1 or a 0- that is, either it contains the term or it doesn't.


```python
for row in range(description_terms.shape[0]):
    for col in range(description_terms.shape[1]):
        if description_terms.iloc[row,col] > 0:
            description_terms.iloc[row,col] = 1
```

    C:\Users\cvint\Anaconda3\lib\site-packages\pandas\core\indexing.py:132: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      self._setitem_with_indexer(indexer, value)
    C:\Users\cvint\Anaconda3\lib\site-packages\ipykernel\__main__.py:4: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    


```python
description_terms
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>abl</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>abl driv</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>across</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>across property</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>act</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>act job</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adequ</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adequ meas</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adv</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>adv excel</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>also</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>also requir</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys</th>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys across</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys adv</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys in</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys inform</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys lik</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys skil</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analys team</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>analyst</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
  </tbody>
</table>
<p>323 rows × 5 columns</p>
</div>



Now we can run the distance measure function with more confidence.


```python
#create list to store distances
distances = []

#get vector representing job description
description = description_terms.iloc[:,description_terms.shape[1]-1]

#iterate through the columns, since each column represents a single document
for col in range(description_terms.shape[1]-1):
    
    #get resume vector
    resume = description_terms.iloc[:,col]
    
    #calculate distance
    dist = np.linalg.norm(description-resume)
    
    #add to list of distances
    distances.append(dist)
```


```python
distances
```




    [16.733200530681511,
     17.146428199482248,
     17.233687939614086,
     16.941074346097416]



Now we can append this distance measure to the dataframe.


```python
resume_data.distances = distances
```

## Sort the Resumes

Now we want to output a sorted list of the resumes in order of how close they are to the job description. The lower the Euclidean distance, then in theory the more similar they are to the job description.


```python
resume_data['distances'] = distances
```


```python
resume_data = resume_data.sort('distances',ascending = True)
```

    C:\Users\cvint\Anaconda3\lib\site-packages\ipykernel\__main__.py:1: FutureWarning: sort(columns=....) is deprecated, use sort_values(by=.....)
      if __name__ == '__main__':
    


```python
resume_data
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>First Name</th>
      <th>Last Name</th>
      <th>Email</th>
      <th>Resume</th>
      <th>City</th>
      <th>State</th>
      <th>Country</th>
      <th>Postal Code</th>
      <th>Date Joined</th>
      <th>Interest Level:</th>
      <th>Desired Job Title:</th>
      <th>distances</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>George</td>
      <td>Doe</td>
      <td>GD@Yahoo.com</td>
      <td>George Doe\n1001 N. 1st st  Henderson, NV. 890...</td>
      <td>Henderson</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 14:20:21</td>
      <td>NaN</td>
      <td>Sr. Systems Administrator</td>
      <td>16.733201</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jane</td>
      <td>Doe</td>
      <td>JaneDoe@yahoo.com</td>
      <td>Jane Doe\n1122 E. Elm, North Las Vegas, NV 890...</td>
      <td>North Las Vegas</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 09:47:19</td>
      <td>NaN</td>
      <td>Payment Operations</td>
      <td>16.941074</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Ellen</td>
      <td>Does</td>
      <td>ED@gmail.com</td>
      <td>Ellen Does\n\n3221 15th Ave. Las Vegas, NV 891...</td>
      <td>Las Vegas</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 13:33:14</td>
      <td>NaN</td>
      <td>Engineering Assistant</td>
      <td>17.146428</td>
    </tr>
    <tr>
      <th>2</th>
      <td>David</td>
      <td>Test</td>
      <td>DTEST@gmail.com</td>
      <td>DAVID TEST\n210 W. Elm Street, Henderson, NV 8...</td>
      <td>Henderson</td>
      <td>Nevada</td>
      <td>US</td>
      <td>NaN</td>
      <td>2016-08-16 12:44:40</td>
      <td>NaN</td>
      <td>Account Manager</td>
      <td>17.233688</td>
    </tr>
  </tbody>
</table>
</div>



## Further Exploration

As text mining and NLP are always a work in progress and methods can always be improved, this is just a first pass at a complex topic. A typical method to include is TF-IDF weighting, which adds more weigh to terms that appear frequently in few documents. Since this is an extremely small corpus and the vocabulary was very short and concise, it might make more sense to test this method on a larger test corpus. Moreover, the 1-0 method might make the most sense when you just want to identify whether or not a resume conains the term 'Python' or 'Financial Analyst'.

Another option to potentially improve results is to attempt other distance measures or implement other features into the ranking. For a larger corpus we could first cluster the documents and implement the cluster assignments and a second-tier feature in forms of similarity to the job description. We could also add more weight to simply the desired job title, so that the hiring manager isn't wasting their time with candidates who don't even want the job that the hiring manager is seeking candidates for.
