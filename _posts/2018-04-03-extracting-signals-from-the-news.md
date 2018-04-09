---
layout:       post
uuid:         a753cd00-1906-0136-63ff-186590cb4787
categories:   machine-learning
tags:         [machine-learning]
title:        'Extracting Signals From the News'
date:         2018-04-03
author:
  name:       David Newcomb
  twitter:    null
  github:     null
feature_img:  null
sitemap:
  lastmod:    2018-04-03T08:48:31
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

At the core of DataFox is a focus on creating a pristine dataset of company information. We want this data to be as clean, accurate and all-encompassing as possible.

In practical terms, this means we want to gather as much relevant information as possible on every company in our database. In one workflow, we contribute toward this goal through data partnerships; we combine, prioritize and resolve a huge variety of data sources.

In another workflow, we focus on the data that partnerships cannot provide. We make a meticulous effort to find relevant and unique company signals from unstructured text. It's an enormous effort, but well-worth the creation of a proprietary, competitively advantageous dataset.

News Sources
------------

Although we also classify unstructured text coming from public websites, company blogs, and tweets, this post will focus solely on news generated from news organizations.

Every 30 minutes of every day, we collect the articles from nearly 7000 of these sources (through RSS feeds) and pass them through our classifier. These sources range from highly specific news sources (e.g. Healthcare IT News) to more widely digested websites (e.g. TechCrunch).

What they all have in common is a high degree of content curation (thank you editors) and a focus on being very up-to-date (thank you capitalism). These features help prevent garbage from entering our pipeline and helps us keep our classification of the news highly targeted and highly relevant.

News Processing: Problem Statement
----------------------------------

Given the constant pipeline of articles being created by these sources, we want to make sense of these articles and extract important company-related signals from them. For as many companies as possible, we want to automatically tag articles with their appropriate signals.

In our domain, we currently classify 68 such signal types that broadly fall into these eight buckets:
* Signal Types
    * Growth
    * Financial
    * People
    * Recognition
    * Events & Marketing
    * Corporate Update
    * Negative News

Categorization of these signal types can be difficult for many reasons:
* Customers expect high precision
    * Consumer trust is built on providing accurate signals

* Customers expect high recall
    * The more signals we can classify the greater the edge our customers have and customers expects us to not miss any events

* The difference between signal types can be subtle
    * For example, we distinguish between a company relocating offices and expanding offices (which shows growth).

* Attribution of a signal to a company can be tricky
    * For example, Google and Facebook are mentioned in a very large number of articles, but only a small fraction are actually focused on these companies.


Classification Approach
-----------------------

