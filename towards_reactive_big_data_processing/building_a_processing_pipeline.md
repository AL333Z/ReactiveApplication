# Building a processing pipeline: a case study

The use case proposed to demostrate the application of RP to the event processing context is an example of sentiment analysis.

From the Wikipedia definition:

> Sentiment analysis (also known as opinion mining) refers to the use of natural language processing, text analysis and computational linguistics to identify and extract subjective information in source materials.

The requirements the the application needs to satisfy are the following:

- the application should monitor and elaborate the sentiment toward two popular brands: Apple and Google;
- the analisys should be performed starting from the information taken from Twitter, by reading and elaborating what people tweets;
- the computation should assign to each tweet a "static score", determined by the relevance of the tweet's author (determined by its number of followers and the number of retweets its tweet got);
- after that, the application should infer if the tweet express a positive or negative comment;
- every tweet should be computed and grouped to the respective brand, and the result should be accumulated, so the users can have an idea of which brand has a better sentiment.

**NB**: This approach to sentiment analysis is for sure oversimplified and not correct at all, but in this thesis author's hopinion what really matters about this case study is not the correctness of the analysis method, but the application of the abstractions introduced in this thesis to solve problems.
