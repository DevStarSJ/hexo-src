---
title: Using async method in static constructor ( C# )
date: 2016-06-12 02:00:00
categories:
- C#
- C#
tags:
- C#
---

# Using async method in static constructor ( C# )

`static constructor` 내부에서 `async` 함수를 호출할 경우 제대로 동작을 하지 않습니다.  
(왠만해서는 이런식으로 code가 이루어지지 않도록 해야하지만, 어쩔수 없이 이렇게 사용해야 할 경우가 발생 할 수 있습니다.)  

* 참고로 `static constructor`는 해당 `class`가 가장 먼저 사용 될 때 실행됩니다.

### CLR-internal lock

`static constructor`는 정확히 1번만 실행되어야 합니다.  
그러므로 `static constructor`가 실행될 때는 내부적으로 `CLR-internal lock`으로 해당 code를 1번만 실행되도록 수행합니다.  
이렇게 lock이 걸린 상태에서 `async`를 이용하여 다른 `thread`로 작업을 수행할 경우 `thread-lock`과 `CLR-internal lock`간의 `deadlock`이 발생해서 안된다고 생각 할 수 있습니다.

<http://blogs.msdn.com/b/pfxteam/archive/2011/05/03/10159682.aspx> 링크의 posting을 보면 설명이 되어 있습니다.

<http://blog.stephencleary.com/2013/01/async-oop-2-constructors.html> 링크의 posting에도 절대로 `static constructor`에서 `async` 작업을 하는 건 `BAD CODE!!!`라고 경고하고 있습니다.  


참고로 위 Link에 있는 posting에서 안된다고 하는 예제들을 만들어서 해보면 잘 됩니다.  
진짜로 async 한 작업 (DB, Network, disk I/O) 을 이용하는 경우에는 안 될 수 있지만, 단순히 `async` 키워드만 붙였다고 해서 `deadlock`이 재현되지는 않습니다.

### 내가 상상하는 안되는 이유...

위 내용들은 어느정도 검증(?)된 posting을 바탕으로 한 내용이구요.  
제가 생각했을 때 안되는 이유를 말씀드리겠습니다.  
어디까지나 혼자만의 상상의 나래로 기록한 썰(?)이지, 검증된 내용은 아닙니다.  

`constructor`라 함은 object가 생성될때 가장 먼저 해줘야 하는 작업입니다.  
`static constructor`는 해당 class의 `static properties, method`들이 사용되기 전에 먼저 실행되어야 할 code들을 모아둔 곳이어야 겠죠.  
`C++98`까지의 modern하지 않은 `C++`에서는 `member variable` (C# 의 properties) 들을 초기화 하는 작업을 `constructor`에서 했습니다.  
C++11 에서는 C# 과 같이 선언과 동시에 초기값를 바로 써 줄 수 있지만, 아마 동작은 `constuctor`실행시 할 거라 생각됩니다.  
C# 도 편의상 선언과 동시에 초기값을 써 줄 수는 있지만, 아마 동작은 `constuctor` 수행시 할 거라 생각됩니다.  

`static constuctor`는 해당 class의 `static method`보다 먼저 수행되어야 할 code라는 썰을 전제로 생각해 본다면,  
그럼 `static constructor`에서 다른 `static method`를 호출하면 어떻게 될까요 ???  
아마 그 시점에 해당 `static method`의 code를 실행가능 하도록 해주지 않을까 예상됩니다.  
(memory에 load한다던지, 아니면 다른 방법으로 실행가능하게 뭔가 조치를 해주겠죠.)  
이게 동일 thread 내에서는 당연히 판단이 가능하여서 아무런 문제가 없이 동작하지만,  
`static constructor`가 수행중이고 아직 완료되지 않은 시점에,  
갑자기 다른 thread가 해당 class의 다른 `static method`에 접근을 할 경우에는 어떻게 해야 할까요 ?  
아마 `static constructor`가 수행중이니 `CLR-internal lock`으로 보호되고 있겠지요 ?  
이 lock은 `static constructor`가 수행을 종료해야 풀리겠구요.  
그런데 그 다른 thread가 `static method`를 수행완료해야만 `static constructor`가 종료될 수 있다면 ???
여기서 `dead lock`이 발생할 것입니다.  
동일 thread 내에서는 `CLR-internal lock` 내부에서 수행이 되도록 잘 설계 했습니다.
당연히 다른 thread에서 접근은 lock으로 보호해야하는 건 맞구요.  
하지만, 해당 thread가 종료되길 기다리는게 `static constructor`인 경우에는 ???  

그래서 아래에 제가 적어놓은 해결 방법 중에,  
`static constructor`에서 자신의 class가 아닌 다른 class의 `async` 작업을 기다리는 경우는 잘 동작합니다.
이것을 이용해서 `async`한 작업을 별도 class로 나누면 역시나 잘 동작합니다.

### Deadlock in async method in static constuctor

강제로 `deadlock`을 발생시키는 code를 만들어 보았습니다.

```C#
using System.Collections.Generic;
using System.Threading.Tasks;

class StaticClass
{
    public static IEnumerable<string> Names { set; get; }

    static StaticClass()
    {
        Names = Task.Run(async () => { return await GetNamesAsync(); }).Result;
    }

    public static async Task<IEnumerable<string>> GetNamesAsync()
    {
        List<string> nameList = new List<string>
        {
            "Luna", "Star", "Philip"
        };

        return nameList;
    }
}

class Program
{
    static void Main(string[] args)
    {
        foreach (string name in StaticClass.Names)
        {
            System.Console.WriteLine(name);
        }
    }
}
```

이런 code를 어떻게 고쳐야 하는지 3가지 방법을 살펴보겠습니다.

### 1. async한 구현의 method를 추가

위 예제의 경우 `GetNamesAsync()` method와 같은 기능을 하는 sync한 method인 `GetNames()`를 추가하는 방법이 있습니다.  
동일한 구현이 2개가 되므로 별로 추천드리는 방법은 아닙니다.  

참고로 아래 예제도 썩 그렇게 좋은 예제코드는 아닙니다.

```C#
using System.Collections.Generic;
using System.Threading.Tasks;

class StaticClass
{
    public static IEnumerable<string> Names { set; get; }

    static StaticClass()
    {
        Names = GetNames();
    }

    public static async Task<IEnumerable<string>> GetNamesAsync()
    {
        List<string> nameList = new List<string>
        {
            "Luna", "Star", "Philip"
        };

        return nameList;
    }

    public static IEnumerable<string> GetNames()
    {
        List<string> nameList = new List<string>
        {
            "Luna", "Star", "Philip"
        };

        return nameList;
    }
}

class Program
{
    static void Main(string[] args)
    {
        foreach (string name in StaticClass.Names)
        {
            System.Console.WriteLine(name);
        }
    }
}
```

하지만 sync한 작업으로 구현 자체가 될 것을 굳이 `async`로 선언할 일은 잘 없습니다.  
그러므로 이렇게 해결될 수 있는 일이라면 애초에 `async`로 구현한거 자체가 제대로 된 설계가 아닐 수 있습니다.

### 2. async 작업을 별도 class로 분리 (또는 async 작업 호출을 별도 class로 제한)

`async` 작업을 별도 class로 분리하거나,
`static constructor`에서 호출하는 `async` 작업을 다른 class의 method로 제한하는 방법이 있습니다.  
이렇게 구현할 경우 원래 `class`에서 sync한 구현과 async한 구현이 모두 필요할 경우 1번과 같은 code 모양이 될 경우가 많습니다.  
`static constructor`에서 호출하는 `async method`가 다른 class의 method일 경우에는 `deadlock`이 걸리지 않았습니다.  

```C#
using System.Collections.Generic;
using System.Threading.Tasks;

class StaticClass
{
    public static IEnumerable<string> Names { set; get; }

    static StaticClass()
    {
        Names = GetNames();
    }

    public static IEnumerable<string> GetNames()
    {
        return Task.Run(async () => { return await AsyncClass.GetNamesAsync(); }).Result; ;
    }
}

class AsyncClass
{
    public static async Task<IEnumerable<string>> GetNamesAsync()
    {
        List<string> nameList = new List<string>
        {
            "Luna", "Star", "Philip"
        };

        return nameList;
    }
}

class Program
{
    static void Main(string[] args)
    {
        foreach (string name in StaticClass.Names)
        {
            System.Console.WriteLine(name);
        }
    }
}
```

### 3. 초기화 작업을 별도 Init method로 분리

개인적으로 이 방법이 가장 깔끔해 보입니다.  
해당 `class`가 사용되기 전에 `Init()`을 호출한 뒤에 사용하면 됩니다.  
`Init()`함수가 호출되기 전에 이미 해당 `class`의 `static constructor`가 실행된 상태이기 때문에 `CLR-internal lock`은 이미 `unlcok`된 상태에서 `async`작업을 수행하게 됩니다.    
하지만 여러 thread에서 `Init()` 함수가 호출될 가능성이 있을 경우에는 사용자가 별도로 `lock`을 걸어서 호출을 해야 합니다.  
해당 기능은 `clsss`가 최초로 사용되기 이전 시점의 아무 곳에서나 호출이 가능하므로, `lock`이 필요없는 적당한 시점에 호출시켜 주는 것이 좋습니다.  

```C#
using System.Collections.Generic;
using System.Threading.Tasks;

class StaticClass
{
    public static IEnumerable<string> Names { set; get; }

    static StaticClass()
    {
        ...
    }

    public static void Init()
    {
        Names = Task.Run(async () => { return await GetNamesAsync(); }).Result;
    }

    public static async Task<IEnumerable<string>> GetNamesAsync()
    {
        List<string> nameList = new List<string>
        {
            "Luna", "Star", "Philip"
        };

        return nameList;
    }
}

class Program
{
    static void Main(string[] args)
    {
        StaticClass.Init();
        foreach (string name in StaticClass.Names)
        {
            System.Console.WriteLine(name);
        }
    }
}
```