We could have approached this as a [multi-label classification](https://en.wikipedia.org/wiki/Multi-label_classification) problem at the article level. In this situation, an article could contain information about a company that both received private funding and expanded to open new offices.

We would then tag the article as containing two signal types, though the specific context of which sentences relate to which signals would be tough to disambiguate.

Instead, we decided to focus on sentences or groups of sentences within the article. We refer to these smaller units as snippets. By narrowing the lens of the classifier, we focus the problem into a multi-label classification problem per snippet.

Though in many cases there will in fact be one signal per snippet (leading to more of a [multiclass classification ](https://en.wikipedia.org/wiki/Multiclass_classification)approach), there are many interesting snippets that can contain several signals.

Here is one such motivating example from  [PR Newswire](https://www.prnewswire.com/news-releases/doordash-secures-535-million-in-funding-from-softbank-sequoia-and-gic-to-expand-its-on-demand-platform-300606365.html) that contains 5 signals:

* *Snippet:*
    * Today DoorDash announced that it raised $535 million in Series D funding led by the SoftBank Group ("SoftBank") with participation from existing investors Sequoia Capital, GIC and Wellcome Trust.

* *Signals:*
    * DoorDash: **funding**
    * SoftBank Group: **investment**
    * Sequoia Capital: **investment**
    * GIC: **investment**
    * Wellcome Trust: **investment**

Broadly, therefore, we decided to tackle this multi-label problem through a natural language processing (NLP) and supervised machine learning (ML) approach. We knew our customers expect high precision out of these signals, so over the span of many years we created a golden set of tagged snippet data.


Gathering Data
--------------

The precursor to all of our work on classification comes from an essential dataset that is continuously being expanded by our highly trained auditors. DataFox has built a team of specialists who have collectively spent tens of thousands of hours combing through our news pipeline, tagging articles with relevant signals.

This huge amount of work has created a useful, clean and structured dataset with hundreds of thousands of relevant signal tag examples. Using our set of internal tools, an auditor will read through an article, highlight a relevant snippet, tag it with any relevant signals and associate it each signal with the relevant company.

![](https://lh3.googleusercontent.com/DxZaZ6lyMcunAMRCsO0U33_GPd_4diJ3iXF2XIiIExpj2i54CLborFEFInID8wVbxWhegJwFL9ZLq1dTpoZeYUBiSSYQPDtPeMerDAd5ISBbHhLcFwKZAOzmiPsBsYdzkltE92ME)

This human classification step creates structure for our training dataset and creates a significant competitive advantage for our machine learning approach.


Preparing Data
--------------

Returning to the crux of the multi-label problem, we knew in order to make sense of the tagged training data, we still needed to extract relevant features.  Specifically, we needed a way to turn the tagged snippet text into features that could be incorporated into a model.

In order to do this, we turned in part to the python library spaCy to perform a significant portion of data cleaning and pre-processing. Using spaCy helped us with the following tasks:

* Apply Named Entity Recognition (NER) to perform substitutions that allow our models to learn generalized sentence structures
    * 2017 or 2018 becomes DATE
    * $200M or €200 becomes CURRENCY
    * DataFox becomes ORGANIZATION

* Remove stopwords, using multiple approaches:
    * A list of known stopwords
    * Words that occur too frequently to be informative
        * maximum document frequency
    * Words that occur too infrequently to be informative
        * minimum document frequency

* Once the snippets had been prepared in this way, we tested a variety of other pre-processing steps:
    * N-grams where *N* = {1, 2, 3, 4}
    * Downsampling the untagged snippets
    * Attempt to avoid excessive bias in the model
    * Balancing the training data using class weights

* And we tried a few different methods to vectorize the text into features:
    * TF-IDF vectorization
    * Bag of Words - Count vectorization  
    * Singular Value Decomposition (SVD) on bag of words

Ultimately, we tested these different methods of preparing the data on different classes of models to gauge the relative effects on the classifier's precision and recall per signal.

Model Selection
---------------

After these data preparation steps, it was time to test out different models and gauge how well they classified the snippets in question. In a typical test/train setup, we chose to set aside 20% of the tagged data as a test set.

We then evaluated individual models using cross-validation on the remaining 80% of data, with discrete outputs of the the mean and variance for precision and recall across all cross-validated runs.

Here were a few of the model classes we tested employing scikit-learn's classes and run using its stochastic gradient descent classifier:

-   Random Forests

-   Support Vector Machines

-   Logistic Regression

In order to compare the results from these models, we used precision, recall and f1-scores per signal as guiding metrics. Additionally, to choose amongst similarly performing models, we considered the interpretability of the model.

In the end, a tuned logistic regression performed best, lending itself well to future re-training and giving us the ability to inspect the most important features contributing to each signal tag. Here are some high-level results from the model's performance on the test set, where the overall precision (78.1%) and the overall recall (73.2%) combined for a healthy f1-score of 74.6%.

<table class="pure-table">
<thead>
<tr>
<th>Signal Type</th>
<th>Precision</th>
<th>Recall</th>
<th>F1-Score</th>
</tr>
</thead>
<tbody>
<tr>
<td>valuation</td>
<td>99.5%</td>
<td>98.3%</td>
<td>98.8%</td>
</tr>
<tr>
<td>dividends</td>
<td>95.2%</td>
<td>99.1%</td>
<td>97.1%</td>
</tr>
<tr>
<td>stock-performance</td>
<td>94.1%</td>
<td>95.5%</td>
<td>95.0%</td>
</tr>
<tr>
<td>. . .</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td class="text-bold">All Signals</td>
<td class="text-bold">78.1%</td>
<td class="text-bold">73.2%</td>
<td class="text-bold">74.6%</td>
</tr>
</tbody>
</table>

In the end, this structured classification of the news brings in fascinating examples, from the international marketing activity of interestingly named French companies:

* *Snippet:*
    * Google Maps on Android and iOS now has a new transportation option. If you're living in a country where French startup BlaBlaCar operates, you can now open the [BlaBlaCar](https://www.blablacar.com/) app and book a ride straight from Google Maps.

* *Signals:*
    * BlaBlaCar: **marketing-activity**
    * Google: **marketing-activity**

To the can't miss details of massive funding rounds in Silicon Valley:
* *Snippet:*
    * Instacart said it raised $200 million in a new funding round this morning led by Coatue Management, as well as Glade Brook Capital Partners

* *Signals:*
    * Instacart: **private-funding**
    * Coatue Management: **investment**
    * Glade Brook Capital Partners: **investment**

Future Work
-----------

As always, at the end of such a project, there are exciting opportunities to improve the current methodology. Here are a few items that are top of mind heading forward:

-   At the tail end of our signal taxonomy are a few signal tags that haven't received enough tags in our training set to perform well in a classification setting. We have room to improve those signal classifications either by artificially boosting our training set, or manually tagging more examples to get a better breadth of examples.

-   We can consider using neural networks like Convolutional Neural Networks (CNNs) or Recurrent Neural Networks (RNNs). In this situation we would use word embeddings as features rather than sentence vector space models. An advantage of using a CNN could be to help make the classifier more context aware than N-grams preprocessing can provide. And an advantage of using an RNN could be making the classifier more temporally aware, processing one word at a time, as opposed to all at once.
