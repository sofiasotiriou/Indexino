# Indexino

##### ElasticSearch engine implemented in python with different techniques for optimization compared

## Project overview 
This project is divided into three distinct phases: 
- Phase 1: _Basic search_. The index matches a field in the query with the corresponding field in every text in the index.
- Phase 2: _Synonym expansion_. Expansion to the phase 1 index with the use of the synonym_graph_filter in the ElasticSearch analyzer.
- Phase 3: _word2vec expansion_. Expansion to the phase 2 index with the use of the Word2Vec model.

## Code structure 
All three phases share a similar structure: 
- Installation and import of necessary modules. 
- Connection to ElasticSearch. 
- Import, analysis and formatting of all documents that go into the index. 
- Creation/deletion (if necessary) of the index. 
- Import, analysis and formatting of the queries. 
- Execution of the queries and formatting of results in a file readable by trec_eval. 

Obviously, the main parts that change from phase to phase are the creation of the index and the execution of the queries. 
The final results as well as the results of selected experiments can be found in [results](./results).
The experiments conducted, the results compared and the conclusions drawn can be found in [findings](./findings.md).
