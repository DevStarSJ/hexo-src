---
title: Entity Framework Code First (Table 생성 및 수정)
date: 2016-06-12 02:00:00
categories:
- C#
- C#
tags:
- C#
---

## Entity Framework Code First (Table 생성 및 수정)

### Code First란 ?

전통적인 방식의 경우 SQL을 이용하여 Database에 Table을 생성한 다음 Application에서 개발을 시작합니다.
`Code First`방식이란 `Domain Class`의 명세를 이용하여 Application 실행 시 해당 Table이 없는 경우 자동으로 생성을 해주는 방식을 말합니다.

Entity Framework 4.1 이후부터 지원해주는 방식이며, `Domain Driven Design`의 경우 유용합니다.

![그림 Code First](https://github.com/DevStarSJ/Study/blob/master/Blog/CSharp/EntityFramework/CodeFirst.Migration/image/code-first.png?raw=true)  
<http://www.entityframeworktutorial.net/code-first/what-is-code-first.aspx>



. Modify Models after Scaffolding

### Code First 실습

#### 1. Project 생성 및 Entity Framework 설치

- `Visual Studio`에서 `C# Console Application Project`를 하나 생성합니다.
- `Nuget Package Manager`를 실행합니다. (아래 방법 중 하나로 실행이 가능합니다.)
  - Solution Explorer에서 Project에서 우클릭하여 `Manage Nuget Packages...`
  - 상단 Menu에서 `Tools` -> `Nuget Package Manager` -> `Manage Nuget Packages for Solution...`
- `EntityFramework`를 검색하여 설치합니다. (4.1 이후 버전으로 설치합니다. 이 Post를 작성할 당시 6.1.3 버전이 최신입니다.)

#### 2. Domain Model 정의

아래와 같이 `School` 과 `Standard`라는 2개의 Domain Model을 정의합니다.

```CSharp
class Student
{
    public int       StudentID   { get; set; }
    public string    StudentName { get; set; }
    public DateTime? DateOfBirth { get; set; }
    public byte[]    Photo       { get; set; }
    public decimal   Height      { get; set; }
    public float     Weight      { get; set; }

    public Standard  Standard    { get; set; }
}

class Standard
{
    public int    StandardID   { get; set; }
    public string StandardName { get; set; }

    public ICollection<Student> Students { get; set; }
}
```

`School`은 `Standard`의 참조를 가지고 있으며, `Standard`는 `School`의 집합을 가지고 있습니다.

#### 3. Entity Framework Context 정의

```CSharp
using System.Data.Entity;
```

DbContext를 사용하기 위해서는 `System.Data.Entity`를 `using`해주면 편합니다.

```CSharp
class SchoolContext : DbContext
{
    public DbSet<Student> Students { get; set; }
    public DbSet<Standard> Standards { get; set; }
}
```

2개의 Domain Model을 DbSet Property로 정의합니다.

특정 Database로의 접근을 원할 경우 DbContext의 생성자로 `Connection String`를 전달하도록 생성자를 정의해주면 됩니다.
Console Application Project에서 default일 경우 Visual Studio와 함께 설치된 LocalDB로 연결 됩니다.

#### 4. Context 실행

```CSharp
class Program
{
    static void Main(string[] args)
    {
        using (var context = new SchoolContext())
        {
            Student s = new Student() { StudentName = "New Student" };

            context.Students.Add(s);
            context.SaveChanges();
        }
    }
}
```

Database에 따로 Table을 생성하지 않고 위 Code를 실행하면 자동으로 Table이 생성됩니다.

![그림 Table Created](https://github.com/DevStarSJ/Study/blob/master/Blog/CSharp/EntityFramework/CodeFirst.Migration/image/EF.Migration.01.png?raw=true)  

그림에서 확인되는 것을 보면 `Standard`에는 `Student`에 대한 항목이 없으며,
`Student`에는 `Standard_StandardID`라는 `Foreign Key`가 추가 된 것이 확인됩니다.

개발 또는 운영 중 새로운 Table이 추가 될 경우에는 `Domain Model`을 선언하고 `Context`의 `DbSet Property`를 선언해주면 됩니다.

### Change Models after Scaffolding

#### 1. Domain Model 수정

먼저 `Teacher` Model을 추가한 뒤,

```CSharp
class Teacher
{
    public int TeacherID { get; set; }
    public string TeacherName { get; set; }
}
```

`Student`에 `Teacher`의 참조를 추가합니다.

```CSharp
class Student
{
    public int StudentID { get; set; }
    public string StudentName { get; set; }
    public DateTime? DateOfBirth { get; set; }
    public byte[] Photo { get; set; }
    public decimal Height { get; set; }
    public float Weight { get; set; }

    public Standard Standard { get; set; }
    public Teacher Teacher { get; set; }
}
```

#### 2. Context 실행

`F5`를 눌러서 실행을 하면 오류가 발생합니다.

`Domain Model`의 새로운 `Property`가 추가될 경우에는 오류가 발생합니다.
(Table 입장에서는 Column이 추가되어야 할 경우)

이 경우에는 Migration 이라는 작업을 따로 해 줘야 합니다.

여러 가지 방법이 존재합니다.

1. `Package Manager Console`에서 `Enable-Migrations` 명령어를 이용
2. `Connection String`에서 `Database name`을 변경
3. `Entity Framework`에서 `Database Initializer`를 사용

위 방법 중 3번 방법이 가장 편하므로, 해당 방법만 설명 드리겠습니다.

나머지 방법에 대해서는 아래 Posting을 참조하시기 바랍니다.
<https://blogs.msdn.microsoft.com/webdev/2013/11/01/tips-when-making-changes-in-entity-framework-code-first-models-after-scaffolding>

#### 3. `Database Initializer` 선언

```CSharp
class SchoolInitializer : DropCreateDatabaseIfModelChanges<SchoolContext>
{
    protected override void Seed(SchoolContext context)
    {
        base.Seed(context);
    }
}
```

별도의 작업이 필요한 경우 `Seed()` 안에서 정의를 해주면 됩니다.
현재는 별 다른 작업이 필요 없으므로 `Seed()`를 `Override`하지 않고 삭제해도 됩니다.

#### 4. `Database Initializer` 실행

Application 실행시 `Database Initializer`를 먼저 실행시켜 주면 오류 없이 Table이 수정됩니다. 
심지어 DbSet Property에 Teacher를 추가하지 않았는데도 불구하고 Teacher Table이 추가되었습니다.

```CSharp
class Program
{
    static void Main(string[] args)
    {
        Database.SetInitializer<SchoolContext>(new SchoolInitializer());

        using (var context = new SchoolContext())
        {
            Student s = new Student() { StudentName = "New Student" };

            context.Students.Add(s);
            context.SaveChanges();
        }
    }
}
```

일단 오류없이 실행은 되었습니다. 
Database를 확인해 보면 Table 정보가 수정된 것을 볼 수 있습니다.

![그림 Table Modified](https://github.com/DevStarSJ/Study/blob/master/Blog/CSharp/EntityFramework/CodeFirst.Migration/image/EF.Migration.02.png?raw=true)  

