---
title: Running Multiple Coroutines Concurrently
date: 2016-08-19 00:00:00
categories:
- Python
- Python
tags:
- Python
---

# Python : Running Multiple Coroutines Concurrently

파이썬에서 `asyncio`로 만들어진 Coroutine들 여러 개를 동시에 돌리는 방법에 대해서 소개해 드리겠습니다.

파이썬 코루틴들은 `event_loop`에 등록이 되어서 동작합니다.

가장 먼저 생각할 수 있는 방법으로는 멀티쓰레드를 이용한 방법입니다만, Python의 쓰레드는 동시에 동작하지 않습니다.
그래서 멀티스레드 작업은 같은 작업을 싱글스레드에서 하는것보다 더 느립니다.
더군다나 하나의 프로세스에서 동작하는 코루틴 들에 `event_loop` 등록을 각각하게 되면 오류가 발생합니다.
그래서 멀티쓰레드로 구현하는 방법은 일단 제외하겠습니다.

### 1. 멀티프로세스(multiprocessing)로 구현

파이썬은 멀티프로세스로 작업을 구현하는것이 가능합니다.
이 경우 주의해야 할 점은 프로세스간에는 메모리 공유가 되지 않습니다.
서로 공유해야할 데이터가 있을 경우에는 다른 방법으로 공유하는 것을 추가구현하여야 합니다.

아래 간단한 예제코드 입니다.
전체 코드가 아니라 실행되는 코드는 아니니 구현방법에 대해서만 참조해 주시기 바랍니다.

```Python
import multiprocessing

def run_server_source():
    ...

def run_server_client():
    ...

if __name__ == "__main__":
    process1 = multiprocessing.Process(target=run_server_source)
    process2 = multiprocessing.Process(target=run_server_client)

    process1.start()
    process2.start()

    process1.join()
    process2.join()
```

### 2. 이벤트 루프(event_loop)를 공유

하나의 프로세스 안에서 이벤트 루프를 공유하는 방법입니다.
단일 프로세스 내에서 동작하기 때문에 데이터를 공유할 일이 있는 경우에는 이 방법이 편리합니다.

이벤트 루프를 공유한다는게 말같지도 않게 들리죠 ?
몇개의 코루틴이든 그냥 `get_event_loop()`를 해서 사용하면 됩니다.
통상적으로 `class` 내부에서 코루틴을 사용하는 경우 내부에서 `get_event_loop()`를 매번 호출해서 사용하는 경우도 있고,
`get_event_loop()`로 반환받은 `loop`를 멤버로 저장한 뒤 계속 사용하는 방법도 있습니다.

이 경우에 `class` 외부에서 `loop`를 주입받아서 사용하도록 `class` 구현을 수정한 뒤 외부에서 `get_event_loop()`로 가져온 `loop`로 코루틴을 사용하는 모든 `class`로 주입을 하면 해당 `class`들이 모두 정상적으로 동작합니다.
그렇게 사용할 모든 `class`에 주입한 다음 `run_forever()`를 한번만 호출하면 됩니다.
너무나도 당연한 말인데, 해당 키워드로 검색 및 질문들이 stackoverflow에 등록되어 있으며, 아주 복잡한 방법으로의 답변들만 등록되어 있더군요.

아래는 간단한 예제 코드입니다.

```Python
from event_server import run_event_server
from client_server import run_client_server
from asyncio import get_event_loop
from log_console import ConsoleClientLogger, ConsoleEventLogger

if __name__ == "__main__":

    loop = get_event_loop()

    client_server = run_client_server(loop, ConsoleClientLogger())
    source_server = run_event_server(loop, client_server, ConsoleEventLogger())
    
    loop.run_forever()
``` 