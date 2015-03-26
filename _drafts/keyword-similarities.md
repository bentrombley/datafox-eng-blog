---
layout: post
title:  "Keyword-Based Company Similarity"
date:   2015-03-23
categories: general
---

Finding Similar Companies
-------------------------

One of the features of our product that many of our customers love is our
"Related Companies" module. On the profile of any company in our database, we
provide our users with a list of other companies that we suggest might be
related. A lot goes into these suggestions, including both some manual effort
and a lot of machine learning. An army of analysts could not comb through
our half million companies and come up with a good list of related companies for
each from the other half million (minus one) companies. And we don't want to
spend our time doing that either, so we apply a combination of engineering and
machine power to the problem.

Our related companies algorithm takes a number of inputs into account (company
sector, company size, news co-mentions, etc), but here we will focus on one aspect
of the algorithm: company keywords. Every company we have in our database is
associated with some list of keywords, most of which we scraped from their
home pages. Since this is how a company describes itself, mostly with the purpose
of luring visitors from organic web searches, it is an accurate list of words
to summarize the company's product and/or industry. By looking at the keywords
listed on a company's home page, we can more accurately triangulate similar
companies by comparing their keywords.

However, comparing two lists of keywords is not entirely obvious. We could look
at some metric based upon exact matches between keyword lists, such as Jaccard
similarity or cosine similarity, but keywords are a very sparse feature. Out of
the thousands of keywords in our database, any given company will list only about
a dozen. To deal with this, we want to be able to harness a measure of similarity
between keywords. After all, if company A lists "cloud storage" as a keyword,
then we should have more confidence they are related to a company listing
"file sharing" as a keyword than a company listing "mobile payments" as a
keyword.

While it can be hard to determine how to compare two keywords, we all know many 
ways to compare two vectors, whether by Euclidean (\\(L_2\\)) distance, Manhattan
(\\(L_1\\)) distance, cosine distance, or any other crazy metric you want to
come up with. If only we could translate our keywords to vectors...


word2vec
--------

