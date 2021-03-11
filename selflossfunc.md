---
title: "LightGBM和XGBoost Custom Loss Function"
author: "Kai Zheng"
tags: ["loss"]
date: 2021-03-10T11:15:31+08:00
draft: true
---

在LightGBM和XGBoost中的自定义损失函数（Custom Loss Function）。
<!--more-->

Training loss and Validation loss
Training loss: This is the function that is optimized on the training data. For example, in a neural network binary classifier, this is usually the binary cross entropy. For the random forest classifier, this is the Gini impurity. The training loss is often called the “objective function” as well. The training loss in LightGBM is called objective.

Validation loss: This is the function that we use to evaluate the performance of our trained model on unseen data. This is often not the same as the training loss. For example, in the case of a classifier, this is often the area under the curve of the receiver operating characteristic (ROC) — though this is never directly optimized, because it is not differentiable. This is often called the “performance or evaluation metric”. The validation loss is often used to tune hyper-parameters. It is often easier to customize, as it doesn’t have as many functional requirements like the training loss does. The validation loss can be non-convex, non-differentiable, and discontinuous. The validation loss in LightGBM is called metric.

我们可以用Validation loss做early stopping：当迭代次数（boosting rounds，树的数量）增加的时候，loss经过early_stopping_rounds不减小，则停止训练。

但是如果Validation loss function是二阶可导的，则可以考虑直接用其作为Training loss直接优化模型。

Training loss: Customizing the training loss in LightGBM requires defining a function that takes in two arrays, the targets and their predictions. In turn, the function should return two arrays of the gradient and hessian of each observation. As noted above, we need to use calculus to derive gradient and hessian and then implement it in Python.

Validation loss: Customizing the validation loss in LightGBM requires defining a function that takes in the same two arrays, but returns three values: a string with name of metric to print, the loss itself, and a boolean about whether higher is better.

官方例子-LGBM中自定义log likelihood loss：

self-defined objective function
f(preds: array, train_data: Dataset) -> grad: array, hess: array
log likelihood loss
def loglikelihood(preds, train_data):
    labels = train_data.get_label()
    preds = 1. / (1. + np.exp(-preds))
    grad = preds - labels
    hess = preds * (1. - preds)
    return grad, hess

self-defined eval metric
f(preds: array, train_data: Dataset) -> name: string, eval_result: float, is_higher_better: bool
binary error
def binary_error(preds, train_data):
    labels = train_data.get_label()
    return 'error', np.mean(labels != (preds > 0.5)), False
实例
（1）自定义MSE
考虑这样一种场景，我们赶车赶飞机，预测我们的出发时间，使得我们等候时间最少。对于早到和晚到，惩罚是不一样的，早到机场火车站，无可厚非。但是要是晚到，就麻烦了… 所以显而易见，我们在建模时候需要加大迟到的惩罚。如下customMSE公式，对于迟到我们加大10倍的惩罚。

