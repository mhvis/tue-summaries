# Machine Learning Engineering

**Bold** variable is a vector!

## 2b

Loss function: how good is the solution?

We want to learn weights according to inputs and a loss function.

Linear models: we have weights/coefficients and a bias/intercept. We assume that errors are normally distributed around the line.
To calculate loss, take the sum of all squared errors/residuals (squared so that the error is always positive).

Solving ordinary least squares:

* Convex
* Gradient descent
* Very easily overfits

Learning rate with decay rate: e.g. exponential decay, inverse-time

Intuition: in neural networks (with lots of weights/dimensions) you will get stuck in a local minimum, however that will usually be a good local minimum.

Stochastic Gradient Descent (SGD): instead of computing gradient on the entire dataset, we use a single data point at a time. Minibatch SGD: in between, compute gradient on batches of data.

In practice: dataset with small number of features, use `LinearRegression`, else use gradient descent with `SGDRegressor` and `loss='squared_loss'`.

L2 regularization/shrinkage (Ridge): one way to reduce overfitting: add component to the loss function that requires that the weights are small, i.e. penalize in the loss function for large weights. We take the sum of the squared weights. Strength of regularization can be controlled with alpha hyperparameter, defaults to 1.0.

Can be optimized similarly:

* Closed form solution, a.k.a. Cholesky
* Gradient descent and variants, most popular ones are conjugate gradient (CG) or stochastic average gradient (SAG/SAGA)
* Use Cholesky for smaller datasets, gradient descent for larger

In practice: `Ridge` 

Other ways to reduce overfitting:

* Add more training data
* Use fewer features
* Scale the data typically helps, usually doesn't hurt

L1 regularization (Lasso): use sum of absolute values of the weights (instead of square in L2). Lasso: least absolute shrinkage and selection operator. Not differentiable due to the absolute value. No closed-form solution. L1 prefers coefficients to be exactly 0, i.e. those features are completely removed from the model.

Coordinate descent: updates 1 weight at a time. Will find the optimal solution of that weight (only for that weight).

Elastic-Net: adds both L1 and L2 regularization, using an additional parameter rho to specifiy the L1/L2 ratio. rho=1 gives L_elastic = L_lasso, rho=0 gives L_elastic = L_ridge.

Other loss functions beside Least Squared:

* Huber, more robust against outliers
* Epsilon insensitive: finds a function with no error larger than epsilon, a.k.a. support vector regression (SVR in sklearn).
* Squared epsilon insensitive
  
All these loss functions can be solved using `SGDRegressor` in sklearn.

## 2c

...

SVM in `scikit-learn`:

* `svm.LinearSVC`: faster for large datasets
* `svm.SVM` with `kernel=linear` allows kernelization
* `svm.LinearSVR` and `svmSVR` are variants for regression (epsilon loss)

`SGDClassifier`

### 3

Kernelization

Kernels are sometimes called generalized dot products. They compute similarity in high-dimensional spaces.

* Linear kernel
* Radial basis function (RBF) kernel
  * Value goes up for closer points and wider kernels (gamma)
  * Gamma: high values cause narrow kernels, overfitting, low vaalues cause wide kernels, underfitting.
  * C: cost of margin violations: high values cause narrow margins, overfitting, low values cause wider margins, underfitting.

In practice:

* C and gamma always need to be tuned, find a good C then tune gamma.
* SVMs expect all features to be approximately on the same scale
* In sklearn, `SVC` for classification with a range of kernels, `SVR` for regression.

For fitting regression data with a kernel: `KernelRidge`


## 22-2-2021: 4 Model selection

K-fold cross-validation: variance is also important to see whether the data is 'balanced'. K=5 is good, k=10 is better.

Stratify

Leave-One-Out cross-validation: k-fold to the extreme, the test set consists of only 1 sample.
Risk of overfitting. Only for small <100 data sets.

OOB: out-of-bootstrap, unsampled samples

'Holy grail': repeated cross-validation, a.k.a. n-times-k-fold. Shuffle data randomly, do k-fold CV, repeat n times. k=5, n=2 seems to be a reasonable tradeoff.

CV with groups: make sure that data from one person is in *either* the thrain *or* test set. Called grouping/blocking. Leave-one-subject-out CV.


### Binary classification

False positive: type 1 error
False negative: type 2 error

Confusion matrix: actual in the rows, predicted in the columns.

Accuracy: number of correct predictions over total number of predictions. Not useful if the dataset is imbalanced, e.g. 'credit card fraud: is 99.99% good enough?'.

Precision: use when you want to limit false positives. Precision=TP/(TP+FP).

Recall (a.k.a. sensitivity, hit rate, true positive rate (TPR)): use this to limit false negatives. Recall=TP/(TP+FN).

F1-score: combination of precision and recall: F1 = 2 * ((precision*recall) / (precision+recall))

Multi-class classification: divide the classes into separate models with binary classification, such that each model has a binary classification of whether it is in a certain class. Then average: micro-averaging (every sample equally important), macro-averaging (use this when classes are imbalanced), weighted averaging (weight score by how many samples there are in each class)

## 24-2-2021

Missed lecture


## 1-3-2021

Assuming separate training and test set: if some models perform worse than other models, those models are underfitting or overfitting more. It's overfitting if the training set error is very high and test set error is much lower (?).

### Regression metrics

* Mean squared error: sum_{test set points} (y_pred - y_actual)^2
  * Root mean squared error (RMSE) often used as well, take root over the final result.
* Mean absolute error: less sensitive to outliers.

R squared: ratio between the model and just predicting the mean. Between 0 and 1, but negative if the model is worse than just predicting the mean.

Visualizing regression errors:

* Prediction plot: predicted vs actual target values
* Residual plot: plot residual vs target value (0)


Bias and variance

* High variance means you're likely overfitting
  * Use more regularization or use a simpler model
* High bias means you're likely underfitting
  * Do less regularization or use a more flexible/complex model
* Ensembling techniques reduces bias or variance directly
  * Bagging (e.g. RandomForests) reduces variance, Boosting reduces bias

### Tuning hyperparameters

* Grid search: choose a range of values for every hyperparameter
  * Doesn't scale to many hyperparameters
* Random search: choose random values
  * In practice much better than grid search


## 8-3-2021

(Incomplete)

Feature engineering:

* Polynomials: add polynomials up to a degree *d*
* Binning: split a feature into multiple bins, e.g. if you have age you could split young and old people in separate bins.
* Categorization

Missing value imputation (?):

* Missing completely at random (MCAR)
* Missing at random (MAR): no relation with the value, e.g. faulty sensors, some people don't fill out forms correctly
* Missing not at random (MNAR): linked to the value, has to be resolved



## 15-3-2021: 8. Neural Networks


Sigmoid function: used to map from [-inf, +inf] to [0,1].

Neuron: a.k.a. node, unit, cell.

Per layer in the network we have a weight matrix with w_{i,j} the weight between node i and j.

