# Text Classification
https://learning.oreilly.com/library/view/text-analytics-with/9781484243541/html/427287_2_En_5_Chapter.xhtml


* Supervised vs unsupervised
* Classification vs regression
* Binary classification vs Multi-class classification vs Multi-lable classification


## blue print

* prep training and test dataset
* preprocess and normalize text documents
* Feature extraction and engineering
* Model training
* Model prediction and evaluations
* Model deployment


# Evaluations

https://heartbeat.fritz.ai/introduction-to-machine-learning-model-evaluation-fa859e1b2d7f

* Holdout

test a model on different data than it was trained on. Divide data into Training set, Validation set, and Test set

* Cross-validation

k-fold cross-validation. Divide data into k partitions (5, or 10 usually), use k as test and remaining as training set. so each data set is used as Test data for onces, and as training data k-1 times. 

### Evaluations for Classifications

* Classification Accuracy

number of correct predictions vs all predictions mode. 

  sklearn.metrics.accuracy_score()



* Confusion Matrix
  an matrix of predicted vs true label. (useful for multi label classifications). The diagnal elements represent the number of points for which the predicted label is equal to the true label 
  ```
  [[14 0 0]
   [0 15 4]
   [0 0 18]]
  ```
  sklearn.metrics.confusion_matrix()

* Logarithmic Loss
  
  an probability value between 0 and 1, the smaller the logloss is better, with a perfect model having a log loss of 0

  sklearn.metrics.log_loss()

* Area under Curve (AUC) for binary classifier 

AUC score need to be closer to 1 and greater than 0.5. A perfect classifier will have ROC curve go along the Y axis and then along the X axis

  from sklearn.metrics import roc_auc_score, roc_curve

* F-Measure (F-score)

https://en.wikipedia.org/wiki/F1_score



F1 = (2/(recall^-1 + precision^-1))
Score considers both precision and recall. Precision is the number of correct positive results divided by the total predicted positive observations (true poistives and false positives). Recall, is the number of correct positive results divided by the number of all relevant samples (total actual positives).  Here "relevant samples" means all actual positives when we are talking positives. 

Note the importance of precision and recall is an aspect of the problem. 

* Regression Metrics (this is not for classifications)

Root Mean Squared Error (RMSE): 

Mean Absolute Error (MAE): 


# Further reading (from Data Science from Scratch,2nd edition, Chapter 21 and Text Analytics with Python:A Practitioner's Guide to Natural Language Processing, Chapter 2)

* NLTK is a popular library of NLP tools for Python.  It has its own entire book, which is available to read [online](http://www.nltk.org/book/).
   It contains over 50 corpora and lexical resources. It comes with a suite of efficient modles for classifications, tokenizations, stemming, lemmatization, tagging, parsing, and semantic reasoning. 

* [gensim](https://radimrehurek.com/gensim/) is a Python library for topic modeling, which is a better bet than our from-scratch model. It has work2Vec

* [spaCy](https://spacy.io/) is a library for “Industrial Strength Natural Language Processing in Python” and is also quite popular. work with  TensorFlow, PyTorch, Scikit-Learn, Gensim, etc. support for several langauges and Pretrained word vectors

* Andrej Karpathy has a famous blog post,  [“The Unreasonable Effectiveness of Recurrent Neural Networks”](http://karpathy.github.io/2015/05/21/rnn-effectiveness/), that’s very much worth reading.

* My day job involves building [AllenNLP](https://allennlp.org/), a Python library for doing NLP research. (At least, as of the time this book went to press, it did.) The library is quite beyond the scope of this book, but you might still find it interesting, and it has a cool interactive demo of many state-of-the-art NLP models.

# General Data Science bookmark

https://skymind.ai/wiki/word2vec

I searched Word2Vec and find this website. Seemed the first stop for most Data Science/AI topics

### Word2Vec

provided by gensim library

Word2Vec is a 2-layer neural net that processes text. Its input is a text corpus and its output is a set of vectors: feature, vectors for words in that corpus. 

It turns text into a numerical form that deep nets can understand.  Given enough data, usage and context, word2vec can make highly accurate guesses about a word's meaning, the guesses can be used to establish a word's association with other words. 