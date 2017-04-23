---
title: Lambda Node.JS Packaging
date: 2016-11-27 15:54:51
categories:
- AWS
- Lambda
tags:
- AWS
- Lambda
- APIGateway
- Node.JS
---
# Lambda Node.JS Packaging

**Labmda**에 **Node.JS**로 구현하는 내용에 대해서는 아래 3개의 Link를 참고해 주세요.

- [Lambda 와 API Gateway 연동 #1 (GET, POST)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateWay.01.md)
- [Lambda 와 API Gateway 연동 #2 (ANY, Deploy Staging, Node.JS Route)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateway.02.Route.md)
- [Lambda 와 API Gateway 연동 #3 (Proxy Resource)](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateway.03.Proxy.md)

이제껏 1개의 **Node.JS** 파일에 **npm module**을 하나도 사용하지 않은 예제만 살펴보았는데,
실제로 개발할 상황에서는 파일도 여러 개로 나눠서 개발하고 **npm module**도 많이 사용할 경우가 많습니다.
이렇게 개발한 코드들을 어떻게 Lambda로 올리는지 이번 포스팅에서 살펴보도록 하겠습니다.

**Lambda 와 API Gateway 연동 #3**까지 구현되어 있다는 가정하에서 진행하겠습니다.

## Node.JS 코드에 npm 사용

- 이전까지 작업한 코드를 `index.js`란 이름으로 작업할 폴더에 저장
- **npm**으로 **lodash** 설치 : `npm install lodash`
- 코드를 아래와 같이 수정

```JavaScript
'use strict';

const _ = require('lodash');

function get(userId) {
  return {
    body: { id: userId, name: "test" }
  };
}

function post(userId, header, body) {
  return {
    body: { id: userId, header: header, body: body }
  };
}

const routeMap = {
  '/test': {
    'GET': (event, context) => {
      const userId = _.get(event,'queryStringParameters.id');
      return get(userId);
    },
    'POST': (event, context) => {
      const userId = _.get(event,'queryStringParameters.id');
      const body = JSON.stringify(_.get(event,'body'));
      const header =  event.headers;
      return post(userId, header, body);
    }
  }
};

function router(event, context) {
  const controller = routeMap[event.path][event.httpMethod];

  if(!controller) {
    return {
      body: { Error: "Invalid Path" }
    };
  }

  return controller(event, context);
}

exports.handler = (event, context, callback) => {
   let result = router(event, context);
   callback(null, {body:JSON.stringify(result)});
}
```

**routeMap** 내부에 `lodash.get` 함수를 사용하는 것으로 코드를 변경하였습니다.

- zip 파일로 압축 : `zip -r ./sample.zip ./`

## Lambda에 zip 파일로 배포

먼저 해당 **Lambda** 설정으로 이동합니다.

- `Code` 탭
  -  Code entry type : `Upload a .ZIP file` 선택
    - `Upload` 버튼을 눌러서 위에서 생성한 `sample.zip`을 올림

<img src="https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/images/lambda.packaging.node.01.png?raw=true">


## 확인

**API Gateway**는 변경사항이 없으므로 결과는 **3장**과 똑같이 나옵니다.

- **GET** 요청 : `https://.../prod/test?id=2`

```JSON
{"body":{"id":"2","name":"test"}}
```

- **POST** 요청 : URL은 **GET**과 동일

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

## 여러 파일을 함께 배포

이미 **npm module**을 포함하여 베포하였으므로 여러 파일을 묶어서 배포하는게 확인되었지만, 우리가 작성한 소스코드 자체도 나눠서 배포해 보도록 하겠습니다.
위에 작성한 코드를 4개의 파일로 나눠서 올려보겠습니다.

- `index.js`

```JavaScript
'use strict';

const router = require('./router');

exports.handler = (event, context, callback) => {
   let result = router(event, context);
   callback(null, {body:JSON.stringify(result)});
}
```

- `router.js`

```JavaScript
'use strict';

const _ = require('lodash');

const routeMap = {
  '/test': {
    'GET': (event, context) => {
      const userId = _.get(event,'queryStringParameters.id');
      return require('./controllers/test/get')(userId);
    },
    'POST': (event, context) => {
      const userId = _.get(event,'queryStringParameters.id');
      const body = JSON.stringify(_.get(event,'body'));
      const header =  event.headers;
      return require('./controllers/test/post')(userId, header, body);
    }
  }
};

module.exports = (event, context) => {
  const controller = routeMap[event.path][event.httpMethod];

  if(!controller) {
    return {
      body: { Error: "Invalid Path" }
    };
  }

  return controller(event, context);
};
```

- `/controllers/test/get.js`

```JavaScript
module.exports = (userId) => {
  return {
    body: { id: userId, name: "test" }
  };
};
```

- `/controllers/test/post.js`

```JavaScript
module.exports = (userId, header, body) => {
  return {
    body: { id: userId, header: header, body: body }
  };
};
```

- **npm**으로 **lodash** 설치 : `npm install lodash`
- zip 파일로 압축 : `zip -r ./sample.zip ./`

위에서와 같은 방법으로 **Lambda**에 배포 후 확인을 해보면 같은 결과가 출력됩니다.

## Lambda에 zip 파일로 배포

먼저 해당 **Lambda** 설정으로 이동합니다.

- `Code` 탭
  -  Code entry type : `Upload a .ZIP file` 선택
    - `Upload` 버튼을 눌러서 위에서 생성한 `sample.zip`을 올림

## 확인

**API Gateway**는 변경사항이 없으므로 결과는 **3장**과 똑같이 나옵니다.

- **GET** 요청 : `https://.../prod/test?id=2`

```JSON
{"body":{"id":"2","name":"test"}}
```

- **POST** 요청 : URL은 **GET**과 동일

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

### 마치며...

이번 포스팅에서 알아본 내용들은 다음과 같습니다.

- **Lambda**에 여러 파일을 `.zip`으로 묶어서 함께 배포
  - **npm module**을 함께 배포
  - 여러 파일로 작성된 소스코드들을 함께 배포 

잘못되었거나, 변경된 점, 기타 추가 사항에 대한 피드백은 언제나 환영합니다. - <seokjoon.yun@gmail.com>  

### 참고

- AWS 공식 가이드 : <http://docs.aws.amazon.com/lambda/latest/dg/nodejs-create-deployment-pkg.html>
