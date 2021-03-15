# Intelligent Architectures


## 5-2-2021

Loss functions:

* SVM Multiclass Loss (Hinge Loss)
  * SVM: Support Vector Machine
  * 
* Softmax Regression

Backpropagation

## 9-2-2021

Roofline plot: at low operational intensity you are bound on (memory) bandwidth, at higher operational intensity you are bound by peak performance.

Regularization:

* Training: add some kind of randomness
* Testing: average out randomness

Possible regularization examples: dropout, data augmentation, DropConnect, fractional pooling, stochastic depth (skip some layers during training), cutout (set random image regions to zero), mixup (train on random blends of images), add random noise.

Dropout: disabling random neurons (?)

Data augmentation: flips, random crops and scales, randomize contrast and brightness.

### Hyperparameters

Model design + hyperparameters -> model parameters

* Model design: # layers, activations, optimizers.
* Hyperparameters: learning rate, dropout
* Model parameters: weights

Choosing hyperparameters: try to train to 100% accuracy on a small sample (overfitting).

1. Check initial loss
2. Overfit a small sample
3. Find learning rate that makes loss go down
4. Coarse grid, train for ~1-5 epochs
5. Refine grid, train longer
6. Look at loss curves
7. Go to step 5

### Loss curve analysis

Loss plateaus: try learning rate step decay

Train accuracy increases but validation accuracy decreases: indication of overfitting! Increase regularization, get more data.

Train and val follow each other closely: means underfitting, train longer, use a bigger model.

### Data preprocessing

Normalization: less sensitive to small changes in weights

In practice for images:

* Subtract the mean image (e.g. AlexNet)

### Weight initialization

* Small random numbers, works for small networks, problems for deep networks.
* "Xavier" initialization: widely used, std=1/sqrt(Din). Activations are nicely scaled for all layers. For conv lyers, Din is kernel_size^2*input_channels.

### Batch normalization

..

### Deep learning software tools

Inference: for deployment

### Network design

1. Understand problem
2. ...

### Design space

Common tradeoffs:

* Accuracy / memory use
* Accuracy / latency
* Accuracy / energy consumption
* Energy consumption / speed

### Network optimization techniques

Pruning and quantization will be used in the assignment.

* Pruning
  * Removal of nodes, connections or kernels (layer?)
  * Can be part of the training
* Quantization
  * Reduce precision of stored data and operators
  * Reduces memory use
  * Compresses network, exploits redundancy
  * *Training may require full precision*
* Weight scaling
* Tensor decomposition

## 12-2-2021


### Pruning

Scenario 1

* Naive pruning: remove weights close to zero
* Parameter pruning based on weight similarity, if two neurons have similar entry weights, you could combine them and sum the 2 output weights.
  * AlexNet: stopping point at around 35% parameters pruned away on layers 6 and 7, more than that reduces the accuracy too much.

Scenario 2: if you have data

* L1: 25% of original size with -0.7% accuracy. (58MB instead of ~200MB).
* Iterative pruning and retraining: lot of effort, but results are good. With L2 regularization you can prune 90% without loss of accuracy.
  1. Prune weights based on magnitudes less than threshold, you get more errors.
  2. Train network until reasonable solution, trying to get rid of the new errors.
  3. Iterate to step 1

Weights distribution moves from a Gaussion distribution around 0 to a reduction around 0.

### Computation with accelerators

Pruning creates sparsity in matrices, however: specialized matrix multiplication hardware (tensor cores) have disadvantage for sparse matrices.

1. The compute load reduction of sparse must be large to capture.
2. Need to increase HW capabilities for sparse.

ReLU: rectified linear units


Hoyer regularization: combination of L1 and L2.

### Implementation: compression

Compressed Sparse Row (CSR):

* Offset array, e.g. 1, 5, 6
* Value array, e.g. 1.3, 0.2, 3.9
* Length scalar, e.g. 3

### From fine to coarse grained pruning

Instead of pruning cells in the matrix, prune out rows (vector-level) or kernels or filters.

### Pruning summary

* Data-free pruning: up to 1.5x reduction
* L1 regularization: 4x
* Retraining up to 6.7x
* Iterative retraining: 10x

Speed and energy advantages are less (3x-6x)

Use a model for sparsity

### Conclusions

Pruning brings a lot, but is not the only tool and has disadvantages (irregularity). It does not work well for all networks and layers.

Correct network design is also very important



## 23-2-2021: Reducing energy consumption: quantization


Quantization

* Floats are expensive, but their relative precision is constant.
* Integers are cheap but their relative precision fluctuates. (1.5->2 is much bigger error than 125.5->126)

Fixed-point number format: integer length + fractional length.

Quantization schemes:

* Linear (uniform) quantization: implies constant step size
  * Symmetric mode: symmetric around 0, asymmetric mode for when values are not distributed around 0.
* Non-linear (non-uniform): step size variable but following a regular pattern.
  * E.g. powers of 2, then smaller values should be more accurately quantized than larger values.
* Codebook: useful when values are not normally distributed.
  * Quantized values (codewords) are represented by a codebook.
  * Codewords are usually selected using a clustering or training, e.g. K-means clustering.


Bit width selection: we need to employ heuristics.


To watch; second half


## 26-2-2021:

Activation functions:

* Rectified linear unit (ReLU): y=max(0, x)

(Roughly off-chip memory requires around 200x more energy than on-chip.)

Data reuse: objective: keep data element uses *close in time*, that usually results in storing data in on chip storage. Ordering your computations is very important!

Usually convulation layers contribute to more than 90% of overall computation and memory accesses. However fully connected layers have a lot of weights which can lead to a lot of memory accesses.

Improvements: pruning, compact architectures, quantization. Typically lowers the accuracy a bit.
Loop optimization: software tricks to effectively use the memory hierarchy. Does not lower the accuracy.

### Data flow loop transformations

Loop-based strenght reduction: replace a multiplication in a loop with a summation, if possible, e.g. because the multiplication uses the iterator value.

Induction variable elimination: replace index calculations with the direct address and increment the address each iteration.

Loop-invariant code motion: extract computed values that are the same each iteration outside of the loop, if possible.

Loop unswitching: conditionals are slower, so preferably not inside an inner loop. If the conditional is not dependent on something inside the loop it can be moved outside.

### Reordering loop transformations

Normal compilers often don't help with this.

Re-use distance!

Loop tiling.

Halide: domain specific language


## 5-3-2021: Quantization to the extreme (binary)


A binary 1 denotes a real 1, a binary 0 denotes a real -1.

Binary multiply-accumulate: multiplication and addition are replaced by bitwise XNOR and PopCount. PopCount is #pos - #neg.

Why +1/-1 instead of +1/0? If you have +1/0 you get an AND, which has more information loss than an XNOR.


Ternary symbol(?)