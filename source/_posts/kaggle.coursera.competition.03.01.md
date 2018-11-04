---
title: Coursera Kaggle 강의(How to win a data science competition) week 3-1 Metrics 요약
date: 2018-10-22 11:04:00
categories:
- DataScience
tags:
- DataScience
- MachineLearning
- Kaggle
---

# Coursera Kaggle 강의(How to win a data science competition) week 3-1 Metrics 요약

## Metrics

어떤 metric을 사용하느냐에 따라 모델이 학습하는 방향이 다르다. 우리가 풀고자하는 문제에 최적화된 metric을 선택하는 것이 중요하다.

### 1. Regression metrics

#### 1.1 MSE, RMSE, R-squared

- **MSE** (Mean Square Error) : target과 predict의 차이값의 제곱의 평균
  - `Best Constant` : target mean
  - `sklearn.metrics.mean_squared_error`

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.01.png)

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.02.png)

- **RMSE** (Root Mean Square Error): MSE에 root취한 값

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.03.png)

MSE의 최소값이 RMSE에서도 최소값이므로 최적화 결과가 같다. 그래서 비교적 구현이 간단한 MSE를 사용하는 경우가 많지만, **learning rate** 같은 몇몇의 hyperparameter 값에 따라 다르게 동작 할 수 있다.  
MSE와 RMSE 값이 32라고 성능이 좋은지 나쁜지 판단이 힘들다. 그래서 상대적인 값으로 평가가 필요할 수 있다.

- **R2** (R Squared)
  - `sklearn.metrics.r2_score`
  - P-Value와 같이 0 ~ 1 사이의 값을 나타내는데, MSE가 0 이면 R2는 1이며, MSE가 constant 모델보다 클 때 R2는 0이다.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.04.png)

#### 1.2 MAE, RMAE

- **MAE** (Mean Absolute Error): target과 predict의 차이 절대값
  - `Best Constant` : target median
  - `sklearn.metrics.mean_absolute_error`

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.05.png)

- **RMAE** (Root Mean Absolute Error): MAE에 root취한 값

#### 1.3 MSE와 MAE의 차이
- MSE의 경우 차이가 2배이면 error가 4배가 되는데, MAE는 차이가 2배이면 error도 2배  
- MSE와 RMSE는 최적해를 찾기위해 gradient하게 접근시 각 지점마다 기울기(미분값)이 다르지만, MAE는 왼쪽은 -1, 오른쪽은 1이다.
- MAE는 outlier에 덜 민감하게 동작한다. (MSE는 제곱을 해서 크게 민감하다.)
- outlier가 없다고 확신이 드는 경우에는 MSE가 더 좋은 경우가 많다.

#### 1.4 (R)MSPE (Mean Square Percent Error), (R)MAPE(Mean Absolute Percent Error) : relative_metric

MSE 와 MAE는 error를 절대적인 값으로 비교를 한다. 그래서 `9 -> 10 (MAE, MSE=1)` 와 `999 -> 1000 (MAE, MSE=1)`는 같은 양의 error로 계산된다.
하지만 `900 -> 1000 (MAE=100, MSE=10000)`는 error의 수치가 훨씬 커진다. 

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.06.png)

MSPE와 MAPE는 각각 MSE와 MAE에다가 전체 데이터 개수의 퍼센트로 계산한다.

`Best Constant` 역시 MSPE는 *weighted target mean*이며, MAPE는 *weighted target median* 값이다.  
앞에 Root가 붙은 버전에 대한 설명은 생략한다.

#### 1.5 RMSLE (Root Mean Square Logarithmic Error)

- `sklearn.metrics.mean_squared_log_error`

로그 스케일로 계산된 RMSE

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.07.png)

Error 곡선의 좌우가 대칭적이지 않다.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.08.png)

### 2. Classification metrics

#### 2.1 Accuracy
- `sklearn.metrics.accuracy_score`

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.09.png)

예측한 값과 target값이 같으면 1 아니면 0 으로 계산하여 평균을 취한 값  
개 그림 맞추기 문제에서 *개* 90, *고양이* 10으로 데이터셋이 있는 경우 모두 *개*라고 대답해도 Accuracy는 0.9가 나온다.

#### 2.2 Logarithmic loss (logloss)
- `sklearn.metrics.log_loss`

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.10.png)

