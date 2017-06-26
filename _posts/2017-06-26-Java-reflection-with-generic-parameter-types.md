---
layout: post
title:  "Java reflection with generic parameter types"
date:   2017-06-26 22:37:35
---
I was dealing with reflection and generic types this week, and I had a lot of problems due Java compiler and how it works with generic types.

I have an aspect to update metrics of some methods and I wanted to add an annotation to some methods to put an specific name in that metrics. Everything was working fine, since I found that some of my methods were not populating the metrics with the names that I expected.

To simplify let´s think in the following code:
```java
public class FooImpl implements Foo<Pojo> {

    @FooAnnotation(value = "doSomethingWithoutParams")
    public void doSomething() {}

    @FooAnnotation(value = "doSomethingWithGenericType")
    public void doSomething(Pojo pojo) {}
}
```

```java
@Around("execution(* *.doSomething(..))")
public void aspect(ProceedingJoinPoint joinPoint) throws Throwable {

    [...]
    joinPoint.proceed();
}
```

I want to retrieve the value of the @FooAnnotation within the aspect. Java does not let you retrieve this easily.

In Java, you can see the generic types class from the caller, but not inside the class that has the generic types. Quoting from StackOverflow a good and easy explanation:
```
You CAN'T see the "T" here:
class MyClass<T> { T myField; }

You CAN see the "T" here:
class FooClass {
    MyClass<? extends Serializable> fooField;
}
```

So, after dealing with lots of code, I realized that the only thing I can do was to retrieve the method, and the target class, and with the arguments stored in the joinPoint, do some kind of matching.

But this matching is something complex, because if your parameter is implementing an interface, you will need to check all the interfaces and match with all the declared methods of your target class that has the same name and number of params than your method.

I was trying to use Spring ClassUtils and the getMostSpecificMethod(Method, Class) method, but finishing with the same results. But then, don´t know how, I´ve seen AopUtils, also from Spring.

This AopUtils match the generic params perfectly with the method you are searching for. After all, the solution is quite easy and small.

```java
@Around("execution(* *.doSomething(..))")
public void aspect(ProceedingJoinPoint joinPoint) throws Throwable {

    Method method = ((MethodSignature) joinPoint.getSignature()).getMethod();
    Assertion.result = AopUtils.getMostSpecificMethod(method, joinPoint.getTarget().getClass()).getAnnotation(FooAnnotation.class).value();
    joinPoint.proceed();
}
```

To use AopUtils you only need org.springframework:spring-aop module, which if you are using aspects for sure you are already importing it.

You can check everything with a working example in my [java-examples project in github](https://github.com/caay2000/java-examples). Search for the reflection.generics package.