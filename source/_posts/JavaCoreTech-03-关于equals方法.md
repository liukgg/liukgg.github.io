---
title: JavaCoreTech-03-关于equals方法
date: 2021-08-08 20:00:55
tags: Java
categories: Java
---

关于equals方法
-----------

>参考 Java核心技术卷I 第5章 继承 5.2.2 equals方法

### equals方法概述
Object类中的euqals方法用于检测一个对象是否等于另外一个对象。

equals方法与==比较：

- ==来判断两个变量是否相等，但是会根据数据类型有所区别：
  - 对于8种基础数据类型（byte、short、int、long、double、float、boolean、char）来说==是判断变量的数值是否相等。
  - 对于引用类型，==比较的是引用的地址是否相等 
- Object.equals里的内部实现，其实还是调用的==
- 在自定义的类中一般会重写 Object.equals 方法，这时一般是根据业务逻辑来判断两个对象是否相等、而不是直接看两者是否是同一个对象。

### 关于 Object, String, Array 类中equals方法源码
根据Java API官方文档可知，Object.equals 方法实际上是判断两个对象是否是同一个对象。
也就是说，对于任意两个非空的对象（引用）x和y，当且仅当x和y指向同一个对象时才会返回true。
实际上，从源码可知， Object.equals 方法本质上就是判断 x == y。

结论：对于自定义的类，一般来说都需要重定义 equals 方法以覆盖Object.equals方法。
例如，比较常见的一种做法是，当类存在数据库且有主键ID的时候，经常会通过判断两个对象的ID是否相等来比较是否相等。

> String 类中重写了 equals 方法
> 声明一个整型数组，查看其equals方法，会发现实际上就是 Object.equals 方法
> java.util.Arrays 中实现了多个不同参数的equals方法，例如 equals(int [] a, int[] b)，但是这已经和Object.equals方法有本质区别了。

- 以下为JDK 11中Object.equals方法的源码:(JDK 1.8 相同)
> 参考： https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html

```java
/**
 * reference： https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html
 * 
 * Indicates whether some other object is "equal to" this one.
 * 
 * The equals method for class Object implements the most discriminating possible equivalence relation on objects; 
 * that is, for any non-null reference values x and y, 
 * this method returns true if and only if x and y refer to the same object (x == y has the value true).
 * 
 * @param   obj   the reference object with which to compare.
 * @return  {@code true} if this object is the same as the obj
 *          argument; {@code false} otherwise.
 */
public class Object{
    public boolean equals(Object obj) {
        return (this == obj);
    }
}
```

- String.equals方法源码：

```java
public final class String
        implements java.io.Serializable, Comparable<String>, CharSequence {
/**
 * Compares this string to the specified object.  The result is {@code
 * true} if and only if the argument is not {@code null} and is a {@code
 * String} object that represents the same sequence of characters as this
 * object.
 *
 * <p>For finer-grained String comparison, refer to
 * {@link java.text.Collator}.
 *
 * @param  anObject
 *         The object to compare this {@code String} against
 *
 * @return  {@code true} if the given object represents a {@code String}
 *          equivalent to this string, {@code false} otherwise
 *
 * @see  #compareTo(String)
 * @see  #equalsIgnoreCase(String)
 */
public boolean equals(Object anObject) {
    if (this == anObject) {
      return true;
    }
    if (anObject instanceof String) {
      String aString = (String)anObject;
      if (coder() == aString.coder()) {
        return isLatin1() ? StringLatin1.equals(value, aString.value)
                : StringUTF16.equals(value, aString.value);
      }
    }
    return false;
  }
}

final class StringLatin1 {
  @HotSpotIntrinsicCandidate
  public static boolean equals(byte[] value, byte[] other) {
    if (value.length == other.length) {
      for (int i = 0; i < value.length; i++) {
        if (value[i] != other[i]) {
          return false;
        }
      }
      return true;
    }
    return false;
  }
}

// final class StringUTF16 类中实现类似，不再赘述。
```

### 自定义equals方法代码示例

### 如何自定义equals方法

### 关于String判断相等
注意事项：
可以用equals方法检测两个字符串是否相等。例如：

```java 
String s = "hello";
String t = "hello";
s.equals(t); // true
s.subString(0, 3).equals("hel"); // true
```

一定不要使用 == 运算符检测两个字符串是否相等！这个运算符只能够确定两个字符串是否存放在同一个位置上。当然，如果字符串在同一个位置上，它们必然相等。
但是，完全有可能将内容相同的多个字符串副本放置在不同的位置上。

```java
String greeting = "hello";
greeting == "hello"; // true
greeting.subString(0, 3) == "hel"; //false
```

说明：
如果虚拟机始终将相同的字符串共享，就可以使用==运算符检测是否相等。但实际上只有字符串字面量是共享的，而 + 或 subString 等操作得到的字符串并不
共享。因此，千万不要使用 == 运算符测试字符串的相等性，以免在程序中出现这种最糟糕的bug，看起来这种bug就像随机产生的间歇性错误。