In this page, I will document the **detailed** though process as well as some technical choices I made through out the document suggestion project.


## Problem Formulation

In this work, we try to recommend documents *D<sub>i</sub>* from a document pool *C* based on a single news article *S*. The pool can either consists of news articles (2018 News Track Background Linking Task), or blogs (2009/2010 Blog Track News Blog Post Ranking Task). Regardless of the genre of the pool, the suggested documents could help reader better digest the source document *S*, and provide interesting documents related to *S* for readers to explore.


## General Framework

The general framework consists of two parts:

1. Document retrieval
2. Retrieval result merging

The framework is inspired by the structure of news articles. According to Fox [1], a news article often starts with a summary of the main story, following by detailed coverage of major aspects of the story. The aspects are reported in the order of importance and the unity and completeness of the writing of the aspects are ensured so that even latter an editor decides to cutoff a part of the article from the end, which are aspects that are least important, the story itself will not damaged. Therefore, we can assume that **each paragraph** covers **an aspect** of the original document, and use the paragraph to retrieve documents related to that aspect of the source document. 

This step requires we to have the paragraph structure/boundary for each source article and do retrieval using these paragraphs. Getting the paragraph structure seems to be straight forward for the News Track data as the structure is provided from the json structure of the article file (<span style="color:red">I am now focusing on the News Track data and will latter put in how I processed the source articles of Blog Track data</span>).

Paragraphs can be used to retrieve documents related to different aspects of a news article, but it might not be reasonable to use *all* of them separately and assume that each paragraph is about an aspect that is different than the aspects of other paragraphs. Although the logic breaks of different topics can probably only happen at the paragraph physical boundaries, but different paragraphs do not necessarily have to be about different aspects. Adjacent paragraphs can be about the same aspect but are broken apart to enhance readability. Therefore, ideally these paragraphs should be merged to groups for different aspects. <span style="color:red">I have not handled this part. It should be addressed in the future. For now, I have used all paragraphs and assumed that they each belongs to a unique aspect.</span>

There are different ways to retrieve documents based on a paragraph. According to the track reports of the News Track on the TREC website the best performing run simply use all words in a document to do retrieval. Therefore, the first and simplest way is to use all words. This method can be denoted as `All_Words` (<span style="color:red"> Have not done that yet, will add it to the baselines latter</span>). I tried a different method. It is motivated by the assumption that the paragraph is about a aspect of the document. Therefore, if I could mine the language model of that specific aspect and use it to retrieve documents, the results would be better than using all words. More specifically, I assume each paragraph is generated following the process described below:

![alt text](http://www.sciweavers.org/tex2img.php?eq=%281-%5Clambda_c%29%2A%28%20%281-%5Clambda_d%29%2A%5Ctheta_p%20%2B%20%5Clambda_d%2A%5Ctheta_d%20%29%20%2B%20%5Clambda_c%2A%5Ctheta_c&bc=White&fc=Black&im=png&fs=12&ff=arev&edit=0)

where $\theta_c$ denotes the collection language model, $\theta_b$ denotes the document language model, and $\theta_p$ denotes the paragraph language model. Collection model can be obtained using collection statistics and other two models need to be estimated. I tried to PLSA based methods to estimate them. The first one is incorporating the document language as part of the PLSA whereas the second method used MLE for document language model and use that in the PLSA. 
