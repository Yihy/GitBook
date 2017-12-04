---
date : 2017-11-26T13:47:08+02:00
tags : ["java","基础"]
title : 类基本方法（toString、hashCode、equals、compareTo）重写
menu : 
  main: 
    parent: '日常问题'
    weight: 10
---
# 类基本方法（toString、hashCode、equals、compareTo）重写

> 使用jar包为 apache的commons集lang包

## 1.  toString()
每一个对象，在转成String字符串的时候都会调用这个方法

实现一
```java
public class Person {
   String name;
   int age;
   boolean smoker;

   ...

   public String toString() {
     return new ToStringBuilder(this).
       append("name", name).
       append("age", age).
       append("smoker", smoker).
       toString();
   }
 }
```

实现二

```java
public String toString() {
   return ToStringBuilder.reflectionToString(this);
 }
```

## 2.  hashCode()

可以说hashCode和equals是要一起重写的。在比较对象是否相同时，会使用hashCode作为该对象的唯一值。常用场景LIst、Set、Map
```java
public class Person {
  String name;
  int age;
  boolean smoker;
  ...
  public int hashCode() {
    // you pick a hard-coded, randomly chosen, non-zero, odd number
    // ideally different for each class
    return new HashCodeBuilder(17, 37).
      append(name).
      append(age).
      append(smoker).
      toHashCode();
  }
}
```

## 3.  equals(Object obj)
这个方法用来比较传入对象是否与自身相等。
```java
public boolean equals(Object obj) {
  if (obj == null) { return false; }
  if (obj == this) { return true; }
  if (obj.getClass() != getClass()) {
    return false;
  }
  MyClass rhs = (MyClass) obj;
  return new EqualsBuilder()
                .appendSuper(super.equals(obj))
                .append(field1, rhs.field1)
                .append(field2, rhs.field2)
                .append(field3, rhs.field3)
                .isEquals();
 }
```
## 4.  compareTo(Object o)
实现排序接口，主要用于List的排序
```java
public int compareTo(Object o) {
    MyClass myClass = (MyClass) o;
    return new CompareToBuilder()
      .appendSuper(super.compareTo(o)
      .append(this.field1, myClass.field1)
      .append(this.field2, myClass.field2)
      .append(this.field3, myClass.field3)
      .toComparison();
  }
```