target과 차이가 클수록 penalty가 커진다. 하나의 큰 error는 여러 개의 작은 error들보다 훨씬 더 penalty가 크다.

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.11.png)

#### 2.3 AUC (Area Under Curve)

얼마나 구분을 잘하느냐, 얼마나 겹치는게 없느냐에 대한 검증
좋은 피처인지 아닌지를 구분할때 많이 사용  

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.12.png)

`True Positive` 와  `False Positive`를 이용하여 `TP`는 위쪽 `FP`는 오른쪽으로 움직이여 곡선을 그림을 그려서 그 아래 면적으로 평가. 면적이 넓을 수록 좋음. 최고 점수는 1

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.13.png)

#### 2.4 Cohen's Kappa

어렵다. 무슨 내용인지 도저히 모르겠음

### 3. General approaches for metrics optimization

- Target metric : 우리가 최적화 하려는 것
- Optimization loss : 모델이 최적화 하는 것

모델은 우리가 지정한 *metric*으로 계산한 *loss*를 최소화하기 위해서 학습한다.

- metric 최적화 전략
  - 처음에 그냥 model을 실행할때 : MSE, Logloss
  - train 데이터 전처리하여 다른 metric에 최적화 할때 : MSPE, MAPE, RMSLE, ...
  - 다른 metric이나 predict 결과를 후처리 할때 : Accuracy, Kappa
  - 기타 : loss function를 스스로 작성
  - 다른 metric의 early stopping 용 : Any...

ex) 1,2차 미분값으로 loss를 계산하고자 할 때

```Python
 def logregobj(preds, dtrain):
     labels = dtrain.get_label()
     preds = 1. / (1. + np.exp(-preds))
     grad = preds - labels 
     hess = preds * (1. - preds)
     return grad, hess
```

- Early stopping
  - M1 metric을 최적화하기 위해서 M2 metric의 최적값을 구하는 경우

![](https://raw.githubusercontent.com/DevStarSJ/Study/master/Blog/Kaggle/Coursera.competition/image/coursera.competition.03.14.png)

### 4. Regression metrics optimization

#### 4.1 MSE, RMSE, R-squared

대부분의 모델에서 잘 동작한다. 
- Tree-based : XGBoost, LightGBM, sklearn.RandomForestRegressor
- Linear models : sklearn.<>Regression, sklearn.SGFRegressor, Vowpal Wabbit (quantile loss)
- Neural nets : Pytorchm Keras, TF, etc.

L2 loss라고도 불림

#### 4.2 MAE

MSE와 최적화 관점에서는 큰 차이가 없지만, 2차 미분이 0이기 때문에 extra boost 방식에서는 사용을 못한다.
- Tree-based : LightGBM, sklearn.RandomForestRegressor
- Linear models : Vowpal Wabbit (quantile loss)
- Neural nets : Pytorchm Keras, TF, etc.

L1 loss, Median regression 이라고도 불림

#### 4.3 MSPE, MAPE

이것은 앞에서 본 2가지 metric과는 용도가 다르다.  
각각 MSE, MAE의 가중 버전이다.  
다른 metric의 early stopping을 위해 사용되기도 한다.

- XGBoost, LightGBM 에서만 `sample_weights`용으로 사용한다.

#### 4.4 RMSLE

로그공간에서의 MSE 최적화 방식

### 5. Classification metrics optimization

#### 5.1 LogLoss (Logistic Loss)

- 모델이 잘 맞추는지를 측정
- Regression의 MAE와 같은 존재  
- RandomForest빼고는 대부분 잘 맞음  
  - Tree-based : XGBoost, LightGBM
  - Linear models : sklearn.<>Regression, sklearn.SGDRegressor, Vowpal Wabbit
  - Neural nets : Pytorchm Keras, TF, etc.

#### 5.2 Accuracy

- metric이나 treshold를 최적화하는데 유용함

#### 5.3 AUC

- 모델이 잘 맞추는것을 측정하거나 logloss를 최적화하기 위해 사용됨
- 한상의 object가 올바른 순서로 정렬될 확률
- Tree-based : XGBoost, LightGBM
- Neural nets : Pytorchm Keras, TF, etc.

#### 5.4 

- MSE를 최적화
- 올바른 threshold를 찾음
  - Bad : np.round(predictions)
