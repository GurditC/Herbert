# Herbert

## Website
[You can visit the website by clicking here!](http://w210herbert-dev.us-west-2.elasticbeanstalk.com/)<br/> 

## What is Herbert
Herbert is a semantic search engine aggregates and distills the most reliable herbal medicine information into a curated single report view for consumers curious about using herbal medicine.<br/>

## Why is it important
Total retail sales of herbal pharmaceuticals surpassed $8.8 billion domestically and the growth has been accelerating over the past decade. Although there is a lot of interest in herbal medication it is difficult for consumers to get trustworthy, reliable, and easy to understand information about the treatments. Standard web searches return a lot of information, but leave it up to the user to sift through all the pages to find the information that is relevant to them. Other services are designed for medical professionals and use complicated jargon that make it difficult for the layperson to understand. While there is lots of interest in alternative herbal medicine, there is also information overload.<br/>

## What are our advantages
*Efficient: Herbert filters irrelevant information and focuses on the semenatic relationship among herbs, conditions and interactions<br/>
*Trustworthy: Herbert aggregates information from trustworthy data sources and cross references among them<br/>
*User Friendly: Herbert avoids using overly technical terms and uses layman vocabularies for easy understanding<br/>
*Transparent: Herbert provides links back to original data sources for user reference<br/>

## Technical Description

![Proccess gif](https://github.com/GurditC/Herbert/tree/master/img/herbert.gif)

Our solution to distilling our sources into the relevant data points about an herb involves a multi-stage pipeline of finer granularity of text at each stage. Essentially, we break down pages of texts to get relevant paragraphs, and then relevant paragraphs to relevant sentences, until we finally reach relevant phrases which are turned into our bullet points.

To get our pages of text we use a combination of restful APIs for available sources such as [PubMed](https://www.ncbi.nlm.nih.gov/home/develop/api/) and [Wikipedia](https://www.mediawiki.org/wiki/API:Main_page) and [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) + [Requests](https://requests.readthedocs.io/en/master/) Python libraries for the others that don’t have APIs such as NCCIH.<br/>

In order to explain our general extraction pipeline, we refer to an example with a Wikipedia page on ginger illustrated by the animation above.<br/>

We start off by getting the relevant paragraphs by selecting the relevant headings from the table of contents.<br/> 

To get the desired headings, we set ‘seed words’ that are related to topics of interest (e.g. the topic of side-effects with “adverse”, “side-effect”, “interact”, and the like).<br/> 

We essentially compare our seed words with the content headings and those “similar enough” to our seed words dictate which paragraphs are relevant while the rest are discarded. The two are made comparable through word embeddings augmented with character level n-grams which can be seen in Mikolov et. al’s paper [“Enriching Word Vectors with Subword Information”](https://arxiv.org/abs/1607.04606). Essentially, we look at not only whole words but chunks (character n-grams) in case we don’t have the word in the vocabulary but can make use of root words, prefixes, suffixes, etc. The word embedding model was trained on millions of PubMed abstracts, full text from the [PubMed Central Open Access subset](http://www.ncbi.nlm.nih.gov/pmc/tools/openftlist/), and texts from an English Wikipedia text dump. The vectors are compared by a cosine similarity and we set a threshold (empirically determined) to decide how “similar” the vectors need to be in order to retain the heading.<br/>

Now that we have our relevant paragraphs (pointed by the content headings),  we look to grab relevant sentences. We do this by using a government supported medical ontology software called [UMLS](https://www.nlm.nih.gov/research/umls/index.html) to identify sentences  with medical content relating to conditions, symptoms, or other medical objects of interest.<br/>

From each relevant sentence, we find our relevant phrases through [relation extraction](http://resources.mpi-inf.mpg.de/d5/clausie/clausie-www13.pdf) in which we look for subject-verb-object triples (SVOs)-the example above being “Ginger alleviates nausea”.<br/> 

Finally, we apply a similar process to the process described to extract content headings (word2vec + cosine similarity + thresholding). We look for verbs in the SVOs that indicate whether the phrase is explaining what the herb either treats, interacts with, or causes.
We normalize the words and combine them along other data sources and put them on the landing pages for our herbs.<br/>


![Summary Diagram](https://github.com/GurditC/Herbert/tree/master/img/TextRank.png) 

For summaries of conditions + herbs, we use [Wikipedia](https://www.mediawiki.org/wiki/API:Main_page) for both entity resolution and the source for summary content. We summarize via an unsupervised graph-based approach known as [TextRank](https://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf).<br/> 



 ![Query Formula](https://github.com/GurditC/Herbert/tree/master/img/BM25.png) 

The underlying search engine is supported by the Python library [Whoosh](https://pypi.org/project/Whoosh/) with the [Okapi Best Matching-25 (BM-25)](https://web.stanford.edu/class/cs276/handouts/lecture12-bm25etc.pdf) algorithm for relevance.<br/>

