---
title: AWS Lambda에 Python Slack Chatbot을 통해서 미세먼지 대기정보 알림이 만들기 
date: 2017-03-26 00:00:00
categories:
- AWS
- Lambda
tags:
- AWS
- Lambda
- APIGateway
- Python
---

# AWS Lambda에 Python Slack Chatbot을 통해서 미세먼지 대기정보 알림이 만들기

Lambda를 이용해서 Slack용 Chatbot을 만들어 보았다.  
개발언어로는 Python을 사용했다.  
Lambda에는 Python 2.7만 지원되어서 작업하면서도 불편한 점이 많았다.  
특히 챗봇이다보니 유니코드(한글) 처리가 필수여서다.  
(Node.JS는 6.1까지 지원해주는데... Python도 3을 빨리 지원해주면 좋겠다.)  

예제로 만들어본 챗봇은 서울시 종로구의 대기상태를 알려주는 기능을 제공한다.
매일 아침 5시 20분(내 기상시간)과 오후 12시 20분(점심 먹으러 가기 전에 지하식당에서 먹을지 밖에 나가서 먹을지 생각하기 위해서)에 알려주도록 설정하였다.
솔직히 오후 12시에 굳이 20분을 한 이유는 CloudWatch의 event 생성을 편하게 하기위해서이다.
솔직히 12시 40분에 알려주는게 더 좋을것 같긴하지만, 그것 때문에 event를 추가로 생성하기엔 귀찮기도 하고, 관리포인트가 두군데가 생기기 때문에 그냥 20분으로 통일했다.

챗봇 구현을 위한 Lambda를 2개로 나누었다.

- Slack에 메세지를 전달해주는 Lambda
- 크롤링하여 대기상태 데이터를 뽑아내고 메세지를 만드는 Lambda

크롤링 Lambda는 SNS를 통해서 메세지 전달 Lambda를 호출한다.  
크롤링 Lambda는 CloudWatch의 event를 통해서 호출된다.  

그 과정을 그림으로 표현하면 아래와 같다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Cloud/AWS/Lambda/images/Lambda.Chatbot.01.png?raw=true">

이전에 표스팅한 내용에 있는 것에 대해서는 설명을 생략하겠다.  
해당 내용에 대해서는 아래 링크를 참고하면 관련 내용이 있다.

- Python 코드를 Lambda에 배포하는 방법 : [Lambda Python Packaging](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda.Packaging.Python.md)
- Lambda -> SNS -> Lambda 로 호출하는 방법 : [Lambda에서 Lambda를 호출하는 방법](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda/Lambda.invoke.Lambda.md)

## 1. Slack에 메세지 보내기

먼저 Slack Bot을 만들기 위해서는 API Token이 필요하다.  
자세한 생성방법은 검색하면 많이 나오니깐 생략하겠다.  
간략하게 설명없이 방법만 적겠다.  

- Slack에서 채널명 클릭해서 나오는 메뉴에서 `Apps & intergrations` 클릭
  - `Bots` 검색해서 클릭
  - `Add Configuration`클릭
    - 안내대로 쭉 설정하고 나오는 화면에서 `API Token`값을 보고 기록해 놓음

이제 Slack에 메세지를 보내는 Python 함수를 먼저 만들어 보자.

만약 Python 버전이 2.7이 아닌 경우에는 `virtualenv`를 통해서 2.7 환경으로 만들어 놓고 작업을 해야한다.  
그 방법은 [Lambda Python Packaging](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda.Packaging.Python.md) 링크를 통해서 확인하기 바란다.

`slacker`라는 패키지를 이용하면 정말 쉽게 작성이 가능하다.  

우리는 Lambda에 배포해야하니 `pip`로 패키지를 설치할 때 해당 폴더에 설치해야 한다.

> pip install slacker -t .

이제 `index.py`라는 파일명으로 아래 코드를 입력한다.

```Python
# -*- coding: utf-8 -*-

from slacker import Slacker

token = '{Slack Bot API Token}'
slack = Slacker(token)

def handler(event, context):
    ch = event["channel"]
    message = event["message"]

    slack.chat.post_message(ch, msg)

if __name__ == '__main__':
    event= {};
    event["channel"] = "#general";
    event["message"] = "메인 테스트";
    
    handler(event, None);
```

실행을 했을 때 Slack의 `#general` 채널에 메세지가 출력되면 정상적으로 동작하는 것이다.

> python index.py

 <img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Cloud/AWS/Lambda/images/Lambda.Chatbot.02.png?raw=true">

 이제 Lambda로 올려서 테스트 해보자.

 >zip -r bot.zip .

 위 명령어로 압축을 한 후에 Lambda에 올려서 배포를 한 후 테스트 데이터를 아래와 같은 형태로 넣은 후 실행해서 Slack에 메세지가 나오면 성공한 것이다.

 ```JSON
{
    "channel": "#general",
    "message": "람다람다 테스트"
}
 ```

  <img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Cloud/AWS/Lambda/images/Lambda.Chatbot.03.png?raw=true">

