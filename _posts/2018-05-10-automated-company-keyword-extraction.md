---
layout:       post
uuid:         0e733c80-36c9-0136-3edf-186590cb4787
categories:   natural-language-processing
tags:         [natural-language-processing, nlp]
title:        'Automated Company Keyword Extraction'
date:         2018-05-10
author:
  name:       David Newcomb
  twitter:    null
  github:     null
feature_img:  null
sitemap:
  lastmod:    2018-05-10T14:43:10
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---
Whether you're a sales ops manager finding best-fit accounts or a VC investor ranking emerging companies in an industry, your process begins by narrowing down the complete universe of companies.

DataFox offers many paths to achieving this goal - you can create a company list filtered by headcount, location, revenue, technographics, parent-child company relationships, or recent [company signals](https://eng.datafox.com/machine-learning/2018/04/03/extracting-signals-from-the-news/), among many other options.

But the place many users begin is the keyword filter feature. Across all company lists created by DataFox users, approximately 40% utilize the keyword filter. Given this high usage rate, we decided to focus on improving the result set for any given keyword search.

## Improving Keyword Search
There were many areas for improvement with our keyword search functionality, and we decided to focus on these main points:

1.  It was sometimes necessary to search multiple similar keywords to get back the relevant results (e.g. needing to search both fintech & financial technology)

2.  Even after including many similar keywords in a search, some relevant companies still weren't being returned to the user

To address these concerns, we took a few steps. First, we normalized frequently searched keywords in the database: CRM became "customer relationship management", e-commerce became "ecommerce", etc. At search time, we then mapped a common spelling or common acronym to our normalized keyword to retrieve all companies that were assigned the normalized keyword. The process of identifying important keywords and their equivalents, though manual, had a large impact on the quality of common searches.

The second step aimed to solve the recall issue - when a user searches for companies using keywords, they expect to see all the companies that are relevant to those keywords. Our existing company keywords had come from structured data sources such as website meta tags, partnerships with data vendors, and manual overrides. But we needed more keywords per company, while maintaining high precision in our search use case. We had already built a website parser to identify and save company descriptions found on company websites, so we decided to use these descriptions as the source of new keywords.

## Extracting Relevant Parts of Descriptions
While already having descriptions from company websites and blogs was a great headstart, we needed to cleanse them in order to obtain better keyword extraction results. On a sentence-by-sentence basis, a company description could contain highly relevant information, followed by non-keyword-relevant language such as the following:

* Boilerplate Legal
    * "Everyone is welcome here. Please, refrain from hateful or discriminatory comments regarding race, ethnicity, religion, gender, disability, sexual orientation or political beliefs."
<br>
* Personal Statements
    * "I am honored to have been part of every struggle and achievement of Samsung Heavy Industries, and to have shared in the company's joys and sorrows."
<br>
* Non-English Languages
    * Latin: "lorem ipsum dolor sit amet, consectetuer adipiscing elit"...
    * Non-English: will be relevant in the future, but this project looked to extract English keywords
<br>
-   Parked Domains
    * "Get your own corner of the Web for less! Register a new .COM for just $9.99 for the first year and get everything you need to make your mark online --- website builder hosting, email, and more"
<br>
-   HTML
    * `<h2 style="margin-left 0px margin-right 0px">`

Using spaCy and high precision regexes, we removed these types of sentences from our company descriptions and thereby helped improve the quality of our keyword extraction.

## Choosing A Keyword Extraction Approach
There are many potential methods for solving this keyword extraction problem. We could build a labeled training set of descriptions-to-keywords, we could choose linguistic features and build a supervised model to [rank keywords](https://www.microsoft.com/en-us/research/publication/a-ranking-approach-to-keyphrase-extraction/). We could try an unsupervised approach such as [TextRank](http://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf), which uses a graph-based algorithm to score and choose important keywords. Or we could implement an unsupervised approach using [neural networks](https://arxiv.org/pdf/1004.3274.pdf).

However, given practical constraints, we decided against creating a full training set due to the time requirements. We also decided to focus mainly on unigrams and bigrams, given the pattern of searches already being done in the app, reducing the appeal of TextRank which can excel at longer keyphrases.

Given these factors, we chose to test a [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) approach with a high scoring threshold. The approach is a straightforward use of statistics, can be quickly iterated on and offers ease of model inspection and calculation. Further, it is well suited for keyword extraction because TF-IDF down-weights words that are extremely common across all companies and up-weights rarer words. Additionally, it gives weight to how frequently a word occurs within a company's descriptions.

TF-IDF requires defining documents that comprise a corpus and are comprised of tokens. In our keyword extraction framework, we concatenated all available company descriptions to create a single document per company. The set of all company documents then became our corpus.  Lastly, we needed to tokenize company descriptions. Many decisions can go into selecting those tokens, which essentially become our keyword candidates. In the next sections, we outline a couple of approaches to this tokenization or keyword candidate selection.

<img src="/assets/images/keyword_extraction_approach.png" width="100%">

<p align="center">
    <i>Figure 1: Keyword Extraction Approach</i>
</p>

In addition to varying our keyword candidate selection through tokenization, we varied other important parameters to the TF-IDF process. We tested different maximum and minimum document frequency ranges, thereby expanding or narrowing our keyword candidates.

And lastly, we tested various TF-IDF score thresholds across the various models to find a good balance between recall and precision of our keyword extraction.

## Keyword Candidate Selection
Since we knew what types of keywords were searched for in DataFox, we had existing parameters in our search for new keywords. For the most part, we were looking for unigrams and bigrams comprised of nouns and, less frequently, adjectives.

Therefore, on a word-by-word basis, there were many types of words which we did not want to consider as possible keywords. Using spaCy to determine parts of speech and speech tags, we blacklisted words like proper nouns and numerals.

We then applied this word tag filtering to two candidate selection strategies:
1.  **N-gram Tokens**
2.  **Noun Phrases**
<br>

## N-gram Tokens
In this first strategy, we began by pre-processing company descriptions. If any word fell into the aforementioned blacklisted word tags, we removed it from the description. Additionally, if any word was a stopword in any language, it was removed from the description. This helped save computational time by reducing the description length before tokenization.

Next, we tokenized the company descriptions into unigrams and bigrams. Once these candidates were selected, we applied a TF-IDF ranking to all keyword candidates on all companies and performed some post-processing as well:

1.  Removed blacklisted keywords
2.  Removed non-English keywords
3.  Removed non-nouns (except whitelisted gerunds)
4.  Removed non-consecutive keywords
    * e.g. services products  below does not appear consecutively in the original description, so it is not accepted as a valid keyword


As an example, the **N-gram Tokens** approach would transform this sentence into its respective keyword candidates:

  * **Description**: "*Google is a multinational corporation that specializes in trustworthy search services and high-tech products.*"

  * **Pre-processed Description**: "*multinational corporation specializes search services high-tech products*"

  * **Raw Tokens**: [multinational, corporation, specializes, search, services, high-tech, products, multinational corporation, corporation specializes, specializes search, search services, services high-tech, high-tech products]

  * **Post-Processed Tokens**: [search, services, products, search services]

## Noun Phrases
Our second approach aimed to use [noun phrases](https://en.wikipedia.org/wiki/Noun_phrase#Identifying_noun_phrases) as tokens. This approach uses sentence structure to narrow the search for keywords. In a noun phrase, the noun is the grammatical head of a phrase which typically includes determiners, adjectives, or prepositions but can include any number of other grammatical components.

This approach allowed us to pick up on trigrams like "internet of things" in addition to bigrams and unigrams (any N-gram beyond N=3, we decided not to consider). However, the approach also had the negative potential to incorporate too many unwanted noun phrase types. Therefore, we subjected leading tokens from the noun phrases to a harsher check by removing them if they fell into the following spaCy tags:

    DT - determiner (THE problem)
    JJR - comparative adj (WORSE problem)
    JJS - superlative adj (WORST problem)
    NNP - proper singular noun (a DAVID problem)
    NNPS - proper plural noun (the DAVIDS' problem)
    PRP - personal pronoun (THEY are problems)
    PRP$ - possessive pronoun (MY problem)
    RB - adverb (CLEARLY a problem)
    RBR - comparative adverb (MORE CLEARLY a problem)
    RBS - superlative adverb (MOST CLEARLY a problem)
    WDT - possessive wh-determiner (WHAT a problem)
    WP - personal wh-pronoun (WHERE a problem)
    WP$ - possessive wh-pronoun (WHOSE problem)

After all unacceptable leading tokens were removed, we checked the phrase for length (it could not contain greater than 3 tokens). Then each token was checked for validity. This validity check was the same as the above blacklisting of certain word tags (proper nouns, company stopwords, etc.).

Lastly, post-processing was not as extensive as with the **N-gram Tokens** approach. Since we were evaluating noun phrases, we didn't need to worry about non-consecutive keywords and non-nouns were considered acceptable. Therefore, we only excluded blacklisted keywords and non-English keywords.

Going back to the Nike example from the previous section, here is what the process and final keyword candidate selection looked like:

  * **Description**: "*Google is a multinational corporation that specializes in trustworthy search services and high-tech products.*"

  * **Raw Noun Phrases**: [Google, a multinational corporation, trustworthy search services, high-tech products]

  * **Post-Processed Noun Phrases**: [trustworthy search services, high-tech products]

## Removing Uninformative Keyword Candidates
For each approach, we also limited keyword candidates by document frequency. We tested various ranges for the minimum and maximum document frequency. If any token existed in fewer documents than the min_df or more documents than the max_df, then that token was removed from keyword candidacy.

The table below shows a few representative ranges. Based on an examination of the existing keyword frequency in our database, there is a long tail of reasonable (though rare) keywords that are only present in  about 10 companies (e.g. bone densitometry). More commonly searched keywords tend to be present in at least 400 companies (e.g. video conferencing).

In our database, approximately 100 keywords are present in above 10K companies (e.g. software, manufacturing), so the document frequency's upper range varies between 10K and 100K, depending on how much precision we want to sacrifice for recall of the most common keywords. Applying these ranges left us with the following four models:

<table class="pure-table" style="margin: 0px auto;">
<thead>
<td>Model</td>
<td>Min DF</td>
<td>Max DF</td>
</thead>
<tbody>
<tr>
<th></th>
<th></th>
<th></th>
</tr>
<tr>
<td>NGRAM TOKENS</td>
<td>10</td>
<td>100K</td>
</tr>
<tr>
<td>NGRAM TOKENS</td>
<td>400</td>
<td>10K</td>
</tr>
<tr>
<td>NOUN PHRASE</td>
<td>10</td>
<td>100K</td>
</tr>
<tr>
<td>NOUN PHRASE</td>
<td>400</td>
<td>10K</td>
</tr>
</tbody>
</table>
<p align="center">
  <i>Table 1: Minimum & Maximum Document Frequencies by Model</i>
</p>

## Evaluation
We didn't have a test dataset linking keywords to descriptions, so we had to get a little creative when defining metrics for evaluation. The goal of keyword extraction is to obtain as many keywords as possible while introducing as little noise as possible. Therefore, measuring the performance of keyword extraction using [recall, precision](https://en.wikipedia.org/wiki/Precision_and_recall) and the combined [F1 score](https://en.wikipedia.org/wiki/F1_score) makes sense.

In order to approximate measures of recall and precision, we turned to our existing keywords on company profiles. Using exact matching, we checked to see how many of these keywords existed within our company descriptions. For every company this gave us an array of keywords which were "discoverable" by our keyword extraction process. This set of companies became our test set.

Next, among the keywords extracted from a company, we checked to see how many of the discoverable keywords were actually discovered. These were our true positives. We treated all other extracted keywords as our false positives for the sake of creating these metrics.

The obvious caveat is that many of these "false positives" could have been legitimate keywords. This falsely decreases our precision but allows us to have some notion of recall. After all, if we simply extract every word in a description as a keyword, we'll capture all discoverable keywords. We want a measure to help expose this flaw.

Lastly, any discoverable keywords that were not discovered were our false negatives. Given these definitions, we calculated recall, precision and F1 score for our extraction approaches under different TF-IDF score thresholds, ranging from 0 to 1 (i.e. from very permissive to highly selective).

As we see in **Figure 2**, each model reaches its peak F1 Score performance at different TF-IDF score thresholds - this is a direct product of the different underlying score distributions for each model. The more interesting feature is the max F1-score that each model achieves.
<img src="/assets/images/keyword_extraction_f1scores.png" width="100%">
<p align="center">
    <i>Figure 2: F1 Scores By Model and TF-IDF Score Threshold</i>
</p>

The max F1 scores indicate that the **N-gram Tokens** models had better peak performance than the **Noun Phrases** models. And it shows that allowing a wider range of keywords (by document frequency) can improve the max F1 score.

Now, the *N-gram Tokens (10-100K)* model receives a large boost in recall, not precision, in comparison to the *N-gram Tokens (400-10K)* model, accounting for much of the up-tick in F1-score. Further, this methodology does not reveal anything about the precision of new keywords extracted. In **Table 2**, we see that the *N-gram Tokens (10-100K)* model would add about 25 times more new keywords than the *N-gram Tokens (400-10K)* model, while affecting only 100K more companies.


<table class="pure-table">
<thead>
<td>Model</td>
<td>Min DF</td>
<td>Max DF</td>
<td>TF-IDF Threshold</td>
<td>Num. Companies</td>
<td>Num. Unique New Keywords</td>
<td>Median kw / co.</td>
</thead>
<tbody>
<tr>
<th></th>
<th></th>
<th></th>
<th></th>
<th></th>
<th></th>
<th></th>
</tr>
<tr>
<td>NGRAM TOKENS</td>
<td>10</td>
<td>100K</td>
<td>0.15</td>
<td>1.19 M</td>
<td>261,577</td>
<td>4</td>
</tr>
<tr>
<td>NGRAM TOKENS</td>
<td>400</td>
<td>10K</td>
<td>0.25</td>
<td>1.08 M</td>
<td>10,247</td>
<td>2</td>
</tr>
<tr>
<td>NOUN PHRASE</td>
<td>10</td>
<td>100K</td>
<td>0.25</td>
<td>1.14 M</td>
<td>124,852</td>
<td>4</td>
</tr>
<tr>
<td>NOUN PHRASE</td>
<td>400</td>
<td>10K</td>
<td>0.3</td>
<td>1.00 M</td>
<td>6,757</td>
<td>2</td>
</tr>
</tbody>
</table>

<p align="center">
    <i>Table 2: Number of Companies & Unique Keywords by Model</i>
</p>

Due to the uncertainty around adding new keywords, we preferred the more conservative model, which adds fewer new keywords per company, but still augments keywords on 1.08 million companies.


## Results
Based on our work above and manual spot checking which corroborated the relative quality of each model, we moved forward with the *N-gram Tokens (400-10K)* approach with a TF-IDF score threshold of 0.25. Of the 1.08 million companies that benefited, here are some noteworthy examples of new keywords we extracted:
* Slack: **software platform**
* Genentech: **medicines**
* DataFox: **marketing teams, prospects**
* Twitter: **tweets, networking platform**
* Nike: **apparel, footwear**
* American Airlines: **airline, flights**
* Flatiron Health: **cancer, cancer research**
* WeWork: **buildings, workspaces**
<br>

## Future Work
We can continue to improve the quality and breadth of company descriptions. Increased effort to clean descriptions will lead to higher quality keywords.

We should also revisit either the noun phrase or TextRank approaches to help extract more complex keyphrases. Since they are each more fundamentally based in the structure of a sentence, they can encode and better utilize the context of a keyword/keyphrase.

Further, we could apply multiple models to rank extract keywords and use a weighted voting scheme to finalize our keyword selection. As with any NLP project, there are many directions left to explore!
