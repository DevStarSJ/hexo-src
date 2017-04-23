---
title: Newtonsoft.Json 사용법
date: 2016-06-12 02:00:00
categories:
- C#
- C#
tags:
- C#
---

# Newtonsoft.Json 사용법

C# 에서 JSON document를 다루기 위해 가장 많이 사용되는 것은 `Newtonsoft.Json`입니다.
nuget manager에서 `JSON`으로 검색시 가장 먼저 나옵니다.
그만큼 많이 사용되며, 사용법 또한 간단합니다.


##1. 설치 및 namespace

솔루션 탐색기 (Solution Explorer)에서 마우스 우클릭 하신뒤 `Manage nuget packages...`을 누르셔서 `Browse` 탭에서 `Newtonsoft.Json`을 검색하셔서 `Install`을 누르면 됩니다.

다른 방법으로는 도구(Tools) -> `Nuget package manager` -> `Package Manager Console` 로 가셔서 아래와 같이 입력하시면 됩니다.

```
PM> Install-Package Newtonsoft.Json
```

사용시 소스코드에서 아래의 `namespace`를 추가해 주시면 됩니다.

```C#
using Newtonsoft.Json.Linq;
```

## 2. 간단한 특징 설명

2개의 Object를 이용해서 사용하시면 됩니다.

- `JObject` : JSON Object 입니다.
  - `JObject` 자체가 name값을 가질 수는 없습니다.
  - (key, value) pair 들을 가질 수 있습니다. 
  - key : string 값입니다.
  - value : `JToken` 타입이며 대부분의 premitive type들과 DateTime, TiemSpan, Uri 값을 직접대입 가능하며, 기타 Object도 입력이 가능합니다.
    - value에 다른 JObject나, JArray를 넣을 수 있습니다.

- `JArray` : JSON Array 입니다.
  - `JObject`와 특징이 거의 비슷하나 key 없이 value 들을 가지고 있습니다.

즉, `JObject`나 `Jarray` 자체는 `name`을 가질 수 없으나, 다른 `JObject`에 `value`로 소속될 경우에는 `key`값을 가져야 하며, 다른 `JArray`에 소속될 경우에는 `key`값 없이 입력됩니다

## 3. JObject 사용법

너무나 간단하기 때문에 별도 설명은 필요 없을듯 합니다.

- 생성 : `new JObject()`
- Element 추가 : `.add(key, value)`

바로 예제를 보도록 하겠습니다.

### 3.1 Element 추가

#### 3.1.1 기본적인 사용법

```C#
var json = new JObject();
json.Add("id", "Luna");
json.Add("name", "Silver");
json.Add("age", 19);

Console.WriteLine(json.ToString());
```
```JSON
{
  "id": "Luna",
  "name": "Silver",
  "age": 19
}
```

#### 3.1.2 JSON 형식의 문자열로 생성

```C#
var json2 = JObject.Parse("{ id : \"Luna\" , name : \"Silver\" , age : 19 }");
json2.Add("blog", "devluna.blogspot.kr");

Console.WriteLine(json2.ToString());
```
```JSON
{
  "id": "Luna",
  "name": "Silver",
  "age": 19,
  "blog": "devluna.blogspot.kr"
}
```

#### 3.1.3 다른 class Object로부터 생성

```C#
User u = new User { id = "SJ", name = "Philip", age = 25 };
var json3 = JObject.FromObject(u);

Console.WriteLine(json3.ToString());
```

```JSON
{
  "id": "SJ",
  "name": "Philip",
  "age": 25
}
```

#### 3.1.4 무명형식으로 생성

```C#
var json4 = JObject.FromObject(new { id = "J01", name = "June", age = 23 });

Console.WriteLine(json4.ToString());
```

```JSON
{
  "id": "J01",
  "name": "June",
  "age": 23
}
```

#### 3.1.5 다른 JObject를 Element로 추가

```C#
var json5 = JObject.Parse("{ id : \"sjy\" , name : \"seok-joon\" , age : 27 }");
json5.Add("friend1", json);
json5.Add("friend2", json2);
json5.Add("friend3", json3);
json5.Add("friend4", json4);

Console.WriteLine(json5.ToString());
```

