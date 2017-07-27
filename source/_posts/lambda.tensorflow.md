---
title: Using Tensorflow Predict on AWS Lambda Function
date: 2017-07-18 16:37:00
categories:
- AWS
- Lambda
tags:
- AWS
- Lambda
- S3
- Python
- Tensorflow
- MachineLearning
---

# Using Tensorflow Predict on AWS Lambda Function

## 왜 이런 생각을 ???

앞서 **AWS ECS**에서 **Tensorflow**를 이용하여 학습을 진행하여 그 결과를 **S3**에 저장하는 방법에 대해서 글을 적었다.

[Deploy Tensorflow Docker Image to AWS ECS](https://github.com/DevStarSJ/Study/blob/master/Blog/Python/TensorFlow/ms/ecs.tensorflow.md)

학습 결과를 **S3**에 남긴 이유는 크게 2가지였다.

- 하나는 다음에 학습을 진행할때 해당 값을 이용하여 계속해서 학습을 진행 할 경우를 대비하자는 것이고,
- 다른 하나는 학습 결과 데이터를 이용해서 예측하는 서비스를 제공하는데 사용하려는 목적이다.

## Lambda에 Tensorflow 배포 ?

**Tensorflow**를 **AWS Lambda**에서 서비스 가능할까 ? 결론부터 말하자면 안된다.

- 맥북에서 **Tensorflow**용 예제 코드를 하나 작성 한 뒤 해당 폴더를 압축하니 용량이 *43 MB* 정도 되었다.

![](/images/tensorflow.deploy.png)

하지만 Lambda에 올려서 실행을 하니 matrix 연산에 사용하는 함수에서 오류가 발생한다. 내부적으로 사용하는 **numpy** 모듈이 해당 OS에서만 동작하도록 배포되는것 같다.

그래서 Docker Image에 **Tensorflow**를 설치해서 압축도 해보고 **EC2**를 Linux용으로 하나 생성하여서 그곳에 Python 3.6을 설치하고 예제코드를 작성한 뒤 해당 폴더를 압축해봤는데 두 경우 모두 용량이 *61 MB* 가 넘었다.

![](/images/tensorflow.linux.deploy.png)

**AWS Lambda**의 경우 코드를 *50 MB* 이하로만 대포가 가능하다.

결국 안된다는 말이다.

페이스북 친구분들의 답변을 보니 **Theano Backend** 나 **MXNet** 등은 **Lambda**에서 서비스가 가능하다는 답변이 있다.

![](/images/lambda.ml.png)

하지만 이번 포스팅의 목적은 `Tensorflow로 학습된 결과를 어떻게 Lambda를 이용해서 가볍게 서비스 할 것인가` 이다.

물론 ECS나 EC2를 이용해서 정적 서버로 Tensorflow를 서비스하는건 가능하다. 너무나도 쉽게 가능하다.

## 안된다면서 어떻게 ???

일단 **Lambda**에서 **Tensorflow**를 배포할 수가 없다.
하지만 이미 **Tensorflow**를 이용해서 학습을 하는 시스템을 구축해 둔 상황이다.

**Tensorflow**의 학습 원리를 잘 생각해보자. **Tensorflow**에 대해서가 아니라 일반적인 **Machine Learning** 관련 학습을 해보신 분들은 관련 수식들에 대해서 본적이 있을 것이다.

- 입력값 출력값의 개수에 따라서 **Matrix 연산**
- `Classify`의 경우에는 **sigmoid**, **softmax** 등의 수식이 필요
- `Deep Learning`의 경우 위 2가지 수식을 계속해서 반복하면 됨

위에 나열된 정도의 수식만 구현하면 예측이 가능하다.

**Optimize Algorithm** 이라던지, **Cost** 계산, **Gradient Descent**와 같이 미분 연산이 들어가는 복잡한 것은 예측할때는 필요가 없으며, 학습할때 필요한 연산들이다.

어차피 예측하는 것만 서비스로 제공할꺼라 관련 수식을 구현하는건 그리 어렵지 않다.

`밑바닥부터 시작하는...`이라는 책들에 관련 수식들이 다 나와 있으며, 김성훈 교수님 동영상에도 나온다.

일단 여기서 사용하기위한 연산에 대해서는 내가 작성해 놓은 코드도 있다.

<https://github.com/DevStarSJ/functional.py/blob/master/linear.py>

이걸 이용하면 아주 가벼운 코드로도 예측해주는 서비스 제공이 가능하다.

## Docker Image 수정

기존에 배포한 **Docker Image** 는 **Tensorflow** 학습 결과 파일을 그대로 **S3**에 저장한다. 예측 서비스를 제공할 **Lambda**에는 **Tensorflow**가 없으므로 해당 파일의 데이터를 활용하지 못한다. 그렇다면 학습 결과를 일반적인 형태(**JSON**이라던지... 아니면 **JSON**이라던지... 하이튼 **JSON**이라던지 !)로  **S3** 에 저장해주는 것으로 코드를 수정하자.

[Deploy Tensorflow Docker Image to AWS ECS](https://github.com/DevStarSJ/Study/blob/master/Blog/Python/TensorFlow/ms/ecs.tensorflow.md)

배포 과정은 위 포스팅에 다 정리해 두었다. 

#### run.py (Docker Image)
```Python
import tensorflow as tf
import numpy as np
import boto3
import datetime
import os
import json

SAVER_FOLDER = "./saver"
BUCKET = 'BUCKET NAME'
TRAIN_DATA = "data-04-zoo.csv"
RESULT_FILE = 'result.json'

for file in os.listdir(SAVER_FOLDER):
    os.remove(SAVER_FOLDER + "/" + file);

s3_client = boto3.client('s3')
s3_client.download_file(BUCKET, TRAIN_DATA, TRAIN_DATA)

xy = np.loadtxt(TRAIN_DATA, delimiter=',', dtype=np.float32)
x_data = xy[:,0:-1]
y_data = xy[:,[-1]]
nb_classes = 7

X = tf.placeholder(tf.float32, [None, 16])
Y = tf.placeholder(tf.int32, [None, 1])

Y_one_hot = tf.one_hot(Y, nb_classes)
Y_one_hot = tf.reshape(Y_one_hot, [-1, nb_classes])

W = tf.Variable(tf.random_normal([16, nb_classes]), name='weight')
b = tf.Variable(tf.random_normal([nb_classes]), name='bias')

logits = tf.matmul(X,W) + b
hypothesis = tf.nn.softmax(logits)
cost_i = tf.nn.softmax_cross_entropy_with_logits(logits=logits, labels=Y_one_hot)
cost = tf.reduce_mean(cost_i)
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.1).minimize(cost)


prediction = tf.argmax(hypothesis, 1)
correct_prediction = tf.equal(prediction, tf.argmax(Y_one_hot, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

saver = tf.train.Saver()

with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    
    for step in range(2001):
        sess.run(optimizer, feed_dict={X: x_data, Y: y_data})
        if step % 100 == 0:
            print(step, sess.run([cost, accuracy], feed_dict={X: x_data, Y: y_data}))

    saver.save(sess,"./saver/save.{}.ckpt".format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
    saver.save(sess,"./saver/save.last.ckpt")

    pred = sess.run(prediction, feed_dict={X: x_data})

    pw, pb = sess.run([W, b])
    result = {'W': pw.tolist(), 'b': pb.tolist()}
    print(result)

    with open(RESULT_FILE, 'w') as outfile:
        json.dump(result, outfile)

s3_client.upload_file(RESULT_FILE, 'dev-tensorflow-savedata', RESULT_FILE)

for file in os.listdir(SAVER_FOLDER):
    print(file)
    s3_client.upload_file(SAVER_FOLDER + "/" + file, 'dev-tensorflow-savedata', file)
```

`run.py` 파일을 위와 같이 수정한 뒤 Repository에 push만 새로 해주면 된다.
그럼 해당 Docker Image가 다음부터 실행될 때 새로 바뀐 이미지로 실행된다.

## Lambda Function 작성

아래 코드로 작성 후 Lambda에 배포한다.

#### index.py
```Python
import boto3
import json
import sys

import functional as F
import linear as L

BUCKET_NAME = 'BUCKET NAME'
KEY = 'result.json'
RESULT_FILE = '/tmp/' + KEY
S3 = boto3.resource('s3')

DEBUG = True
def debug_print(obj):
    if DEBUG:
        print(obj)
    

def get_predict_data():
    try:
        S3.Bucket(BUCKET_NAME).download_file(KEY, RESULT_FILE)
        with open(RESULT_FILE) as data_file:    
            return json.load(data_file)
    except:
        print(sys.exc_info()[0])
        return None
    return None

def get_x(event):
    get_querystring= F.curryr(F.get)('queryStringParameters')
    get_X = F.curryr(F.get)('X')

    return F.go(event, 
        get_querystring, 
        get_X,
        lambda x: json.loads(x),
        lambda x: [x])

def handler(event, context):
    debug_print(event)

    predict_data = get_predict_data()
    debug_print(predict_data)
    if predict_data is None:
        return {'statusCode': 500}

    X = get_x(event)
    if X is None:
        return {'statusCode': 404}

    W = predict_data['W']
    b = predict_data['b']

    H = L.matadd(L.matmul(X,W)[0],b)
    S = [L.sigmoid(x) for x in H]
    M = L.softmax(S)
    answer = L.argmax(M)

    debug_print([answer, M])
    debug_print(sum(M))

    body = { 'answer': answer, 'rating': M }

    response = {
        'statusCode': 200,
        'body': json.dumps(body)
    }

    return response

if __name__ == "__main__":
    event = {"queryStringParameters": {"X": "[0,0,1,0,0,1,1,1,1,0,0,1,0,1,0,0]"}}
    print(handler(event,None))
```

`function.py`, `linear.py` 파일은 <https://github.com/DevStarSJ/functional.py>에서 다운로드가 가능핟. 3개의 파일을 압축하여 `Lambda Function`으로 배포 후 `API Gateway`에 연결하면 해당 url을 API로 호출하는것 만으로 서비스 제공이 가능해진다.

`https://(API Gateway URL)?X=[0,0,0,0,0,1,1,1,1,0,1,1,1,1,1,0]`

```JSON
{
    "answer": 2,
     "rating": [0.19675508709346667, 0.08691888941180405, 0.20361211198193085,
                0.16208064446318374, 0.12462949929899679, 0.07610383897536456, 
                0.14989992877525316
               ]
}
```

**Lambda** 및 **API Gateway** 배포에 대해서는 예전에 포스팅 한 글들을 참조 바란다.

- [Lambda 와 API Gateway 연동 #1 (GET, POST)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateWay.01.md)
- [Lambda 와 API Gateway 연동 #2 (ANY, Deploy Staging, Node.JS Route)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateway.02.Route.md)
- [Lambda 와 API Gateway 연동 #3 (Proxy Resource)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateway.03.Proxy.md)
- [Lambda Node.JS Packaging](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda.Packaging.Node.md)
- [AWS Lambda에 Python Handler 만들기](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda.Python.md)
- [Lambda Python Packaging](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda.Packaging.Python.md)

## Tensorflow 학습 코드

참고로 이 코드는 [김성훈 교수님의 모두를 위한 딥러닝 강좌](https://www.youtube.com/watch?v=BS6O0zOGX4E&list=PLlMkM4tgfjnLSOjrEJN31gZATbcj_MpUm) 에서 소개된 코드를 이용하였다.

동물의 여러가지 특징들에 대해서 입력받아서 이 동물의 종류가 무엇인가에 대한 학습데이터이다.

<https://archive.ics.uci.edu/ml/machine-learning-databases/zoo> 에서 해당 데이터 내용에 대해서 확인이 가능하다.

여기에는 김성훈 교수님이 미리 만들어 놓은 [data-04-zoo.csv](https://github.com/hunkim/DeepLearningZeroToAll/blob/master/data-04-zoo.csv) 파일을 이용하겠다.

- `data-04-zoo.csv`를 S3에 업로드
- 코드를 실행할 위치에 `saver`라는 폴더 생성 (`mkdir saver`)  

