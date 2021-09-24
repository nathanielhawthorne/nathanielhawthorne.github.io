---
layout:     post
title:      2021-09-24-note-Member variables, class variables, and local variables in java
subtitle:   dream
date:       2021-09-24
author:     xiong
header-img: img/post-bg-fa.jpg
catalog: true
tags:
    - Xiong
---


# 2021-09-24-note-Member variables, class variables, and local variables in java
## 1.Member variables(Instance variable)

Defined in the class, outside the method body. The variable is instantiated when the object is created. Member variables can be accessed by class methods, construction methods, and statement blocks of a specific class.

```java
public class  ClassName{
    int a;
    public void printNumber（）{
        // other code
    }
}
```



## 2.Class variable(static variable)

Defined in the class, outside the method body, but there must be static to declare the variable type. Static members belong to the entire class and can be called by object name or class name.

```java
public class  ClassName{
    static int a;
    public void printNumber（）{
        // other code
    }
}
```

## 3.Local variable

Variables defined in methods, construction methods, and statement blocks. Its declaration and initialization are implemented in the method, and it is automatically destroyed after the method ends

```java
public class  ClassName{
    public void printNumber（）{
        int a;
    }
    // other code
}
```

## 4.The difference between member variables and class variables:

### 1.The life cycles of the two variables are different

+ Member variables exist as the object is created, and released as the object is recycled.
+ Static variables exist as the class is loaded, and disappear as the class disappears.

### 2.Different calling methods

+ Member variables can only be called by objects.
+ Static variables can be called by objects, and they can also be called by class names.

### 3.Alias are different

+ Member variables are also called instance variables.
+ Static variables are also called class variables.

### 4.Data storage location is different

+ Member variables are stored in objects in heap memory, so they are also called object-specific data.
+ Static variable data is stored in the static area of the method area (shared data area), so it is also called the shared data of the object.

## 5.feature

1. I want to realize object sharing of common data among objects. This data can be statically modified.
2. Members that are statically modified can be called directly by the class name. In other words, static members have one more calling method. Class name. Static mode.
3. The static is loaded with the loading of the class. And it takes precedence over object existence.