```JSON
{
  "id": "sjy",
  "name": "seok-joon",
  "age": 27,
  "friend1": {
    "id": "Luna",
    "name": "Silver",
    "age": 19
  },
  "friend2": {
    "id": "Luna",
    "name": "Silver",
    "age": 19,
    "blog": "devluna.blogspot.kr"
  },
  "friend3": {
    "id": "SJ",
    "name": "Philip",
    "age": 25
  },
  "friend4": {
    "id": "J01",
    "name": "June",
    "age": 23
  }
}
```

### 3.2 Element값 사용하기

#### 3.2.1 Element값 읽기

`[ ]` 연산자에 key값을 넣어주면 해당 value를 얻을 수 있습니다.

```C#
var json4_name = json4["name"];

Console.WriteLine(json4_name);
```

```
June
```

#### 3.2.2 Element값 삭제하기

`.Remove(key)`를 이용해서 삭제가 가능합니다.  

```C#
json4.Remove("name");

Console.WriteLine(json4.ToString());
```

```JSON
{
  "id": "J01",
  "age": 23
}
```

`.RemoveAll()`로 모든 Element를 다 삭제 할 수도 있습니다.

```C#
json5.RemoveAll();

Console.WriteLine(json5.ToString());
```

```JSON
{}
```

## 4. JArray 사용법

Element 입력시 key를 가지지 않는 다는 것을 빼고는 JObject와 거의 같습니다.

### 4.1 Element 추가하기

#### 4.1.1 기본적인 사용법

```C#
var jarray = new JArray();
jarray.Add(1);
jarray.Add("Luna");
jarray.Add(DateTime.Now);

Console.WriteLine(jarray.ToString());
```

```JSON
[
  1,
  "Luna",
  "2016-05-21T09:45:27.1049839+09:00"
]
```

### 4.1.2 JObject를 Element로 추가

```C#
var jFriends = new JArray();
jFriends.Add(json);
jFriends.Add(json2);
jFriends.Add(json3);
jFriends.Add(json4);

Console.WriteLine(jFriends.ToString());
```

```JSON
[
  {
    "id": "Luna",
    "name": "Silver",
    "age": 19
  },
  {
    "id": "Luna",
    "name": "Silver",
    "age": 19,
    "blog": "devluna.blogspot.kr"
  },
  {
    "id": "SJ",
    "name": "Philip",
    "age": 25
  },
  {
    "id": "J01",
    "age": 23
  }
]
```

#### 4.1.3 JArray를 Element로 추가

```C#
var jarray2 = new JArray();
jarray2.Add(jarray);
jarray2.Add(jFriends);

Console.WriteLine(jarray2.ToString());
```

```JSON
[
  [
    1,
    "Luna",
    "2016-05-21T09:51:03.2882071+09:00"
  ],
  [
    {
      "id": "Luna",
      "name": "Silver",
      "age": 19
    },
    {
      "id": "Luna",
      "name": "Silver",
      "age": 19,
      "blog": "devluna.blogspot.kr"
    },
    {
      "id": "SJ",
      "name": "Philip",
      "age": 25
    },
    {
      "id": "J01",
      "age": 23
    }
  ]
]
```

### 4.2 Element값 사용하기

#### 4.2.1 Element값 읽기

`[ ]` 연산자로 읽을 수 있습니다.

```C#
var jf0 = jFriends[0];

Console.WriteLine(jf0.ToString());
```

```JSON
{
  "id": "Luna",
  "name": "Silver",
  "age": 19
}
```

`for` , `foreach`로 iteration이 가능합니다.
```C#
foreach(JObject fElement in jFriends)
{
    var fName = fElement["name"] ?? "<NULL>";
    Console.WriteLine(fName);
}
```

```
Silver
Silver
Philip
<NULL>
```

#### 4.2.2 Element값 삭제하기

```C#
jFriends.Remove(jFriends[1]);
jFriends.Remove(jFriends[2]);

Console.WriteLine(jFriends.ToString());
```

```JSON
[
  {
    "id": "Luna",
    "name": "Silver",
    "age": 19
  },
  {
    "id": "SJ",
    "name": "Philip",
    "age": 25
  }
]
```

## 5. JObject에 JArray 추가하기

```C#
json2.Add("Friends", jFriends);

Console.WriteLine(json2.ToString());
```

```JSON
{
  "id": "Luna",
  "name": "Silver",
  "age": 19,
  "blog": "devluna.blogspot.kr",
  "Friends": [
    {
      "id": "Luna",
      "name": "Silver",
      "age": 19
    },
    {
      "id": "SJ",
      "name": "Philip",
      "age": 25
    }
  ]
}
```