<!-- word2vec default dims? -->
<!-- NIPS year? -->
<!-- word2vec examples? -->
A very interesting recent project out of [Google](https://datafox.co/google) does
just that. Training a neural network on word-context occurrences in a huge corpus
of text, *[word2vec][word2vec-google-code]* encodes words as vectors.
The basics are described at the
[Google Code repository][word2vec-google-code]
for their Python implementation, and you can find more detail in a
[series][word2vec-paper-1] [of][word2vec-paper-2] [papers][word2vec-paper-3]
linked to from the word2vec Google code page.
The vector encodings they produce have some surprising and desirable arithmetic
properties. Using an implementation trained on the Google News data set, they
provide a couple of interesting examples of how arithmetic operations on the
vectors capture some semantic meaning:

* `word2vec('king') - word2vec('man') + word2vec('woman')` is close in cosine
  similarity to `word2vec('queen')`
* `word2vec('france')` is close in cosine similarity to `word2vec('spain')`,
  `word2vec('belgium')`, etc.

It would be great if we could harness these same ideas in our context.
Unfortunately, using the default *word2vec* implementation directly is not
optimal because the data it is trained is too broad and misses out on the
nuances of domain-specific training data.
<!--
for a couple of reasons: (a) the data it is trained on is too broad
and misses out on the nuances of a domain-specific approach, and (b) the default
implementation only considers single words, whereas keywords are often multiple
words or short phrases (e.g. "mobile payments" or "cloud storage").
-->


Neural Networks, or Matrix Factorization
----------------------------------------

<!-- look at word2vec docs on implementation... -->
We want to train a "*keyword2vec*" model so we can more robustly compare both
keywords themselves and companies based on their listed keywords.
The *word2vec* project provides
a way to train using a new data set, but training neural networks is never easy:
they are sensitive to parameter tuning (step-size schedules for stochastic gradient
descent, number of hidden layers, number of hidden units in each layer, etc.),
they are not very interpretable, and they are often slow to train.

A recent paper from NIPS 2014,
[Neural Word Embedding as Implicit Matrix Factorization][word-embed-mf], shows
that the neural
network implementation of *word2vec* is actually very similar to a low-rank
factorization of a certain matrix. In particular, the word vectors represent
the loadings found in a matrix factorization of a positive
[pointwise mutual information][pmi-wikipedia] (\\(PPMI\\)) matrix
\\(P \in \mathbb{R}^{V \times V}\\), where

$$
P_{w,v} = PPMI(w,v) = \max\left\{0, PMI(w,v)\right\},\;\;\;
PMI(w,v) = \log \frac{p(w,v)}{p(w)p(v)}
$$

in which \\(V\\) is the size of the vocabulary, \\(w\\) and \\(v\\) are words,
\\(p(w)\\) is the probability of seeing word \\(w\\),
and \\(p(w,v)\\) is the probability of seeing words \\(w\\) and \\(v\\)
together.

In our setting, we find \\(p(w,v)\\) by counting co-occurrences of words \\(w\\)
and \\(v\\) in a company's keywords set, while \\(p(w)\\) and \\(p(v)\\) are
found from total occurrences across all companies' keywords.

To obtain our word vectors, we now factorize the \\(PPMI\\) matrix. Provided a
desired dimension \\(d\\) for the word vectors, we find a factorization of
\\(P\\) so that

$$
P \approx W W^T
$$

where \\(W \in \mathbb{R}^{V \times d}\\). Notice that we can motivate a
factorization of this form (\\(W W^T\\)) because \\(P\\) is a symmetric matrix
-- i.e. \\(P_{w,v} = P_{v,w}\\). As it turns out, finding an optimal solution
in terms of the \\(L_2\\) norm is feasible by taking the
[singular value decomposition][svd-wikipedia] (SVD) of the matrix,
\\(P = U \Lambda U^T\\), and keeping only the first \\(d\\) dimensions of the
diagonal matrix \\(\Lambda\\). It turns out that
\\(\tilde{P} = U \tilde\Lambda U^T\\), where \\(\tilde\Lambda\\) contains only
the first \\(d\\) elements of the diagonal matrix \\(\Lambda\\), is the
rank-\\(d\\) matrix that minimizes the Frobenius norm of the difference between
\\(P\\) and any other rank-\\(d\\) matrix -- i.e.

$$
\tilde{P} \in \arg\min_{\hat{P}\;:\;\textrm{rank}(\hat{P}) = d} ||P - \hat{P}||_F
$$

Using this, we go back to our earlier problem of finding \\(W\\) so that
\\(P \approx W W^T\\). If we take

$$
W = U \tilde{\Lambda}^{1/2}
$$

we now have a matrix \\(W\\) such that \\(WW^T = U \tilde\Lambda U^T\\) is very
close to our original
\\(PPMI\\) matrix \\(P\\). Furthemore, each row of \\(W\\) can now be used as a
vector representing the corresponding word -- i.e. \\(W_{i\cdot}\\) is the
\\(d\\)-dimensional vector corresponding to the \\(i^{th}\\) word. And, it turns
out, these vectors have similar properties with the vectors obtained from
*word2vec*.


Using the Keyword Vectors
-------------------------

Now that we have our own trained vectors for the keywords obtained for our
companies, we can have some fun!

Similar Keywords
================

We can find similar keywords by comparing the vector representation of one
keyword to the vector representations of all other keywords. Looking, for
example, at the keyword  **cloud storage**, we find that the closest other
keywords in terms of cosine similarity are

<!-- TODO: a table would be nice, but then have to mess w/ the css -->
<!-- TODO: choose one, or another one entirely (also, can cherry-pick...) -->
* **flash storage**: 0.981116179237997
* **idrive**: 0.978347988316545
* **online file sync**: 0.977817384377984
* **your external hard drive in the cloud**: 0.973818440471018
* **backup online**: 0.973174935235816
* **online storage**: 0.970892387415698
* **free online storage**: 0.970609131103706
* **online backup solutions**: 0.969503062603722
* **online file backup**: 0.968670947689898
* **online sync**: 0.965037194231968
* etc...

and for the keyword **mobile payments**,

* **mobile payment**: 0.985906495453295
* **payments**: 0.982129350125816
* **android payments**: 0.979123374750015
* **ivr payments**: 0.978475369212524
* **mwallet**: 0.977912206702434
* **fintech   payments**: 0.977378781616218
* **mobile payment enabler**: 0.974429434165879
* **mobile payment solution**: 0.974288467923257
* **credit cards & transaction processing**: 0.974219772716241
* **card payments**: 0.972751346246811
* etc...


Similar Companies
=================

Although we incorporate much more into our "Related Companies" algorithm, the
keywords alone can provide pretty good similarity scores. If we encode each
company as the mean of its keyword vectors, we can look again at the cosine
similarity between each company and all other companies.

Some examples are, for [Dropbox](https://datafox.co/dropbox) (staying with the
cloud storage / collaboration theme),

* [Box](https://datafox.co/box): 0.988772243220449
* [SugarSync](https://datafox.co/sugarsync): 0.973269196861258
* [Hightail](https://datafox.co/hightail): 0.972030152842286
* [SkyDox](https://datafox.co/skydox): 0.964419747675301
* [filobite](https://datafox.co/filobite): 0.961857655119854
* [Firedrive Media](https://datafox.co/firedrive-media): 0.959809673680361
* [Yakimbi](https://datafox.co/yakimbi): 0.958234583938729
* [Mediafire](https://datafox.co/mediafire): 0.956988047920301
* [Filesgateway.com](https://datafox.co/filesgateway-com): 0.956174150033562
* [Zoolz](https://datafox.co/zoolz): 0.955439684268182
* etc...

and, for [BOKU](https://datafox.co/boku) (staying with the mobile payments
theme),

* [PayAnywhere](https://datafox.co/payanywhere): 0.967311639462488
* [Zong](https://datafox.co/zong): 0.966622717338347
* [Square](https://datafox.co/square): 0.962493155772685
* [VeriFone](https://datafox.co/verifone): 0.957852113994113
* [Fortumo](https://datafox.co/fortumo): 0.956916229626648
* [QFPay](https://datafox.co/qfpay): 0.956654558103476
* [Onebip](https://datafox.co/onebip) (acquired by [Neomobile](https://datafox.co/neomobile)): 0.9550171271087
* [PaybyMe](https://datafox.co/paybyme): 0.954839544810337
* [Zaypay.com](https://datafox.co/zaypay): 0.953663005838281
* [obopay](https://datafox.co/obopay): 0.953425806463978
* etc...

<!-- ALSO: could plot vectors in first 2 dimensions for various keywords / companies -->

[word2vec-google-code]: https://code.google.com/p/word2vec/
[word2vec-paper-1]: http://arxiv.org/pdf/1301.3781.pdf
[word2vec-paper-2]: http://arxiv.org/pdf/1310.4546.pdf
[word2vec-paper-3]: http://research.microsoft.com/pubs/189726/rvecs.pdf
[word-embed-mf]: http://papers.nips.cc/paper/5477-neural-word-embedding-as-implicit-matrix-factorization.pdf
[pmi-wikipedia]: http://en.wikipedia.org/wiki/Pointwise_mutual_information
[svd-wikipedia]: http://en.wikipedia.org/wiki/Singular_value_decomposition
