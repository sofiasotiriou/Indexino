# Findings and comparisons of different types of information retrieval 
*It is important to note that all comparisons are done on specific metrics: number of relevant documents retrieved (out of the 4928 that are actually relevant), Mean Average Precision, Geometric Mean Average Precision, and Precision on the first 5, 10, 15, 20 retrieved documents.*

## First phase 
The creation of the index is straightforward, the part that could affect the results is the structure of the query or the weights of the similarity function parameters. 

After experimenting with only using specific fields with high relevance like "title" and "text" or using all the fields but with varying weights or even changing the parameters of the similarity function, it was actually found that the best results were produced when all the fields where used without weights and the default similarity function.  <br>
<img src=https://github.com/user-attachments/assets/b540c749-2c81-4030-a260-2653bd1a8d26 width="35%"> 

This configuration produces the following results: <br>
- For size = 20 <br>
<img src=https://github.com/user-attachments/assets/f9833246-86b2-4276-90c3-77a8cadfcaee width="30%">
<img src=https://github.com/user-attachments/assets/e924420e-0540-46f7-ae52-e8fbf8cf0b7f width="30%">

- For size = 30 <br>
<img src=https://github.com/user-attachments/assets/fe079a66-d078-47b6-a1ec-694b68491974 width="30%">
<img src=https://github.com/user-attachments/assets/2db0f3bf-6cba-4ada-a35a-fe3316ef0d4c width="30%">

- For size = 50 <br>
<img src=https://github.com/user-attachments/assets/e2a05ae5-5722-4b6d-9d01-b3d55cc9bbe8 width="30%">
<img src=https://github.com/user-attachments/assets/ad9cc25e-c4e3-4fd5-aa2c-922a505fe43f width="30%">

Then, it was found that the allContent field of a document (which contains all the fields of a document concatenated) provides the best best results when all the fields of a query are compared to it, instead of the field-by-field comparison above. More specifically, this code <br>
<img src=https://github.com/user-attachments/assets/e5aa40e0-f020-44d6-ba72-eff5acca65fc width="40%"> <br>
produces the following results:

- For size = 20 <br>
<img src=https://github.com/user-attachments/assets/d521fb38-eef4-4eee-8dc9-2ad881bb0221 width="30%">
<img src=https://github.com/user-attachments/assets/ad4a9a55-bb3e-4b24-b4d2-c0fe8742d06a width="30%">

- For size = 30 <br>
<img src=https://github.com/user-attachments/assets/5d811830-2ec1-4780-9464-d3acf124f410 width="30%">
<img src=https://github.com/user-attachments/assets/daa5ace8-a7b4-472d-a03e-195548adfb35 width="30%">

- For size = 50 <br>
<img src=https://github.com/user-attachments/assets/f81f9b6f-038e-4f95-aebb-622b7d7a9b4a width="30%">
<img src=https://github.com/user-attachments/assets/f0e69751-62aa-4e77-8d7d-df2dedc5f31c width="30%">



## Second phase 
For this phase, the index from the first phase was expanded by adding the synonym graph filter to the elasticSearch analyzer. Two scenarios were chosen to compare and contrast: 
expanding the index with synonyms for the nouns (index _nouns_) and expanding the index with synonyms for the nouns and the verbs (index _nv_). These two scenarios (_nouns_ and _nv_) were chosen based on the idea that each scenario would involve some form of scaling (from no synonyms to synonyms for nouns, and then to synonyms for both nouns and verbs), in order to observe whether each extension led to improved results, rather than simply comparing two unrelated scenarios.

### Comparisons to previous phase organized by metric: 
- True Positive documents retrieved <br>
<img src=https://github.com/user-attachments/assets/1b2cec1c-f01f-4e9a-ac14-d9900b303f2d width="70%">

- MAP <br>
<img src=https://github.com/user-attachments/assets/88a46c84-614b-41b8-8460-c21f7e680928 width="70%">

- gmMAP <br>
<img src=https://github.com/user-attachments/assets/772c81d5-61bc-476d-aafb-7672bbea4088 width="70%">

- P5  <br>
<img src=https://github.com/user-attachments/assets/1c414132-44f4-4554-8ad2-37e3d72a8b34 width="70%">

- P10 <br>
<img src=https://github.com/user-attachments/assets/4490c1a4-4b42-4439-b5c1-0a9c34b59cab width="70%">

- P15 <br>
<img src=https://github.com/user-attachments/assets/3cd69ca4-400e-400c-9a43-5c588c6cd798 width="70%">

- P20 <br>
<img src=https://github.com/user-attachments/assets/e1b465a3-691f-4d18-a451-0b00da6fbf4c width="70%">

### Observations and conclusions 
Regarding the retrieval of relevant documents, it is clear that regardless of the value of k (the number of retrieved documents – 20, 30, or 50), the _nouns_ scenario significantly increased the number of relevant documents (about 180 more relevant documents), while the _nv_ scenario also increased them, but to a much lesser extent (about 25 more relevant documents). This shows that the initial extension (using noun synonyms) substantially improves retrieval, while further additions (including verb synonyms) bring smaller improvements.

This pattern appears across all metrics, with the _nouns_ scenario providing a significant improvement, and the _nv_ scenario offering additional but clearly smaller gains. However, in every case, the best overall results are achieved with nv — the scenario with the most extensive synonym expansion.

