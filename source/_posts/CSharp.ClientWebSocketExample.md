---
title: Client WebSocket Example without ASP.NET
date: 2016-08-31 02:00:00
categories:
- C#
- C#
tags:
- C#
---

# Client WebSocket Example without ASP.NET

ASP.NET을 사용하지 않고 일반적인 C# (Console, Winform) 에서 WebSocket Server에 접속하는 코드 예제 입니다.

결론만 말씀드리자면... `WebSocketSharp` 만 사용하세요.

<https://github.com/sta/websocket-sharp>

그래도 나머지의 사용법도 설명해 드리겠습니다.

## 1. WebSocketSharp

`Nuget Manager`에서 `WebSocketSharp`를 `include prerelease`를 체크한 상태에서 검색해서 설치하세요.

```CSharp
using WebSocketSharp;

WebSocket ws = new WebSocket(url: "ws://localhost:5000");

ws.Connect();

ws.Send(new byte[] { 0x01, 0x02, 0x03 });

ws.OnMessage += (sender, e) =>
{
    Console.WriteLine(e.Data); // string
    byte[] data = e.RawData // byte []
};
```

## 2. WebSocketSharp-clone 사용

nuget manager에서 `websocket-sharp.clone` 로 검색하면 나옵니다.

`WebSocketSharp`에 비해 사용하기 조금 더 까다롭습니다.
하지만, 비동기(`async`)처리를 지원해 줍니다.

`OnMessage` 이벤트를 생성자에게 입력해줘야 하는데, 타입이 까다로워서 쉽지 않습니다.
(저도 구현해보려고 시도하다가 귀찮아서 관뒀습니다.)

```CSharp
using WebSocketSharp;

WebSocket ws = new WebSocket(url: "ws://localhost:5000");

await ws.Connect();

Console.WriteLine(ws.ReadyState); // Open

bool result = await ws.Send(new byte[] { 0x03, 0x01, 0x10, 0xFF });

Console.WriteLine(result.ToString()); // true
```

## 3. System.Net.WebSockets 사용

별도의 nuget 설치 필요없이 사용이 가능합니다.

`SendAsync()`가 실행 중 다시 실행하면 오류가 발생합니다.
그러니 별도의 `Lock`작업을 해주던지 메세지 전송을 `Queue`처리해서 한 곳에서만 동작하게 구현을 해줘야 합니다.

`ReceiveAsync()`를 호출해서 값을 읽어야 하므로, 구현하기에 불편함이 있습니다.
해당 함수를 이용해서 `Event`를 만들어서 구현을 해주면 편리하게 사용이 가능할거라 생각됩니다.
귀찮아서 저도 그렇게 구현해 보려다 말았습니다.

```CSharp
using System.Net.WebSockets;

ClientWebSocket ws = new ClientWebSocket();
Uri uri = new Uri("ws://localhost:5000");

await ws.ConnectAsync(uri, CancellationToken.None);

Console.WriteLine(ws.State); // open

var segment = new ArraySegment<byte>(new byte[] { 0x03, 0x01, 0x10, 0xFF });
await ws.SendAsync(segment, WebSocketMessageType.Binary, true, CancellationToken.None);
```
