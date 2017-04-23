---
title: Lambda에서 Lambda를 호출하는 방법 
date: 2017-03-19 00:00:00
categories:
- AWS
- Lambda
tags:
- AWS
- Lambda
---

# Lambda에서 Lambda를 호출하는 방법

AWS Lambda에서 Lambda를 호출하는 방법들에 대해서 소개하겠다.

## 언제 Lambda에서 Lambda를 호출하면 편리할까 ?

첫째, 배치 성격의 작업을 여러 개로 나누어 병렬로 실행할 경우를 생각할 수 있다.
Lambda 내에서 배치작업을 모두 다 실행시키도록 작성 할 수도 있겠지만,
첫번째 Lambda에서 작업 전 필요한 준비작업을 한 후 이것을 다른 Lambda로 전달 할 수 있다.
그 과정에서 다음 Lambda 1개에게 전달할게 아니라 비동기 invoke로 한 번에 여러 개의 Lambda를 생성하여 작업하면 전체 실행 시간을 줄일수 있을 것이다.
(하지만 동시에 여러 Lambda가 실행될 경우 계정당 Lambda Limit 내에서 실행되도록 조정은 해야 한다.)

이 경우 [AWS Step Functions](https://aws.amazon.com/step-functions) 를 이용할 수도 있다.
Step Functions를 사용하려고 살펴봤는데 아직까진 사용 사례도 없고, 그 과정에서의 비용도 있고, API Gateway와의 연동에 대해서도 확신이 없다.
[API Gateway에서 Step Function 호출을 지원](https://aws.amazon.com/ko/about-aws/whats-new/2017/02/amazon-api-gateway-integration-with-aws-step-functions)한다는 글이 2017.02.15 에 올라오긴 했다.

둘째, 빠른 API 응답을 위해서 사용할 수 있다.
Lambda를 만들었을 때의 원래 의도는 AWS 서비스 상에서의 event성 호출들에 대한 처리를 전적으로 할 목적이었다고 한다.
하지만 API Gateway가 나온 이후로 Lambda를 이용하여 Serverless API를 구축하는게 가능하다.
API를 호출한 사용자 입장에서 생각해 본다면 필요한 데이터를 빨리 응답해 주고, 나머지 작업들은 그 뒤에 따로 처리를 하는 방법이 있으면 좋을 것이다.
예를 들어서 사용자가 접속시 호출해야하는 API가 있다고 가정해보자.
이 API에서는 사용자가 이전에 어떤 작업을 하고 있었는지를 조회해서 알려줘야하며, 현재 시점의 접속 상태(시간, 위치, 접속한 기기정보 등)를 기록해야 한다면, 먼저 사용자에게 전달해야할 정보를 조회한 뒤, 기록해야 할 정보에 대해서 다른 Lambda를 비동기 invoke 한 후 응답을 하도록 설계가 가능하다.

Lambda에서 Lambda 를 호출하는 방법에는 동기 invoke와 비동기 invoke 2가지가 있다.

- 동기 invoke : Lambda 호출 후 그 응답을 기다린다. 응답 결과를 보고 뭔가 처리를 해야할 경우에 사용된다.
- 비동기 invokde : Lambda 호출 후 응답을 기다리지 않고 지나간다.

동기든 비동기든 둘 다 Lambda 비용에 대해서 생각을 한다면 Lambda 를 2개 이상으로 나누는 것이 무조건 손해이다.
Lambda는 과금이 *설정 메모리 x 수행시간(100 ms 단위)* 이다.
만약 평균 수행시간 199 ms 인 작업을 2개의 Lambda로 나뉠 경우 100 ms + 99 ms로 나뉜다면 1개로 하는 것과 같은 비용이 들겠지만 101ms + 98ms 로 나뉜다면 100ms 수행시간에 대한 비용을 더 지불해야 한다. 이건 비동기 호출의 경우이고, 동기 호출이라면 첫번째 Lambda가 2번째 Lambda를 기다려야 하므로 최선의 경우라도 300 ms 만큼의 요금이 부과될 것이다. 더군다나 2번째로 호출되는 Lambda가 cold start 될 경우 첫번째 Lambda가 그 응답을 기다려야 하므로 비용은 훨씬 더 늘 것이다.
이점을 충분히 고려해서 해당 비용을 감안하더라도 Lambda를 나누는것이 유리하다고 판단이 되는 작업을 나누길 바란다.

이제부터 Lambda에서 Lambda를 호출하는 방법들에 대해서 하나 하나씩 살펴보도록 하자.

예제는 **Node.JS**로 작업하겠다.
AWS내의 event를 **JSON Object**로 전달하므로 Node.JS로 작업하는게 가장 편하다.
다른 언어로 하려면 event 모양별로 model을 미리 선언해 주거나 stream을 이용해서 해당 언어별 JSON을 지원해주는 라이브러리로 전달하는 방법을 사용하여야 한다. 응답해야할 결과가 있고, 그것이 JSON Object일 경우에는 응답에도 똑같은 처리를 해줘야 한다. Python의 경우는 그나마 JSON 친화적(?) 이어서 Node.JS 다음으로 편한 것 같다.

## Lambda에서 Lambda를 호출하는 방법

### 1. Lambda를 직접 invoke

유일하게 Lambda 간의 동기 invoke가 가능한 방법이다.  
가장 쉬운 방법이기도 하다.

먼저 호출 될 Lambda를 먼저 등록하자.
Lambda 등록에 대한 설명은 생략하겠다.
이전에 포스팅한 글을 참조하길 바란다.

> [AWS Lambda와 API Gateway를 이용해서 Serverless Web API 만들기](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda%2BAPIGateWay.01.md)

#### 1.1 호출될 Lambda 등록

아래 코드로 `echoTest`란 이름으로 Lambda를 등록한다.

```JavaScript
'use strict';

const util = require('util');

exports.handler = (event, context, callback) => {
  console.info(util.inspect(event, {depth:null}));
  callback(null, event);
};
```

등록 후 오른쪽 위를 보면 `ARN (Amazon Resource Name)`을 볼 수 있다.

>`ARN` - arn:aws:lambda:{region}:{id}:function:echoTest

이걸 이용해서 `aws-sdk`를 사용해서 Lambda를 호출할 것이다.

#### 1.2 동기 호출 코드

이제 호출 할 Lambda를 작성해보록 하자.

먼저 local에서 테스트해보기 위해서는 `aws-sdk`를 설치해야 한다.
Lambda 상에는 설치가 되어 있으므로 같이 올릴 필요가 없다.

> npm install aws-sdk --save

```JavaScript
'use strict'

const aws = require('aws-sdk');
const lambda = new aws.Lambda({
  region: 'ap-northeast-1' //change to your region
});

const event = { id: "1", name:"Luna"};
lambda.invoke({
  FunctionName: 'echoTest',
  Payload: JSON.stringify(event, null, 2) // pass params
}, function(error, data) {
  if (error) {
    console.info(error);
  } else {
    console.info(data);
  }
});
```

만약 위 코드를 `index.js`로 저장했다면 아래와 같이 실행하면 결과를 확인할 수 있다.

> node index.js

```JavaScript
{ StatusCode: 200, Payload: '{"id":"1","name":"Luna"}' }
```

#### 1.3 비동기 호출 코드

위 코드는 동기 invoke 하였다.
비동기로 하기 위해서는 예전에는 `InvokeAsync()` 함수를 사용했는데 이 방법은 *deprecated* 되었다.
지금은 `InvocationType`이란 인자에 `Event`값을 추가해주면 된다.
참고로 default인 동기 호출의 설정값은 `InvocationType: 'RequestResponse'`이다.

```JavaScript
'use strict'

const aws = require('aws-sdk');
const lambda = new aws.Lambda({
  region: 'ap-northeast-1' //change to your region
});

const event = { id: "1", name:"Luna"};
lambda.invoke({
  FunctionName: 'echoTest',
  InvocationType: 'Event',
  Payload: JSON.stringify(event, null, 2) // pass params
}, function(error, data) {
  if (error) {
    console.info(error);
  } else {
    console.info(data);
  }
});
```

위와 같이 코드를 수정 후 호출하면 응답코드가 다르게 전달되는걸 확인할 수 있다.

```JavaScript
{ StatusCode: 202, Payload: '' }
```

앞의 동기 invoke의 경우 `200(성공)`으로 서버가 요청을 정상적으로 처리했다는 코드를 보내왔지만,
아래 비동기 invoke의 경우 `202(허용됨)`으로 서버가 요청을 접수했지만 아직 처리하지 않았다는 응답을 보내왔다.

Lambda 상에서 제대로 처리된지 볼려면 echoTest에 `console.info`로 출력한 내용을 보면 된다.

>
- 11:52:05
START RequestId: 094d4d3e-0c4f-11e7-ab68-a96f2d3f979f Version: $LATEST  
- 11:52:05
2017-03-19T02:52:05.890Z	094d4d3e-0c4f-11e7-ab68-a96f2d3f979f	{ id: '1', name: 'Luna' }  
- 11:52:05
END RequestId: 094d4d3e-0c4f-11e7-ab68-a96f2d3f979f  
- 11:52:05
REPORT RequestId: 094d4d3e-0c4f-11e7-ab68-a96f2d3f979f	Duration: 208.59 ms	Billed Duration: 300 ms Memory Size: 128 MB	Max Memory Used: 15 MB  
- 11:58:09
START RequestId: e23bf84a-0c4f-11e7-81f6-25e9632564aa Version: $LATEST  
- 11:58:09
2017-03-19T02:58:09.644Z	e23bf84a-0c4f-11e7-81f6-25e9632564aa	{ id: '1', name: 'Luna' }  
- 11:58:09
END RequestId: e23bf84a-0c4f-11e7-81f6-25e9632564aa  
- 11:58:09
REPORT RequestId: e23bf84a-0c4f-11e7-81f6-25e9632564aa	Duration: 23.59 ms	Billed Duration: 100 ms Memory Size: 128 MB	Max

위 11:52에 수행된게 동기로 호출한 결과이며 아래 11:58에 수행된게 비동기로 호출한 결과이다. 둘 다 똑같이 `console.info`로 출력한 결과가 있는걸로 봐서 정상적으로 수행되었다고 판단할 수 있다. 수행시간이 다른건 앞에껀 cold start가 되었고, 뒤에껀 warm start가 된 결과로 보인다.

위 테스트 코드를 Lambda로 올리는 과정은 생략하겠다.
수행할 코드들을 `exports.handler = (event, context, callback) => { }`로 감싸서 올리면 된다.

### 2. SNS의 Subscription으로 Lambda를 등록

[AWS SNS (Simple Notification Service)](https://aws.amazon.com/sns)는 간단한 푸시 알림 서비스이다.
SNS를 하나 만들어서 실행할 Lambda를 구독자로 등록해 놓고, 해당 SNS에 publish하는 방법으로 연동을 생각해 볼 수 있다.

#### 2.1 SNS 생성 및 Lambda를 subscription으로 등록

먼저 SNS를 생성한다.

1. SNS dashboard에서 `Create topic`을 누름 (Topics에서 `Create new topic`를 눌러도 됨)
2. Topic 생성
  - Topic name : testSns
  - Display name : testSns

`Actions`를 눌러서 `Edit topic policy`나 `Edit topic delivery policy`를 눌러서 각종 설정이 가능하다.
여기에서는 그 내용에 대해서는 다루지 않겠다.
별로 어렵지 않으니 필요할 경우 직접 해보길 바란다.

이제 해당 SNS에 Lambda를 등록할건데,
앞에서 작성한 `testEcho`를 등록하도록 하자.

1. 생성된 SNS의 ARN링크  `arn:aws:sns:{region}:{id}:testSns`를 눌러서 `Topic details`로 들어감
2. `Create subscription`을 누름
  - Protocol : `AWS Lambda`를 선택
  - Endpoint : 실행할 Lambda의 ARN을 선택
  - Version or alias : 일단 default 그대로 둠 (실제 서비스 할 경우에는 alias로 설정을 해야 편함)

Version을 **$LASTEST**로 설정을 하면 관련 권한이 없다고 오류가 발생한다.
일단은 그냥 넘어가자.
실제 서비스 하는 경우라도 **$LASTEST**는 상당히 위험한 방법이다.
Lambda 배포 후 테스트 과정을 거쳐서 검증되기 전까지는 `alias`를 사용해서 믿을만한 버전으로 서비스하는게 좋다.

#### 2.2 SNS에서 Lambda 호출 테스트

SNS 콘솔에서 Lambda를 제대로 호출하는지 테스트 해보자.

- `Topic details` 화면에서 `Publish to topic`를 누름
  - Subject : `test subject`
  - Message format : `JSON`
  - `JSON message generator`를 눌러서 Message에 입력 : `지켜보고 있다.`
  - `Generate JSON` 을 누름
  - `Publish message` 누름

이제 Lambda 의 CloudFront로 가서 제대로 호출되었는지 살펴보자.

```JavaScript
{ Records: 
[ { EventSource: 'aws:sns',
EventVersion: '1.0',
EventSubscriptionArn: 'arn:aws:sns{region}:{id}:testSns:1009056c-eb87-4f68-9cb1-de50b0024b90',
Sns: 
{ Type: 'Notification',
MessageId: 'd0e91f46-c46c-5ead-9633-4e013fb6c04d',
TopicArn: 'arn:aws:sns{region}:{id}:testSns',
Subject: 'test subject',
Message: '지켜보고 있다.',
Timestamp: '2017-03-19T03:23:06.272Z',
SignatureVersion: '1',
Signature: '...',
SigningCertUrl:  'https://...',
UnsubscribeUrl: 'https://...',
MessageAttributes: {} } } ] }
```

위의 결과가 출력된걸로 봐서 제대로 호출되었다고 판단이 가능하다.

#### 2.3 코드에서 SNS를 호출하기

아래와 같이 코드를 작성하자.
그냥 `sns.js`로 저장하도록 하겠다.

```JavaScript
'use strict'

const AWS = require('aws-sdk');

AWS.config.update({
  accessKeyId: "...",
  secretAccessKey: "...",
  region: '...'
});

const sns = new AWS.SNS();
const params = {
  TargetArn:'arn:aws:{region}:{id}:testSns',
  Message:'{"id":"snsFromNodeJS", "name":"Luna"}}',
  Subject: 'TestSNS'
};

sns.publish(params, function(err,data){
  if (err) {
    console.log('Error sending a message', err);
  } else {
    console.log('Sent message:', data.MessageId);
    console.info(data);
  }
});
```

실행하면 다음과 같은 응답을 얻을 수 있다.

> node sns.js

```JavaScript
{ ResponseMetadata: { RequestId: '64efe66a-abfa-5a12-8abd-b72d55ebe4ce' },
  MessageId: '207e6adf-277a-5191-a94b-3b98d96b3f4d' }
```

Lambda 의 CloudFront로 가서 제대로 호출되었는지 살펴보자.

```JavaScript
[ { EventSource: 'aws:sns',
EventVersion: '1.0',
EventSubscriptionArn: 'arn:aws:sns:{region}:{id}:testSns:1009056c-eb87-4f68-9cb1-de50b0024b90',
Sns: 
{ Type: 'Notification',
MessageId: '207e6adf-277a-5191-a94b-3b98d96b3f4d',
TopicArn: 'arn:aws:sns:{region}:{id}:testSns',
Subject: 'TestSNS',
Message: '{"id":"snsFromNodeJS", "name":"Luna"}}',
Timestamp: '2017-03-19T03:32:33.996Z',
SignatureVersion: '1',
Signature: '...',
SigningCertUrl: 'https://...',
UnsubscribeUrl: 'https://...',
MessageAttributes: {} } } ] }
```

위 출력문으로 봐서 제대로 호출된게 확인 된다.

### 3. Kinesis로 Lambda 호출하기

[AWS Kinesis](hhttps://aws.amazon.com/kinesis/streams) 는 스트리밍 데이터들을 처리하기 위한 용도로 만들어 졌다.
입력받은 데이터들을 100개 단위(설정에서 변경 가능)로 묶어서 처리한다던지 그런 용도로 많이 사용한다.
Kinesis를 Lambda에서 Lambda 호출용으로 사용하는건 좀 오바스럽긴 하지만 일단 한번 해보자.

#### 3.1 Kinesis Stream 생성

- Kinesis Streams 콘솔로 들어가서 `Create stream`를 누름
  - Stream name : `testKinesis` 입력
  - Number of shards : `1`입력
  - `Create stream`을 누름 (나머지 설정은 그냥 건드리지 않음)

Kinesis Stream이 만들어지고 난 다음 Details 탭으로 들어가서 ARN을 확인한다.

> arn:aws:kinesis:{region}:{id}:stream/testKinesis

#### 3.2 Lambda에서 trggier로 Kinesis 등록하기

먼저 Lambda의 Role에 Kinesis 관련 권한이 있는지 확인하고 없으면 추가해 준다.

```
"kinesis:GetRecords",
"kinesis:GetShardIterator",
"kinesis:DescribeStream",
"kinesis:ListStreams"
```

위에 나열한 권한이 필요하다.

1. `echoTest` Lambda 설정으로 이동
2. `Add trigger`를 누름
  - `Kinesis` 선택
    - Kinesis stream : `testKinesis` 선택
    - Batch size : `1` 입력
    - Start position : `Trim horizon`을 선택
    - `Submit` 누름

#### 3.3 코드에서 Kinesis 호출하기

아래와 같이 코드를 작성하자.
그냥 `kinesis.js`로 저장하도록 하겠다.

```JavaScript
'use strict'

const AWS = require('aws-sdk');

AWS.config.update({
  accessKeyId: "...",
  secretAccessKey: "...",
  region: '...'
});

const kinesis = new AWS.Kinesis();

const KinesisData = {
  event_name: "Kinesis 예제",
  event_created_at: new Date()
};

const params = {
  Data: JSON.stringify(KinesisData),
  PartitionKey: 'partition',
  StreamName: 'testKinesis'
};

kinesis.putRecord(params, function(err,data){
  if (err) {
    console.log('Error sending a message', err);
  } else {
    console.info(data);
  }
});
```

이제 실행해보자.

>node kinesis.js

```JavaScript
{ ShardId: 'shardId-000000000000',
  SequenceNumber: '49571470617410410658004564491315382019801680888626937858' }
```

Lambda쪽 실행 결과를 살펴보자.

```JavaScript
{ Records: 
[ { kinesis: 
{ kinesisSchemaVersion: '1.0',
partitionKey: 'partition',
sequenceNumber: '49571470617410410658004564491315382019801680888626937858',
data: 'eyJldmVudF9uYW1lIjoiS2luZXNpcyDsmIjsoJwiLCJldmVudF9jcmVhdGVkX2F0IjoiMjAxNy0wMy0xOVQwNDo0ODo0My4xOTlaIn0=',
approximateArrivalTimestamp: 1489898923.642 },
eventSource: 'aws:kinesis',
eventVersion: '1.0',
eventID: 'shardId-000000000000:49571470617410410658004564491315382019801680888626937858',
eventName: 'aws:kinesis:record',
invokeIdentityArn: 'arn:aws:iam::768556645518:role/lambda2',
awsRegion: 'ap-northeast-1',
eventSourceARN: 'arn:aws:kinesis:ap-northeast-1:768556645518:stream/testKinesis' } ] }
```

살펴보니 data의 값이 누가봐도 암호화 된 것으로 보인다.

> eyJldmVudF9uYW1lIjoiS2luZXNpcyDsmIjsoJwiLCJldmVudF9jcmVhdGVkX2F0IjoiMjAxNy0wMy0xOVQwNDo0ODo0My4xOTlaIn0=

[Amazon Kinesis Streams Service API Reference](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html) 를 보면 `base64`로 암호화 되었다고 나온다.

#### 3.4 Lambda 코드에서 Kinesis Data 복호화

`echoTest` Lambda 코드를 아래와 같이 수정한다.

```
'use strict';

const util = require('util');

exports.handler = (event, context, callback) => {
  const records = event.Records;
  console.log("records =", records);
  
  records.forEach(r => {
    const data = new Buffer(r.kinesis.data, 'base64').toString('utf-8');
    console.info(data);
  });
  
  callback(null, event);
};
```

다시 Kinesis 호출하는 코드를 실행해 보자.

> node kinesis.js

```JavaScript
{ ShardId: 'shardId-000000000000',
  SequenceNumber: '49571470617410410658004564491322635574719439376016736258' }
```
Lambda쪽 실행 결과를 살펴보자.

```JavaScript
{
    "event_name": "Kinesis 예제",
    "event_created_at": "2017-03-19T05:05:52.273Z"
}
```

데이터가 잘 전달된 것을 확인할 수 있다.

Kinesis에서 받은 event를 보면 **Records**란 Key에 대해서 배열로 생성되어 있다.
Kinesis에서 Data들을 설정된 값 단위로 배열을 전달해 준다.
만약 100개로 선언되었다고 해서 항상 100개 단위로 주는건 아니다.
그 갯수 이하라도 데이터를 전달해주니 그걸 고려해서 코드를 작성해야 한다.

### 4. 그 밖에 방법들

앞의 Kinesis 연동 방법을 설정하는 중 Lambda의 Triggers 탭에서 본 것처럼 AWS 상의 여러 서비스에서 Lambda 수행이 가능하다.
그걸 이용해서 호출하려는 Lambda에서 해당 서비스에 데이터를 전달하고 Lambda를 호출하는 방법으로의 설정이 얼마든지 가능하다.
하지만 단순히 Lambda to Lambda 호출을 위해서 그런 방법을 사용하는건 비효율적인듯하다.

대신, 원래 하려는 작업이 S3나 DynamoDB에 데이터를 쌓는게 목적이라면 거기에 데이터를 저장하고 해당 작업에 대한 Trigger로 Lambda를 수행하도록 설정하는건 괜찮을것 같다.

이 내용에 대해서는 따로 다루지 않겠다.
AWS 공식문서 상에 자습서를 참고해서 따라해보면 금방 익힐 수 있을 것이다.

- [자습서: DynamoDB 테이블에서 새 항목 처리](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.Lambda.Tutorial.html)






    