Furthermore, as expected, increasing k also increases the number of retrieved relevant documents (in all three scenarios, for k=50, approximately 500 more relevant documents were found compared to k=20). However, as k increases, the rate of increase in relevant documents decreases. For example, for k=20, the increases between scenarios were 223 and 37 extra documents, while for k=50, the increases were 153 and then only 23 extra documents. This confirms that improvements in search tend to affect the top-ranked documents more than those retrieved later. This pattern is consistent across all metrics: the larger the value of k, the smaller the improvements.

The MAP (Mean Average Precision) has its highest value at k=20 and using the _nv_ index, indicating that most relevant documents appear high in the ranked list. In contrast, when more documents are retrieved, they tend to be less relevant. The previously mentioned patterns also appear here: as k increases, changes become smaller, and the overall improvement from scenario to scenario diminishes.

As for gmMAP (geometric mean Average Precision), its significant improvement from the baseline index to _nouns_ across all k values shows how synonym expansion stabilizes retrieval and helps in answering more difficult queries, especially among the top-ranked results. For k=20, gmMAP nearly doubles, meaning that even for harder queries, relevant answers are now found with synonym use. The best results in this case are with _nv_, for k=50, indicating that the system’s overall performance improves, with relevant documents found even among a large number of retrieved texts.

The precision of the first retrieved documents is best in the top 5 results for the _nv_ scenario, showing that the best retrieval occurs when both _nouns_ and verbs are included in the expansion. It also shows that the most relevant documents are among the first retrieved, while precision steadily drops for the next sets (top 10, 15, 20 documents). Additionally, precision decreases as the number of retrieved documents (k) increases — in all tables and for all scenarios, precision is highest at k=20.

Finally, it is noted that from phase 1 to phase 2, the results have improved significantly, which is expected since in phase 2, beyond word-for-word matching, conceptual search using synonyms is also performed.


## Third phase 
In the final phase, the word2vec model was integrated to the code of the second phase. To finetune the model, some experiments were performed against the results brought from the initial un-finetuned version of the retrieval with word2vec (index is named w2v). The results of all the experiments that did not improve the metrics are included [here](./results/phase3) and not in this report to avoid unnecessary information crowding

First, considering that not all fields are equally important, the idea to expand only some of them to reduce noise was considered. The [results of only expanding title and text](./results/phase3/titleTextExpanded.txt) were the relative best, but without really affecting the results or runtime in a significant way, so the decision to keep them all was made. 

Then, experiments concerning the training parameters of the model where performed, from which the best configuration of training parameters was determined as shown below. <br>
<img src=https://github.com/user-attachments/assets/01a98d96-7a44-401a-b4b0-eb3762689d0a width="40%"> <br>

This is a logical conclusion considering that the corpus is relatively small (~25.000 documents), so it makes sense that sg=0 (CBOW architecture, which works better with smaller text corpora) and that the generally smaller parameters (embedding size = 50, context window = 5) yield relatively better results than [higher training parameters](./results/phase3/highTrainingParams.txt).

Then, experiments with the weights of the traditional retrieval versus word2vec were performed to determine which of the two ways should affect the results the most. The conclusion drawn was that no weights is the best solution (with the [results for the best weight configuration](./results/phase3/withWeights.txt) being a little worse that the unweighted version).

Finally, a [standard preprocessing](./results/phase3/preprocessing.txt) of the documents was tried, which also did not improve the existing best-case scenario (simple hybrid retrieval), so the "winner" of phase 3 (which will also be compared with the results of the previous phases) is the original simple hybrid retrieval. 

### Comparisons to previous phases organized by metric: 
- True Positive documents retrieved <br>
<img src=https://github.com/user-attachments/assets/a224595b-8213-4a51-a0bc-10f475a08fa9 width="70%">

- MAP <br>
<img src=https://github.com/user-attachments/assets/acffeb39-b6a6-4923-a004-33eb70b69ea6 width="70%">

- gmMAP <br>
<img src=https://github.com/user-attachments/assets/edd41b53-4164-4c7d-a81d-b907f059eaf8 width="70%">

- P5  <br>
<img src=https://github.com/user-attachments/assets/63b1b965-1b72-4a42-89b3-645dfb7489c1 width="70%">

- P10 <br>
<img src=https://github.com/user-attachments/assets/1cc0cf64-4078-4d55-932d-992a53df7c1a width="70%">

- P15 <br>
<img src=https://github.com/user-attachments/assets/6ab94a72-f8d2-458e-b62c-04f89da46e84 width="70%">

- P20 <br>
<img src=https://github.com/user-attachments/assets/ae6aa61b-0c88-40a8-aecf-0ae7c35ea01f width="70%">

### Observations and conclusions 
The main conclusion that can be drawn is that the addition of Word2Vec leads to noticeably worse results across all metrics. These poorer results were expected because the size of the corpus (~25,000 documents) is too small for the effective training of a model using Word2Vec. As a result, the model is undertrained and cannot produce accurate outputs, thus lowering the Precision.

Moreover, the corpus contains scientific texts, which likely include specialized terms and terminology—words that the model is not equipped to handle, or for which the usual semantic processing does not apply. This causes the model’s predictions to be even more off target.

This factor, combined with the very limited training data, clearly explains why the results in phase three are worse than those in phases one and two.

As for the results depending on the number of documents returned by the search, the same patterns seen in previous phases continue to apply:
- num_rel_ret (number of relevant retrieved documents) increases as k increases
- while MAP and P@5, P@10, P@15, P@20 decrease — though now with much lower values.

Therefore, the main takeaway remains that adding Word2Vec to a small, domain-specific corpus (with many specialized terms) does not improve retrieval — in fact, it significantly hinders it.
