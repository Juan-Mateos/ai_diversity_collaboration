### Collection

We use [OpenAlex](https://www.openalex.org) as our core research data source. OpenAlex is an open scientometric database developed to replace Microsoft Academic Graph, a database of academic publications that was recently discontinued. OpenAlex includes information about:

- Works (papers, books, datasets etc., [example](https://api.openalex.org/works/W2741809807))
- Authors (who create works, [example](https://api.openalex.org/authors/A2208157607))
- Venues (journals/repos that contains works, [example](https://api.openalex.org/venues/V1983995261))
- Institutions (organisations/institutions affiliated with a work, [example](https://openalex.org/I114027177))
- Concepts (tags works with topics based on a machine learning analysis of the MAG corpus [example](https://openalex.org/C2778407487))[^2])

[^2]: See [here](https://docs.google.com/document/d/1OgXSLriHO3Ekz0OYoaoP_h0sPcuvV4EqX7VgLLblKe4/edit) for additional information about OpenAlex' concept-tagging methodology.

OpenAlex is a new database but current discussions suggest that it is the best open dataset available. Its coverage and accuracy is already comparable to established players such as Scopus or Dimensions, and it may [surpass MAG](https://arxiv.org/abs/2206.14168) in some aspects.

OpenAlex provides an [open and accessible API](https://docs.openalex.org/api) for querying and collecting data from its corpus. Our approach has been to use broad, high-level concepts related to our areas of interest, and query the corpus of works for each of the last 15 years that have been tagged with those concepts, to build a collection of works that covers all potentially relevant papers. Through a combination of batch processing these queries in parallel, and the high level of efficiency and stability of OpenAlex's API, this collection can be done in under a day.

We have collected all the OpenAlex `works` (papers) tagged with the concepts "Artificial intelligence" (around 2 million documents).

### Generic processing

We have removed from our corpora all works missing an abstract and applied the [FastText language classifier](https://fasttext.cc/docs/en/language-identification.html) to titles in order to identify those in English, and remove the rest.[^3]

[^3]:
    Concept tagging in non-English articles might be less reliable, and they are not suitable for downstream analysis using Natural Language Processing. We could include in specific parts of the analysis if useful. ]
    This leaves us with 1.5 million AI articles.

### Artificial intelligence corpus processing

A visual inspection of a subset of the AI corpus suggests that there is a non-trivial number of false positives in the data. In particular, we have noticed a number of systematic misclassifications of papers in the following topics:

- Education: This includes papers about learning and learners, which get confused with machine learning.
- Neuroscience: This includes papers about biological neural networks, which get confused with artificial neural networks used in AI research.
- ICT: This includes papers about telecommunications, ICT networks and cybersecurity that get confused with artificial neural networks and deep networks used in AI research.
- Linguistics: This includes papers about language, translation etc., which get confused with computational linguistics and natural language papers.

We have adopted two strategies to remove false positives from the corpus: First, we have implemented a heuristic filter that removes from the corpus any papers that include terms related to the categories above such as "learning", "neural network" or "language" but no terms related to machine learning and artificial intelligence (see @tbl:terms).

| Terms                                                                                                                                                                                                                                                                                                                                                                                |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| accuracy, ai, ann, artificial, artificial intelligence, bayes, classifier, clustering, convolution, deep, dnn, encoder, federat, gan, generative, gnn, machine, machine learning, natural language processing, nlp, pattern recognition, predict, rbf, reinforcement, representation, rnn, state of the art, state-of-the-art, statistical, supervis, train, transformer, unsupervis |

: Terms related to AI and machine learning in our heuristic filter {#tbl:terms}

Second, we have excluded articles with concept scores below a threshold value for key concepts. More specifically, we have only retained articles which have either a score above 0.4 in "Artificial intelligence" and/or a score above 0.3 in "Machine learning" and/or a score above 0.6 in "Deep learning".[^4]

[^4]: We note that the resulting corpus will include AI and machine learning papers that use statistical machine learning techniques such as random forests, support vector machines and gradient boosting which the literature suggests have played an important role in the biological sciences and genomics.

We have selected these thresholds after evaluating the impact of various combinations of values on a subset of the data (comprising the years 2012, 2017 and 2021) which we have labelled with information about 1,702 preprints submitted to top AI conferences (including NEURIPS, ICML, CVF, ECCV, AAAI, PMLR, SIGKKD and IJCAI) identified in the [Papers with Code](https://paperswithcode.com/) corpus and 2,256 preprints from the arXiv that are not labelled with AI related categories (cs.AI, cs.CL, cs.CV, cs.IR, cs.LG, cs.NE, cs.SO, math.ST, q-bio.QM, stat.ML), and which we assume are not-related to AI or Machine Learning.

The aforementioned combination performs best in terms of the F1-Score (which combines precision and recall i.e. the ability of the definition to generate accurate positive predictions while reducing the numer of erroneous negative predictions), with a score of 0.73.[^5] This is underpinned by 0.68 predictive accuracy and 0.79 recall.

@tbl:examples presents some examples of papers which are correctly / incorrectly classified with our current definition. The "word soup" paper has been misclassified because the OpenAlex system assigned it "Speech recognition" and "Natural Language Processing" tags. In the false negative case of the reinforcement learning paper, we notice that the AI score is below our threshold but (unsurprisingly) the reinforcement score is very high (0.7) suggesting potential avenues to improve our classification by including higher-granularity concepts into our selection procedure.

| Status         | Title                                                                                    |
| -------------- | ---------------------------------------------------------------------------------------- |
| True positive  | A Survey on Language Modeling using Neural Networks                                      |
| False positive | Transcribing handwritten text images with a word soup game                               |
| True negative  | Pedagogical models of concordance use: correlations between concordance user preferences |
| False negative | Dimension-Free Rates for Natural Policy Gradient in Multi-Agent Reinforcement Learning.  |

: Examples of papers in different categories {#tbl:examples}

[^5]: More formally, precision is defined as the percentage of entities which are predicted to be in a class (in our case AI papers) that are in fact in that class; recall is defined as the percentage of instances that are in a class that are predicted in that class.

When we apply all our filters to the data, including the heuristic filters and the concept-based filters, this results in a corpus with 1m observations. Although this is a substantial decrease from the initial 1.9m corpus, it is still likely to overestimate the number of AI papers for reasons noted above. This could be addressed with downstream filtering.

## Other known issues

* Some works are not tagged with lower-level concepts in the taxonomy
* Relevant concepts like Machine learning are not descendents of "Artificial Intelligence"

These will be addressed in follow-up work.