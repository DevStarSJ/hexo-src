---
title: C# Attribute 상속 후 override 할 경우 부모 값도 잘 가져오는지 확인
date: 2016-08-10 02:00:00
categories:
- C#
- C#
tags:
- C#
---

# C# Attribute 상속 후 override 할 경우 부모 값도 잘 가져오는지 확인

C# Attribute class 생성 및 사용, 상속시에 동작 등에 대해서 확인 가능한 예제코드 입니다.

코드 다운로드 : <https://github.com/DevStarSJ/Study/tree/master/Blog/CSharp/AttributeInheritTest>

## Step 1. Attribute 존재여부 검사

#### AttributeInheritTest.01.cs
```C#
using System;

public class Test1Attribute : Attribute
{
    public int t1 { get; set; }
    public Test1Attribute(int t)
    {
        t1 = t;
    }
}

public class Test2Attribute : Attribute
{
    public int t2 { get; set; }
    public Test2Attribute(int t)
    {
        t2 = t;
    }
}

public class Parent
{
    [Test1(1)]
    [Test2(2)]
    public int id { get; set; }
}

class Program
{
    static void Main(string[] args)
    {
        var propertyInfo = typeof(Parent).GetProperty(nameof(Parent.id));

        var attributes = Attribute.GetCustomAttributes(propertyInfo, true);
        foreach (var a in attributes)
        {
            Console.WriteLine(a.GetType().ToString());
        }
    }
}
```

```
Test2Attribute
Test1Attribute
```

## Step 2. Attribute 내부 값 사용하기

`foreach` 내부만 수정

#### AttributeInheritTest.02.cs
```C#
foreach (var a in attributes)
{
    Console.WriteLine(a.GetType().ToString());

    if (a.GetType() == typeof(Test1Attribute))
    {
        Console.WriteLine((a as Test1Attribute).t1);
    }
    else if (a.GetType() == typeof(Test2Attribute))
    {
        Console.WriteLine((a as Test2Attribute).t2);
    }
}
```

```
Test2Attribute
2
Test1Attribute
1
```

## Step 3. 상속받은 후 Attribute 존재여부 검사 및 값 사용하기

`Child` 정의 및 `propertyInfo` 생성 부분만 수정

#### AttributeInheritTest.03.cs
```C#
public class Child : Parent
{
}

var propertyInfo = typeof(Child).GetProperty(nameof(Child.id));
```

```
Test2Attribute
2
Test1Attribute
1
```

## Step 4. 상속받은 후 property를 override한 후 Attribute 재정의

재정의한 Attribute만 바뀌었고, 나머지는 부모의 것을 가져오는 것을 확인 할 수 있습니다.

`Parent`의 `id`를 `virtual`처리 하였고, `Child`의 `id`를 `override`한 후 `Test2Attribute`만 재정의 하였습니다.

#### AttributeInheritTest.04.cs
```C#
public class Parent
{
    [Test1(1)]
    [Test2(2)]
    public virtual int id { get; set; }
}

public class Child : Parent
{
    [Test2(3)]
    public override int id { get; set; }
}
```

```
Test2Attribute
3
Test1Attribute
1
```
