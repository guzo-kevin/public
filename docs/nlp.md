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



