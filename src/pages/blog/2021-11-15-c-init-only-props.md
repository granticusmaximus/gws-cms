---
template: blog-post
title: C# Init Only Props
slug: csharp-init-only-props
date: 2021-08-13 21:51
description: In this article, I am going to explain Init-only setter property
  which has been introduced in C# 9.0.
featuredImage: /assets/unsplash-image-ke0nc8-58mq.jpg
---
In this article, I am going to explain Init-only setter property which has been introduced in C# 9.0.

Each C# release has its own features. In C# 9.0, many new wonderful features have been added and here we are going to discuss important feature Init-only setter property.

Before we start, we should have basic understanding of immutable and mutable in c#.

What’s meaning of Mutable and Immutable in English?

Answers are:

* Mutable – Can change
* Immutable – Cannot Change

In simple words we can say, immutable means object cannot be changed once it is created. If you want to change, you have to create new object/ assign new memory. Example of immutable type is string in C#.

## What is Init-only setter property?

Before we start with Init-only property, we have to understand the traditional way of working (properties).

Let’s start with an example,

Suppose, we have Member class which has 3 properties,

1. ID - Member unique ID
2. Name - Member Name
3. Address - Member Address

```csharp
public class Member
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Address { get; set; }
}
```

Now create object of Member class and assign some values,

```csharp
using System;

namespace C_9._0
{
    class Program
    {
        static void Main(string[] args)
        {
            Member memberObj = new Member();
            memberObj.Id= 1;
            memberObj.Name="Kirtesh Shah";
            memberObj.Address="Vadodara";

            Console.WriteLine("****************START - Member Details***********");

            Console.WriteLine("ID : -" + memberObj.Id);
            Console.WriteLine("Name :- " + memberObj.Name);
            Console.WriteLine("Address:- " + memberObj.Address);
            Console.ReadLine();

            Console.WriteLine("****************END - Member Details***********");
        }
    }
}
```

All looks good right? But what happens if someone changed Member ID? ID is unique and it should not be editable, which means it should be immutable, not mutable. That's the problem with the traditional way, properties are mutable.\
\
If we want to make properties immutable in the traditional way, we have to pass values in constructor like the below code,

```csharp
public class Member
{
    public Member(int memberId)
    {
        Id = memberId;
    }
    public int Id { get; }
    public string Name { get; set; }
    public string Address { get; set; }
}
```

ID property is immutable now. Let's see the below code:

```csharp
using System;
namespace C_9._0 {
    class Program {
        static void Main(string[] args) {
            Member memberObj = new Member(1);
            memberObj.Name = "Kirtesh Shah";
            memberObj.Address = "Vadodara";
            Console.WriteLine("****************START - Member Details***********");
            Console.WriteLine("ID : " + memberObj.Id);
            Console.WriteLine("Name : " + memberObj.Name);
            Console.WriteLine("Address: " + memberObj.Address);
            Console.ReadLine();
            Console.WriteLine("****************END - Member Details***********");
        }
    }
}
```

In C# 9.0, we can achieve the same thing using Init-only property.

```csharp
public class Member {
    public int Id {
        get;
        init;
    } // set is replaced with init
    public string Name {
        get;
        set;
    }
    public string Address {
        get;
        set;
    }
}

using System;
namespace C_9._0 {
    class Program {
        static void Main(string[] args) {
            Member memberObj = new Member {
                Id = 1
            };
            memberObj.Name = "Kirtesh Shah";
            memberObj.Address = "Vadodara";
            Console.WriteLine("****************START - Member Details***********");
            Console.WriteLine("ID : " + memberObj.Id);
            Console.WriteLine("Name : " + memberObj.Name);
            Console.WriteLine("Address: " + memberObj.Address);
            Console.ReadLine();
            Console.WriteLine("****************END - Member Details***********");
        }
    }
}
```

It is the power of Init-only property. What will we do, if we want to do some validation like ID should not be null or string? Init-only property has the capability to set read only property to implement validations.

Suppose we have Member class like below,

```csharp
public class Member
{
    public int Id { get; init; }
    public string Name { get; init; }
    public string Address { get; init; }
}
```

Now we will try to create a Member class object like below,

```csharp
using System;

namespace C_9._0
{
    class Program
    {
        static void Main(string[] args)
        {
            Member memberObj = new Member
            {
                  Id =1
            };
            Console.WriteLine("****************START - Member Details***********");

            Console.WriteLine("ID : " + memberObj.Id);
            Console.WriteLine("Name : " + memberObj.Name);
            Console.WriteLine("Address: " + memberObj.Address);
            Console.ReadLine();

            Console.WriteLine("****************END - Member Details***********");
        }
    }
}
```

Init-only properties can or cannot be set as per your requirement. As you notice in the above code, only ID property is set and name and address properties are not set. Please note that if any property is not set at the time of object creation that property cannot be set.

I hope you enjoyed this article.