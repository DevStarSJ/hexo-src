---
title: Lambda Python Packaging
date: 2016-11-24 16:54:51
categories:
- AWS
- Lambda
tags:
- AWS
- Lambda
- APIGateway
- Python
---

# Lambda Python Packaging

**AWS Lambda**에 대해 다루는 6번째 글입니다.

- [Lambda 와 API Gateway 연동 #1 (GET, POST)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateWay.01.md)
- [Lambda 와 API Gateway 연동 #2 (ANY, Deploy Staging, Node.JS Route)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateway.02.Route.md)
- [Lambda 와 API Gateway 연동 #3 (Proxy Resource)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateway.03.Proxy.md)
- [Lambda Node.JS Packaging](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda.Packaging.Node.md)
- [AWS Lambda에 Python Handler 만들기](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda.Python.md)

지난번 글에서 1개의 **Python** 파일로 구현하여 **AWS Lambda**에 올리는 방법에 대해서 다뤘는데,
이번 글에서는 여러 개의 파일로 나뉘어서 구현한 경우와 외부 라이브러리를 **pip**로 설치할 경우 어떻게 해야하는지에 대해서 다뤄보겠습니다.


## Previously on Lambda series


**Lambda**의 생성 및 **API Gateway**와의 연결은 되어 있다는 가정하에 진행하겠습니다.
관련 내용들은 앞의 글들에 다 있지만 읽기 귀찮으시다는 분들을 위해 간략한 따라하기를 살없이 뼈만 추려서 먼저 소개하고 시작하겠습니다.


### Create Lambda

- `AWS` 로그인 후 `Lambda` 탭으로 이동
- `Create a Lambda Function` 선택
- Select blueprint에서 `Blank Function` 선택
- Configure function 에서 일단 바로 `Next` 선택 (미리 연결할 API Gateway가 있다면 여기서 연결하면 됨)
- `Configure function`에서 함수 정의
  - Name : `testLambda-luna`
  - Runtime : `Python 2.7`
  - Code : 아래 소개되어 있는 `Python Lambda Code`를 입력
  - Role & Existing role : 일단은 적당히 선택 (만약 Lambda에서 다른 AWS 서비스 RDS, S3 등을 연동할려면 필요)
  - 아래 코드 입력 후 `Next` 선택
- `Create Function` 선택

### Set API Gateway

- `AWS` 메인 화면으로 이동 후 `API Gateway` 탭으로 이동
- `Create API` 선택
  - API name : `testAPI-luna`
  - `Create API` 선택
- `/`에서 `Actions` -> `Create Resource`를 선택
  - `Configure as proxy resource` 를 체크한 후 `Create Resource`를 누름
- `ANY` 선택
  - `Integration Request` 선택
    - **Integration type** : `Lambda Function`
      - **Lambda Region** : 람다를 생성한 지역 서버 선택
      - **Lambda Function** : 람다 명칭 기입
- `Actions` -> `Deploy API` 선택 후 그냥 `prod`로 스테이징

### Python Lambda Code

```Python
import json

def get(event):
    user_id = event['queryStringParameters']['id']
    return { 'body': { 'id': user_id, 'name': "test" } }

def post(event):
    user_id = event['queryStringParameters']['id']
    body = event['body']
    header = event['headers']
    return { 'body': { 'id': user_id, 'header': header, 'body': body } }

route_map = {
    '/test': {
        'GET': get,
        'POST': post
    }
};

def router(event):
    controller = route_map[event['path']][event['httpMethod']];
    
    if not controller:
        return { 'body': { 'Error': "Invalid Path" } }
    
    return controller(event);

def handler(event, context):
    result = router(event);
    return { 'body' : json.dumps(result) }
```

## Python Code 패키징 연습

새로운 코드를 만드는것 보다는 일단 정상적으로 동작하는 것이 확인된 코드를 파일로 나눠가면서 진행하겠습니다.

### Step 1. 통파일을 그냥 .zip으로 압축하여 올리기

작업할 폴더를 하나 만듭니다. 일단 `packaging.test` 라는 이름으로 만들어 보겠습니다.

```
mkdir packaging.test
cd packaging.test
```

그 안에 원래 **Lambda**에 올려놓은 코드를 그대로 복사하여 `index.py`로 생성합니다.

#### index.py
```Python
import json

def get(event):
    user_id = event['queryStringParameters']['id']
    return { 'body': { 'id': user_id, 'name': "test" } }

def post(event):
    user_id = event['queryStringParameters']['id']
    body = event['body']
    header = event['headers']
    return { 'body': { 'id': user_id, 'header': header, 'body': body } }

route_map = {
    '/test': {
        'GET': get,
        'POST': post
    }
};

def router(event):
    controller = route_map[event['path']][event['httpMethod']];
    
    if not controller:
        return { 'body': { 'Error': "Invalid Path" } }
    
    return controller(event);

def handler(event, context):
    result = router(event);
    return { 'body' : json.dumps(result) }
```

`.zip`파일로 압축할 때 해당 `index.py` 파일이 루트에 위치해야 합니다.
즉, `packaging.test` 폴더를 압축하는게 아니라 그 안에 들어와서 압축을 해야 합니다.

```
zip sample.zip index.py
```
이제 **Lambda**에 올리신 후 테스트 해보면 됩니다.
올리는 방법과 테스트 방법은 계속 동일하기 때문에 여기서 한 번만 소개하고 밑에서는 따로 소개하지 않겠습니다.

먼저 해당 **Lambda** 설정으로 이동합니다.

- `Code` 탭
  -  Code entry type : `Upload a .ZIP file` 선택
    - `Upload` 버튼을 눌러서 위에서 생성한 `sample.zip`을 올림

- **GET** 요청 (브라우저에서 주소 입력) : `https://.../prod/test?id=2`

```JSON
{"body":{"id":"2","name":"test"}}
```

- **POST** 요청 (**POSTMAN**이나 **curl**등을 활용) : URL은 **GET**과 동일

```JSON
{
  "body": {
    "id": "2",
    "header": {
      ...
    },
    "body": "\"{\\n  \"id\": \"123\",\\n  \"age\": \"25\"\\n}\""
  }
}
```


### Step 2. 파일 나누기

위의 파일을 2개로 나누어서 올려보겠습니다.
같은 폴더에 `router.py`파일을 하나 생성 한 후 파일 내용을 아래와 같이 수정해 주세요.

#### index.py
```Python
import json
from router import router

def handler(event, context):
    result = router(event);
    return { 'body' : json.dumps(result) }
```

#### router.py
```Python
def get(event):
    user_id = event['queryStringParameters']['id']
    return { 'body': { 'id': user_id, 'name': "test" } }

def post(event):
    user_id = event['queryStringParameters']['id']
    body = event['body']
    header = event['headers']
    return { 'body': { 'id': user_id, 'header': header, 'body': body } }

route_map = {
    '/test': {
        'GET': get,
        'POST': post
    }
};

def router(event):
    controller = route_map[event['path']][event['httpMethod']];
    
    if not controller:
        return { 'body': { 'Error': "Invalid Path" } }
    
    return controller(event);
```

`.zip` 파일로 압축합니다.

```
zip sample.zip .
```

코드를 올린 후 테스트 했을 때 똑같은 결과가 나와야 합니다.

### Step 3. 폴더로 나누기

작업 중인 폴더에 `/controllers/test`의 2단계의 폴더를 추가 합니다.

```
mkdir controllers
cd controllers
mkdir test
```

그런 다음 2개의 폴더에 각각 `__init__.py`라는 파일을 만듭니다.
내용은 아무것도 없는 빈 파일로 생성합니다.
앞으로 작성할 2개의 파일도 미리 생성해 놓겠습니다.

```
touch __init__.py
cd test
touch __init__.py
touch post.py
touch get.py
```

`/controllers/test/`안에 `post.py`, `get.py`에 기존에 있던 `router.py`의 내용을 나눠서 수정합니다.

#### index.py
```Python
import json
from router import router

def handler(event, context):
    result = router(event);
    return { 'body' : json.dumps(result) }
```

#### router.py

```Python
import controllers.test.get
import controllers.test.post

route_map = {
    '/test': {
        'GET': controllers.test.get.handler,
        'POST': controllers.test.post.handler
    }
};

def router(event):
    controller = route_map[event['path']][event['httpMethod']];
    
    if not controller:
        return { 'body': { 'Error': "Invalid Path" } }
    
    return controller(event);
```

#### controllers/test/post.py

```Python
def handler(event):
    user_id = event['queryStringParameters']['id']
    body = event['body']
    header = event['headers']
    return { 'body': { 'id': user_id, 'header': header, 'body': body } }
```

#### controllers/test/get.py

```Python
def handler(event):
    user_id = event['queryStringParameters']['id']
    return { 'body': { 'id': user_id, 'name': "test" } }
```

`index.py`가 위치한 폴더로 이동하여 압축을 합니다.

```
zip -r sample.zip .
```

코드를 올린 후 테스트 했을 때 똑같은 결과가 나와야 합니다.

### Step 4. 외부 라이브러리를 pip로 설치하여 같이 올리기

외부 라이브러리 설치를 하기위해서는 주의해야 할 사항들이 몇가지 있습니다.

외부 라이브러리를 해당 폴더 내에 설치하기 위해서는 `virtualenv`를 사용하여 가상환경을 구성해주는게 편합니다.
그렇지 않으면 이미 해당 라이브러리가 설치된 경우 충돌이 일어 날수가 있어서 설치 자체가 쉽지 않습니다.
그리고 현재 기본적으로 실행되는 **Python**의 버전이 **3.x.x** 버전일 경우에도 문제가 됩니다.
현재 **AWS Lambda**에서 지원하는 **Python**이 **2.7**버전이기 때문입니다.

**Python 3**에서도 `virtualenv` 환경으로 **Python 2.7**로 생성이 가능합니다만, 필자는 구글에서 찾아봐서 몇 번 시도를 해봤는데 계속 실패하더라구요.
그래서 그냥 과감하게 그 당시 기본으로 설치된 **Python 3.5.12 (Conda)**를 날려버렸습니다.
그리고 따로 **Python 공식 페이지**에 들어가서 **2.7.12**와 **3.5.12**를 설치했습니다.
**Python** 설치는 **homebrew** 같은것으로 설치하는 것보다는 그냥 공홈에서 **.pkg** 같은걸로 다운받아서 설치하는게 정신 건강에 좋습니다.

```
$ python -V
Python 2.7.12
```

기본 버전이 **2.7.12** 라는 것이 확인되었으니 그냥 `virtualenv`로 가상환경을 만들면 되겠네요.
먼저 `index.py`가 위치한 곳으로 이동 후 다음과 같이 입력하여 실행해주세요.

```
virtualenv myvenv
```

이제 가상 환경으로 활성화 합니다.

```
source myvenv/bin/activate
```

이제 원하는 라이브러리를 로컬로 해당 폴더에 설치하면 됩니다.

예제로 작성할 코드라 가볍고 사용하기 쉬운 `requests`를 설치해 보도록 하겠습니다.

```
pip install requests -t .
```

일단은 단순하게 그냥 `index.py`만 수정해서 `requests`가 동작하는 코드로 수정 후 올려보도록 하죠.

#### index.py

```Python
import json
import requests

from router import router

def handler(event, context):
    result = router(event);

    URL = 'http://www.tistory.com'
    response = requests.get(URL)

    result['request_data'] = response.text

    return { 'body' : json.dumps(result) }
```

해당 폴더 이하를 몽땅 압축하여 올립니다. (**myvenv** 는 제외하고 싶은데... 어떻게 하는지 잘 모르겠습니다.)

```
zip -r sample.zip .
```

테스트를 하면 `request_data`안에 데이터들이 추가된 것을 확인 할 수 있습니다.



### Step 5. 좀 더 새련된 패키징 처리

**Step 4**에서 살펴본 내용만으로는 실제 서비스 가능한 수준의 코드를 만드는데는 몇가지 문제가 있습니다.

1. 압축파일의 크기가 너무 크다. `virtualenv`용 파일은 제외하고 압축하고 싶다.
2. **pip**를 이용한 모듈을 로컬에 설치했는데, 그럼 `index.py`와 `router.py`같이 루트폴더에 있는 파일들말고 `get.py`나 `post.py`같이 서브폴더에 있는 파일에서는 어떻게 모듈들을 사용해야 할까 ?

첫번째 문제에 대해서는 저도 맥에서 `zip`명령어를 사용해서 어떻게 특정 파일/폴더를 빼고 압축을 할수 있는지에 대해서 못찾았습니다.
그래서 그냥 `src`라는 폴더를 하나 만들어 그 안에서 작업을 하고 해당 폴더 안에서 `.zip`파일로 패키징해서 올리니 `virtualenv`관련 파일들은 자연스럽게 빠지게 되었습니다.

두번째 문제에 대해서 부모디렉토리의 모듈에 대해서 접근하는 여러가지 방법을 시도해 봤는데 코드가 많이 지저분해더군요.
코드를 깔끔하게 유지하면서 해결가능한 방법으로는 2가지 정도가 있습니다.

**해당 모듈이 사용되는 최하위 폴더에 설치한다.**
이 경우에는 해당 폴더의 상위에서는 모두 사용이 가능합니다.
하지만 그 상위 폴더와 같은 레벨의 다른 폴더에서의 접근은 힘듭니다. 
그 폴더에서도 사용하려면 그 폴더내의 어딘가에 또 설치를 하는 방법으로 해결을 해야 합니다.
이 방법은 별로 좋은 방법 같지 않으니 패스하기로 합니다.

**최상위 모듈에서 하위 폴더내 모듈에게 전달한다.**
개인적으로 추천하는 방법입니다.
**index.py** 같이 프로그램 내의 시작점에 해당되는 곳에서 사용되는 모든 모듈들의 객체를 선언하여 이것을 하위 폴더내의 모듈을 호출할 때 같이 전달하는 방식입니다.
설명으로만 느낌이 잘 안오실수 있으니 예제 코드로 살펴보겠습니다.

위 **Step 4** 예제를 조금 변경하여 `/test`경로로 `GET`방식으로 요청한 경우 쿼리 스트링으로 `url`값을 받아서 해당 사이트의 내용을 `requests` 모듈을 이용해서 가져오는 코드로 변경해 보겠습니다.

**Step 4**까지 진행되었다는 가정하에 진행할 경우에는 바로 하면 되지만 새로운 폴더에 작업을 하는 경우라면 아래와 같이 `virtualenv`를 이용한 가상환경으로 들어가서 `pip`를 로컬로 설치해 주신뒤 코드를 작성해 주세요.

```
virtualenv myvenv
source myvenv/bin/activate
mkdir src
cd src
```

압축할 때 **myvenv**내의 파일들이 같이 압축되지 않도록 작업을 **src**폴더 내에서 하겠습니다.

```
mkdir modules
touch modules/__init__.py
pip install requests -t modules/
```

modules을 다른 폴더에서 불러오기 위해서는 해당 폴더내에 `__init__.py` 파일이 있어야 합니다.

```
mkdir controllers
cd controllers
mkdir test
touch __init__.py
cd test
touch __init__.py
cd ..
cd ..
```

`index.py`에서 `requests` 모듈에 대한 객체를 선언하여 그것을 `/controllers/test/get.py`까지 전달하는 코드를 작성해 보겠습니다.

#### index.py
```Python
import json
import modules.requests

from router import router

def handler(event, context):
    
    packages = {}
    packages['requests'] = modules.requests
    
    result = router(packages, event);

    return { 'body' : json.dumps(result) }
```

`router.py`에서는 전달받은 `packages`를 그대로 전달해주는 코드만 추가했습니다.

#### router.py

```Python
import controllers.test.get
import controllers.test.post

route_map = {
    '/test': {
        'GET': controllers.test.get.handler,
        'POST': controllers.test.post.handler
    }
};

def router(packages, event):
    controller = route_map[event['path']][event['httpMethod']];
    
    if not controller:
        return { 'body': { 'Error': "Invalid Path" } }
    
    return controller(packages, event);
```

`post.py`에서 외부 모듈을 사용하지는 않지만, 일단 받아줍시다. 공짠데요.

#### controllers/test/post.py

```Python
def handler(packages, event):
    user_id = event['queryStringParameters']['id']
    body = event['body']
    header = event['headers']
    return { 'body': { 'id': user_id, 'header': header, 'body': body } }
```

`get.py`에서는 전달받은 `packages`에서 `requests` 객체를 가져와서 사용하는 코드가 추가되었습니다.

#### controllers/test/get.py

```Python
def handler(packages, event):
    requests = packages['requests']
    
    request_url = event['queryStringParameters']['url']
    response = requests.get('http://' + request_url)

    return { 'body': { 'url': request_url, 'text': response.text } }
```

이제 `index.py`가 있는 위치로 와서 압축한 뒤에 **Lambda**에 올려 놓고 테스트 해 보겠습니다.

```
zip -r sample.zip .
```

`https://본인 Lambda 주소.../prod/test?url=www.naver.com` 식으로 브라우저에서 입력하여 결과가 제대로 오는지 확인하시면 됩니다.


### 마치며...

이번 포스팅에서 알아본 내용들은 다음과 같습니다.

- **Lambda**에 **Python** 코드를 패키징해서 올리는 방법
- **vitrualenv**를 활용해서 원하는 폴더에 **pip**로 모듈을 설치하는 방법
- 로컬에 **pip**로 설치한 모듈들을 서브 폴더내에서도 활용하는 방법

잘못되었거나, 변경된 점, 기타 추가 사항에 대한 피드백은 언제나 환영합니다. - <seokjoon.yun@gmail.com>  

### 참고

- AWS 공식 가이드 : <http://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html>