# newsfeed-nlp

## Abstract 
This unsupervised learning project allows the average news consumer to experience a stream-lined information acquisition process, free of repetition. Working in Python and using a Kaggle dataset (https://www.kaggle.com/snapcrack/all-the-news) of 85,000 news articles, we extract significance from the texts by utilizing a modified TFIDF-Vectorizer to pre-process the data. We experiment with various clustering techniques (Kmeans, HAC, and KNN), paired with various success metrics to gauge effectiveness of the news consolidation. Visualizations are created using Seaborn and Matplotlib, along with D3 for ease of exploration. Spacy is used as the primary NLP.

## Motivation
In a political climate of intensely polarizing takes on the latest scandals, international relations, and national issues, it can be difficult to make sense of all the data. With the rise of social media, the ability to publish has been democratized and repetition in the newsfeed runs rampant. By grouping together news stories by topic, not only is the newsfeed browsing process streamlined, but clusters are formed that provide differing perspective on the same story.

# Methodology - Data Cleaning
Below is a snippet of the original data (https://www.kaggle.com/snapcrack/all-the-news), with the 'content' column omitted. 

|        |                                                                                   |             |                   |          |      |       |                                                                                                                         | 
|--------|-----------------------------------------------------------------------------------|-------------|-------------------|----------|------|-------|-------------------------------------------------------------------------------------------------------------------------| 
| id     | title                                                                             | publication | author           | date     | year | month | url       | content                                                                                                              | 
| 151908 | Alton Sterling‚Äôs son: ‚ÄôEveryone needs to protest the right way, with peace‚Äô | Guardian    | Jessica Glenza    | 7/13/16  | 2016 | 7     | https://www.theguardian.com/us-news/2016/jul/13/alton-sterling-son-cameron-protesters-baton-rouge                       | 
| 151909 | Shakespeare‚Äôs first four folios sell at auction for almost ¬£2.5m               | Guardian    |                   | 5/25/16  | 2016 | 5     | https://www.theguardian.com/culture/2016/may/25/shakespeares-first-four-folios-sell-at-auction-for-almost-25m           | 
| 151910 | My grandmother‚Äôs death saved me from a life of debt                             | Guardian    | Robert Pendry     | 10/31/16 | 2016 | 10    | https://www.theguardian.com/commentisfree/2016/oct/31/grandmothers-death-saved-me-life-of-debt                          | 
| 151911 | I feared my life lacked meaning. Cancer pushed me to find some                    | Guardian    | Bradford Frost    | 11/26/16 | 2016 | 11    | https://www.theguardian.com/commentisfree/2016/nov/26/cancer-diagnosis-existential-life-accomplishments-meaning         | 
| 151912 | Texas man serving life sentence innocent of double murder, judge says             | Guardian    |                   | 8/20/16  | 2016 | 8     | https://www.theguardian.com/us-news/2016/aug/20/texas-life-sentence-innocence-dna-richard-bryan-kussmaul                | 
| 

The orginal dataset was cleansed to only keep rows which contained both a url and a date. This data cleansing resulted in a working dataframe of 82,920 articles.

# Methodology - Analysis

## Data Exploration
We began by exploring the distribution of certain notable factors of the data, such as date and publisher. There are 10 unique pulishers in our subset of the data, with an interesting distribution in publishing date:

<img src="https://github.com/parkervg/news-article-clustering/blob/master/Visualizations/date_distribution.png" width="1000">


For the purpose of our project, a document similarity-based task, it seemed most appropriate to run our clustering tests on a section of the data found to the right. Considering that the texts we are working with are all news stories, the frequency of having multiple distinct news stories published about the same event is increased when they are more chronologically compact, i.e. the chance of having an article about the 2016 Iowa Caucus 7 months after it took place is unlikely. 

We began by labelling each datapoint with a succinct event label to accurately describe the event it portrayed. One such example is found below (with abbreviated content), which yielded the label "daallo_airlines_explosion."

|  event                         |  |   content                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | date      | id      | month  |  publication                   |           month                                                  |                  url                                                                                                               | year     | 
|---------------------------|--|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|-------|---|---------------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|------| 
| daallo_airlines_explosion |  | MOGADISHU, Somalia (AP) ‚Äî An explosion that blew a hole in a jetliner shortly after takeoff and left one man missing was believed to have been caused by a bomb, the pilot said Wednesday, describing how the crew calmed frightened passengers as smoke enveloped the cabin before he brought the plane back to Mogadishu‚Äôs airport for an emergency landing. Daallo Airlines said all passengers except one got off the plane safely. It previously... | 2/3/16 | 95647 | 2 | Talking Points Memo | Flight Lands Safely After Suspected Bomb Blew Hole In Plane | https://web.archive.org/web/20160204014156/http://talkingpointsmemo.com/world-news/daallo-airlines-explosion-plane-lands-safely | 2016 | 


This labelling was completed for 1,000 of the articles, to be used for our clustering and success metrics. 


## Pre-Processing
TFIDF is used as the primary pre-processing method, with some adjustments to account for entity weighting. First, a special tokenizer was created which takes into consideration the entity type of the token. If the token is not an entity, the stem of the token is taken using nltk's [snowball stemmer](http://www.nltk.org/howto/stem.html). Stemming refers to the process of reducing an inflected (or sometimes derived) word to their base form, even if is not identical to its morphological root. If the token is a location, organization or event, the token is saved in all capital letters. If the token is a person, it is saved as a title (only the first letter of each word is capitalized). This was achieved using [Spacy's entity recognition](https://spacy.io/usage/linguistic-features).

```python
sent = ["The United Nations ban pineapple on pizza, but Bill Gates intends to fights back."]
tokens = tokenize_and_stem_NER(sent)
print(tokens)
>>>> ['THE UNITED NATIONS', 'ban', 'pineappl', 'pizza', 'Bill Gates', 'intend', 'fight']
```

Then, to give extra significance to these entities, their TFIDF score was manipulated; specifically, in creating the TF_dict from the file [tfidf_from_scratch_with_entities.py](https://github.com/parkervg/news-article-clustering/blob/master/Pre-Processing/tfidf_from_scratch_with_entities), the non-person entities were multipled by a factor of 4, and the person entities a factor of 1.3. 

## Clustering
As noted before, the three clustering methods that we utilized in our project were Kmeans, HAC, and KNN. Ultimately, after many iterations and logging of [success rates](https://github.com/parkervg/news-article-clustering/blob/master/Success_Rates.md), [HAC (Hierarchical Agglomerative Clustering)](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html) proved to be the most successful method.

Below is a graphic, 2D representation of the multidimensional TFIDF matrix and resulting clusters (as clustered by Kmeans). The annotation by each data point is the "predominant event" as explained below in the Defining Success section, and the size of each point is directly related to the number of articles in that specific cluster. 

![Article Centers](/Visualizations/Article_Centers.png)


## Defining Success
### [F1 Score](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)
The F1 score is defined as the harmonic mean between precision and recall. Being as this is an unsupervised learning project, the definition of "y_true" was not inherently obvious. When looking at a cluster of, say, {'zika_std': 1, 'robert_finicum_shooting': 3}, it is difficult from the mere labels which event is a false positive. In response to this ambiguity, a success algorithm was made to assign as many clusters as possible to one unique, predominant event. 

This predominant event was found by invoking not only the frequency of events, but the individual events' "ratio" when compared to the total occurences of that event. First, the predominant event was defined merely as that event which occured most often within a specific cluster. For example, in the instance of *Cluster A* containing {'zika_std': 1, 'robert_finicum_shooting': 3}, the predominant event would be 'robert_finicum_shooting.'

Second, the ratio of an event is invoked. The process of defining predominance merely by quantity of events results in doubling-up of events; multiple clusters are assigned the same event. If *Cluster B* yields {'santorum_drops_out': 1, 'robert_finicum_shooting': 2}, it would also recieve 'robert_finicum_shooting' as the predominant label. But perhaps the label "santorum_drops_out" is used only 4 times across the whole dataset, while the label "robert_finicum_shooting" was used a total of 10 times. To compare the ratio of santorum_drops_out (1/4 = .25) and robert_finicum_shooting (2/10 = .2), the label santorum_drops_out would be more significant in this particular cluster. 

If, between two clusters there is a tie in ratios of their predominant event, the clusters are dismissed when calculating the F1 Score. For our purposes, the F1 Score is not intended to be a perfect measure, but merely a gauge by which some improvement may be noticed as we pass through different clustering models.

### [Silhouette Score](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.silhouette_score.html)
While the F1 Score defines success based on the values being assigned, the Silhouette Score uses the intrinsic properties of the clusters themselves, with no weight given to the meaning of labels. 

A value of +1 indicates that the sample is far away from its neighboring cluster and very close to the cluster its assigned. Similarly, value of -1 indicates that the point is close to its neighboring cluster than to the cluster its assigned. And, a value of 0 means its at the boundary of the distance between the two cluster. Value of +1 is ideal and -1 is least preferred. Hence, higher the value better is the cluster configuration.

## Summary
Our 500 cluster HAC model can cluster together the 1,000 news articles we had pre-processed with an *F1-Score of 0.922* and a *Sillhouette Score of 0.093*. 


Possible Roadblocks:
* Specificity in clusters. Russia = Russian election hacking or Russian international relations or Russian Olympic ban?
* Intersections in broad topics. E.g. "Donald Trump speaks about Hurricane Matthew" about DT or hurricane?
Notes:
* Put special emphasis on dates and names when organizing clusters (use spacy entity recognition) 