## 2. 대기정보를 얻기위해서 웹 크롤링하기

위에 작성한 코드와는 다른 폴더에서 작업하겠다.  
왜냐면 다른 Lambda로 배포할 코드라 사용하지 않는 패키지까지 같이 포함되어 배포할 용량이 커지는걸 피하기 위해서다.

서울특별시 대기환경정보(<http://cleanair.seoul.go.kr/air_city.htm?method=measure>)에서 관련 데이터를 크롤링했다.  
(네이버는 크롤링을 막아놔서... ㅠㅠ 아마 적절히 User-Agent값을 넣는다던지, headers 정보를 감쪽같이 속인다던지, 웹브라우저 엔진을 이용한다던지 하는 방법을 이용하면 되겠지만... 뭐 대단한 거라고 그냥 다른 곳을 선택했다. 최대한 쉽게 쉽게...)

웹페이지 소스를 가져오기 위해서 `requests`를 사용했으며 해당 소스를 분석하기 위해서 `beautifulsoup`를 사용하였다.  
그동안 말로만 듣던 beautifulsoup를 처음 써봤는데, 100% 마음에 들지는 않았지만 그래도 원하는 기능들이 꽤 많이 구현되어 있었다.  
텍스트를 한줄 한줄 읽어가면서 분석해야하는 일을 많이 줄여주었다.

>pip install requests beautifulsoup -t .

위 페이지에서 개발자 도구를 열어서 소스도 살펴보고... 각 element 별로 클릭해서 적절한 id값도 찾아보고... 이래저래 해봤는데... 한방에 딱! 뽑아 낼 수 있는 구조가 아니다.
그냥 일단 받아놓고 노가다로 찾아야 겠다. 

```Python
from BeautifulSoup import BeautifulSoup
import json
import requests

URL = 'http://cleanair.seoul.go.kr/air_city.htm?method=measure'

response = requests.get(URL)
html_doc = response.text
soup = BeautifulSoup(html_doc)

print soup.prettify()
```

로 일단 찍어보자.  
여기서 뭘 보겠다는게 아니라 그냥 일단 잘 동작하나 보자.
겁나 길게 html이 우루루 출력되면 성공한 것이다.  

다시한번 개발자 도구로 해당페이지를 보니 아래쪽 표 부분에 서울시 각 도별로 대기상태 값들이 있다.
`<table>` 태그 정보들에 대해서 값을 추출해 보자.

```Python
tables = soup.findAll('table')
print tables
```

3번째 테이블에 원하는 정보가 있다.

각 `<tr>`별로 살펴봐서 우리가 원하는 위치(`종로구`)에 관한 정보만 찾아보자.

```Python
dataTable = tables[2]
trs = dataTable.findAll('tr')

for tr in trs:
    if '종로구' in str(tr):
        print tr;
```

우리가 원하는 정보는 저기에 다 있는게 확인되었다.  
이제 이걸 이용해서 메세지를 만들어서 출력해 보겠다.  

```Python
dataTable = tables[2]
trs = dataTable.findAll('tr')

for tr in trs:
    if '종로구' in str(tr):
        tds = tr.findAll('td')

        message1 = u'종로구의 현재 통합대기환경지수는 {}({}) 입니다.'.format(tds[7].getText(), tds[8].getText())
        message2 = u'미세먼지: {}㎍/㎥, 초미세먼지: {}㎍/㎥, 오존: {}ppm, 이산화질소: {}ppm, 일산화탄소: {}ppm, 아황산가스: {}ppm'.format(tds[1].getText(), tds[2].getText(), tds[3].getText(), tds[4].getText(), tds[5].getText(), tds[6].getText())

        messageTotal = message1 + u"(" + message2 + ")"
        print messageTotal
```

>종로구의 현재 통합대기환경지수는 보통(70) 입니다.(미세먼지: 44㎍/㎥, 초미세먼지: 25㎍/㎥, 오존: 0.036ppm, 이산화질소: 0.033ppm, 일산화탄소: 0.5ppm, 아황산가스: 0.004ppm)

이제까지의 코드를 한번 정리해 보겠다.

```Python
# -*- coding: utf-8 -*-

from BeautifulSoup import BeautifulSoup
import json
import requests

URL = 'http://cleanair.seoul.go.kr/air_city.htm?method=measure'

def GetInfo(gu):
    response = requests.get(URL)
    html_doc = response.text
    soup = BeautifulSoup(html_doc)

    tables = soup.findAll('table');
    dataTable = tables[2]
    trs = dataTable.findAll('tr')

    for tr in trs:
        if gu in str(tr):
            return tr;

def MakeMessage(data):
    tds = data.findAll('td')

    message1 = u'종로구의 현재 통합대기환경지수는 {}({}) 입니다.'.format(tds[7].getText(), tds[8].getText())
    message2 = u'미세먼지: {}㎍/㎥, 초미세먼지: {}㎍/㎥, 오존: {}ppm, 이산화질소: {}ppm, 일산화탄소: {}ppm, 아황산가스: {}ppm'.format(tds[1].getText(), tds[2].getText(), tds[3].getText(), tds[4].getText(), tds[5].getText(), tds[6].getText())

    messageTotal = message1 + u"(" + message2 + ")"
    return messageTotal

if __name__ == "__main__":
    print MakeMessage(GetInfo('종로구'))
```

## 3. SNS을 통해서 Slack Lambda 호출하기

자세한 설명은 [Lambda에서 Lambda를 호출하는 방법](https://github.com/DevStarSJ/Study/blob/master/Blog/Cloud/AWS/Lambda/Lambda.invoke.Lambda.md) 링크를 통해서 확인하기 바란다.

해당 SNS를 통해서 메세지를 보내기 위해서 SNS의 ARN 주소를 어디에 적어 놓아야 한다.

SNS 에서 Lambda 로 전달되는 메세지 형태를 처리하기 위해서는 위 코드를 수정해야 한다.  
참고로 `Subject`로 채널명을 입력받고 `Message` 전달할 메세지를 입력받도록 설정하였다.

```Python
# -*- coding: utf-8 -*-

from slacker import Slacker

token = '{Slack Bot API Token}'
slack = Slacker(token)

def handler(event, context):
    keys = event.keys()
    
    ch = ""
    message = ""
    
    if 'channel' in keys:
        ch = event["channel"]
        msg = event["message"]
    else:
        ch = event['Records'][0]['Sns']['Subject'];
        msg = event['Records'][0]['Sns']['Message'];

    slack.chat.post_message(ch, msg)

if __name__ == '__main__':
    event= {};
    event["channel"] = "#general";
    event["message"] = "메인 테스트";
    
    handler(event, None);
```

### 4. 대기정보 Lambda에서 SNS로 메세지 보내기

2번 항목에서 만든 코드에 SNS를 통해서 메세지를 전달하는 코드를 추가해 보자.  
먼저 AWS 서비스를 이용하기 위해 `boto`를 설치하자.

>pip install boto -t .

```Python
# -*- coding: utf-8 -*-

from BeautifulSoup import BeautifulSoup
from datetime import datetime
import json
import requests
import boto.sns

URL = 'http://cleanair.seoul.go.kr/air_city.htm?method=measure'

REGION = '{리전}'
TOPIC  = '{SNS ARN 주소}'

conn = boto.sns.connect_to_region( REGION )

def GetInfo(gu):
    response = requests.get(URL)
    html_doc = response.text
    soup = BeautifulSoup(html_doc)

    tables = soup.findAll('table');
    dataTable = tables[2]
    trs = dataTable.findAll('tr')

    for tr in trs:
        if gu in str(tr):
            return tr;

def MakeMessage(data):
    tds = data.findAll('td')

    message1 = u'종로구의 현재 통합대기환경지수는 {}({}) 입니다.'.format(tds[7].getText(), tds[8].getText())
    message2 = u'미세먼지: {}㎍/㎥, 초미세먼지: {}㎍/㎥, 오존: {}ppm, 이산화질소: {}ppm, 일산화탄소: {}ppm, 아황산가스: {}ppm'.format(tds[1].getText(), tds[2].getText(), tds[3].getText(), tds[4].getText(), tds[5].getText(), tds[6].getText())

    messageTotal = message1 + u"(" + message2 + ")"
    return messageTotal

def handler(event, context):
    msg = MakeMessage(GetInfo('종로구'))

    pub = conn.publish( topic = TOPIC, subject ="#general"  ,message = msg )

if __name__ == "__main__":
    handler(None, None)
```

Lambda로 올리기 전에 테스트로 실행해 본 후 정상적으로 실행되는지 확인해 보자.  

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Cloud/AWS/Lambda/images/Lambda.Chatbot.04.png?raw=true">

위 코드 실행 후 Slcak을 통해서 메세지가 전달되었다면 성공한 것이다.  
위 코드도 폴더 전체를 압축해서 Lambda로 배포하자. 

## 5. CloudWatch를 통해서 event 생성하기

CloudWatch의 event를 통해서 특정 시점에 Lambda 를 실행시킬 수 있다.

- `CloudWatch` -> `Events` -> `Create rule`
  - `Add taget`을 눌러서 위에 배포한 Lambda를 선택 : 별도로 인자로 받는게 없으므로 나머지 설정들은 기본 그대로 둠
  - `Event Source`를 `Schedule`로 선택
    - `Cron Expression` : `20 3,20 * * ? *`로 입력 (UTC 기준 매일 3시 20분, 20시 20분에 실행을 시킨다는 뜻)

저장을 하면 끝이다.

이제 설정한 시간에 Lambda가 실행되어서 Slack에 메세지를 전달해 줄 것이다.

<img src="https://github.com/DevStarSJ/Study/raw/master/Blog/Cloud/AWS/Lambda/images/Lambda.Chatbot.05.png?raw=true">

하루가 지난 시점에 확인해보니 설정한 시간에 꼬박꼬박 실행 중이다.

