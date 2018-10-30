---
title: Coursera Kaggle 강의(How to win a data science competition) week 4-1 Hyperparameter Tuning 요약
date: 2018-10-30 15:08:00
categories:
- DataScience
tags:
- DataScience
- MachineLearning
- Kaggle
---

# Coursera Kaggle 강의(How to win a data science competition) week 4-1 Hyperparameter Tuning 요약

## Hyperparameter Tuning

- 어떤 hyperparameter가 모델에 가장 큰 영향을 미치는지 알아야 한다.
- 모든 hyperparameter를 튜닝할 시간이 없다.

- 수동 : 변경 후 결과 측정
- 자동화된 라이브러리
  - Hyperopt
  - Scikit-optimize
  - Spearmint
  - GpyOpt
  - RobO
  - SMAC3

일반적으로 수동으로 하는것이 더 빠르다.

Hyperparameter 중 해당 값이 커질수록 underfitting 되는 것도 있고 (red), 해당 값이 커질수록 overfitting 되는 것도 있다. (green)

## 1. Tree-based models

XGBoost, LightGBM, CatBoost, sklearn의 RandomForest 및 Extra Trees models, Regularized Greedy Forest 등...

### 1.1 Gradient Boosting

트리를 만드는 매개변수

- max_depth (XGBoost, LightGBM) (green) : tree의 최대 깊이. 클수록 fit이 더 빠르게 됨. 하지만 학습 시간이 굉장히 오래 걸리게됨. 통상적으로 7로 시작하는게 좋음.

- num_leaves (LightGBM) (green): 트리의 리프수를 제어

- subsample (XGBoost), bagging_fraction (LightGBM) (green): 0 ~ 1
- colsample_bytree, colsample_bylevel (XGBoost), feature_fraction (LightGBM) (green) : tree 분할시 일부만을 사용

- min_child_weight (XGBoost), min_data_in_leaf (LightGBM) / lambda, alpha (XGBoost) , lambda_l1, lambda_l2 (LightGBM) (red): 정규화 매개변수, min_child_weight가 가장 중요. 최소값은 0 (가장 보수적 모델). 작업에 따라 최적 값은 0, 5, 15, 300 되르모 넓은 값을 사용하는 것에 주저하지 마라.

학습관련 매개변수

- eta (XGBoost), learning_rate (LightGBM) (green): 학습 가중치. 너무 낮으면 수렴하지 않을 수 있음.
- num_round (XGBoost), num_iterations (LightGBM) (green): 학습 회수

random seed를 바꾸는 방법도 있지만, 일반적으로 큰 차이가 없다.

### 1.2 Random Forest, Extra Trees

Extra Trees는 Random Forest의 무작위 버전이며 매개변수가 동일

- n_estimators : 트리 수, 매루 작은 수(10)로 설정하여 시간이 얼마나 걸리는지 본 다음 300 정도의 큰 값으로 설정하는 식으로 접근. 어느 시점 이후에는 그 변화가 미비할 수 있으므로 적당값을 찾는게 좋기는 하지만, 클 수록 좋음.
- max_depth (green) : 트리의 깊이. 7로 시작하는게 좋으나 일반적으로 Random Forest 의 최적값은 Gradient Boosting보다 높으므로 주저없이 높게 설정해라. None으로 설정하면 깊이가 무제한으로 생성된다.
- max_features (green) : subsample(XGBoost) 과 비슷함.
- min_samples_leaf (red) : min_child_weight (XGBoost)와 비슷

Random Forest Classifier의 경우 트리 분할을 향상시키는 기준으로 Gini 와 Entropy를 사용하는데, 둘 다 시도해서 성능이 좋은 것을 선택해야 한다. 자주 Gini가 더 좋지만, Entropy가 더 좋은 경우도 있다. 

- random_state : 랜덤 시드
- n_jobs : 보유한 코어 수로 설정

## 2. Neural Nets

Keras, PyTouch, Tensorflow, MxNet, ...

- layer 의 뉴론 수 (green)
- layer 수 (green)
- Optimizers
  - SGD + momentum (red)
  - Adam / Adamdelta / Adagrad / ... (green)
- Batch size (green)
- Learning rate
- Regularization
  - L1 / L2 for weights (red)
  - Dropout / Dropconnect (red)
  - Static dropconnect (red)
