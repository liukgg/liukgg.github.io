---
title: JavaCoreTech-03-关于equals方法
date: 2021-08-08 20:00:55
tags: Java
categories: Java
---

关于equals方法
-----------

> 参考 Java核心技术卷I 第5章 继承 5.2.2 equals方法
> 参考: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html
> 参考: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html

### equals方法概述
Object类中的euqals方法用于检测一个对象是否等于另外一个对象。

equals方法与 == 的区别与联系：

- == 用来判断两个变量是否相等，但是会根据数据类型有所区别：
  - 对于8种基础数据类型（byte、short、int、long、double、float、boolean、char）来说, == 是判断变量的数值是否相等。
  - 对于引用类型，== 比较的是引用的地址是否相等。 
- Object.equals 里的内部实现，其实还是调用的==.
- 在自定义的类中一般会重写 Object.equals 方法，这时一般是根据业务逻辑来判断两个对象是否相等、而不是直接看两者是否是同一个对象。

### 阅读学习 Object, String, Array 类中equals方法源码
根据Java API官方文档可知，Object.equals 方法实际上是判断两个对象是否是同一个对象。
也就是说，对于任意两个非空的对象（引用）x和y，当且仅当x和y指向同一个对象时才会返回true。
实际上，从源码可知， Object.equals 方法本质上就是判断 x == y。

结论：对于自定义的类，一般来说都需要重定义 equals 方法以覆盖Object.equals方法。
例如，比较常见的一种做法是，当类存在数据库且有主键ID的时候，经常会通过判断两个对象的ID是否相等来比较是否相等。

> String 类中重写了 equals 方法；
> 声明一个整型数组，查看其equals方法，会发现实际上就是 Object.equals 方法;
> java.util.Arrays 中实现了多个不同参数的equals方法，例如 equals(int [] a, int[] b)，但是这已经和Object.equals方法有本质区别了。

- 以下为JDK 11中 Object.equals 方法的源码: (JDK 1.8 相同)
> 参考: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html

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

- String.equals 方法源码：

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

### String 判断相等的方法与注意事项
可以用equals方法检测两个字符串是否相等。例如：

```java 
String s = "hello";
String t = "hello";
s.equals(t); // true
s.subString(0, 3).equals("hel"); // true
```

一定不要使用 == 运算符检测两个字符串是否相等！
这个运算符只能够确定两个字符串是否存放在同一个位置上。
当然，如果字符串在同一个位置上，它们必然相等。
但是，完全有可能将内容相同的多个字符串副本放置在不同的位置上。

```java
String greeting = "hello";
greeting == "hello"; // true
greeting.subString(0, 3) == "hel"; //false
```

说明：
如果虚拟机始终将相同的字符串共享，就可以使用==运算符检测是否相等。
但实际上只有字符串字面量是共享的，而 + 或 subString 等操作得到的字符串并不共享。
因此，千万不要使用 == 运算符测试字符串的相等性，以免在程序中出现这种最糟糕的bug，看起来这种bug就像随机产生的间歇性错误。

### 自定义equals方法代码示例
为了更好地展示equals方法的定义，Employee类做了一些简化，只保留了id和name两个字段，
实际业务场景下一般会有更多字段，但是判断是否相等一般还是比较id。

从以下代码示例可以看出，实际上自定义类的equals方法定义和String中的equals方法定义是类似的。
因此，JDK中实际上有大量优秀源码实现，我们可以经常去探索和学习。
例如，除了 String.equals 方法，还有一个很值得学习的是 Arrays.sort方法，里面对于排序算法的选择和实现很精巧、值得推敲和学习。

```java
public class Employee {
  private  int id;
  private String name;

  public Employee(int id, String name) {
    this. id = id;
    this.name = name;
  }

  public boolean equals(Object otherObject) {
    if (this == otherObject) return true;
    if (otherObject == null) return false;
    if (getClass() != otherObject.getClass()) return false;

    Employee other = (Employee) otherObject;

    return id == other.id;
  }
}

public class EmployeeTest {
  public static void main(String[] args) {
    Employee a = new Employee(1, "abc");
    Employee b = new Employee(1, "abc");
    Employee c = new Employee(2, "abc");
    System.out.println(a.equals(b)); // true
    System.out.println(a.equals(c)); // false
    System.out.println(a == b); // false, a 和 b的id相等，所以根据我们的equals方法定义来看是相等的；但是==比较的是引用地址，由于a和b不是同一个对象，所以返回false
    System.out.println(a == c); // false
  }
}

```

### 如何自定义equals方法
##### Java语言规范要求
Java语言规范要求equals方法具有以下特性：
-  1 ) 自反性: 对于任何非空引用 x, x.equals(x) 应该返回 true.
-  2 ) 对称性: 对于任何引用 x 和 y, 当且仅当 y.equals(x) 返回 true, x.equals(y) 也应该返回 true。
-  3 ) 传递性: 对于任何引用 x、 y 和 z, 如果 x.equals(y) 返 true， y.equals(z) 返回 true, x.equals(z) 也应该返回 true。
-  4 ) 一致性: 如果 x 和 y 引用的对象没有发生变化， 反复调用 x.equals(y) 应该返回同样的结果。
-  5 ) 对于任意非空引用 x, x.equals(null) 应该返回 false.

  这些规则十分合乎情理， 从而避免了类库实现者在数据结构中定位一个元素时还要考虑调用 x.equals(y), 还是调用 y.equals(x) 的问题。

这些内容在 Object.euqals 方法的文档中也有提到：

> https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html
> 
> public boolean equals(Object obj)
> Indicates whether some other object is "equal to" this one.
> The equals method implements an equivalence relation on non-null object references:
> 
> It is reflexive: for any non-null reference value x, x.equals(x) should return true.
> It is symmetric: for any non-null reference values x and y, x.equals(y) should return true if and only if y.equals(x) returns true.
> It is transitive: for any non-null reference values x, y, and z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) should return true.
> It is consistent: for any non-null reference values x and y, multiple invocations of x.equals(y) consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified.
> For any non-null reference value x, x.equals(null) should return false.
> The equals method for class Object implements the most discriminating possible equivalence relation on objects; that is, for any non-null reference values x and y, this method returns true if and only if x and y refer to the same object (x == y has the value true).
> 
> Note that it is generally necessary to override the hashCode method whenever this method is overridden, so as to maintain the general contract for the hashCode method, which states that equal objects must have equal hash codes.

##### 下面给出编写一个完美的 equals 方法的建议：
1) 显式参数命名为 otherObject, 稍后需要将它转换成另一个叫做 other 的变量。 

2) 检测 this 与 otherObject 是否引用同一个对象:
    if (this = otherObject) return true;

    这条语句只是一个优化。 实际上， 这是一种经常采用的形式。 因为计算这个等式要比一个一个地比较类中的域所付出的代价小得多。

3) 检测 otherObject 是否为 null , 如果为 null , 返回 false。 这项检测是很必要的。 
   if (otherObject = null) return false;

4) 比较 this 与 otherObject 是否属于同一个类。如果 equals 的语义在每个子类中有所改变， 就使用 getClass 检测:
    if (getClass() != otherObject.getCIassO) return false;

    如果所有的子类都拥有统一的语义， 就使用 instanceof 检测: 
    if (!(otherObject instanceof ClassName)) return false;

5) 将 otherObject 强制转换为相应的类类型变量: 
   
   ClassName other = (ClassName) otherObject
  
   因为传入的参数是 Object 对象类型，因此这里必须要做强制类型转换。否则 otherObject 无法调用 Employee 类的方法。

6）现在开始对所有需要比较的域进行比较了。使用 == 比较基本类型域，使用equals比较对象域。如果所有的域都匹配，就返回true; 否则返回false。

  return field1 == other.field1 && Objects.equals(fie1d2, other.field2)

  如果在子类中重新定义equals, 就要在其中包含调用super.equals(other).

##### 参考案例：

```java

public class Employee {
    private  int id;
    private String name;

    public Employee(int id, String name) {
        this. id = id;
        this.name = name;
    }

    public boolean equals(Object otherObject) {
        if (this == otherObject) return true;
        if (otherObject == null) return false;
        if (getClass() != otherObject.getClass()) return false;

        Employee other = (Employee) otherObject;

        return id == other.id;
    }
}
```

##### 关于equals方法的补充说明
有些作者认为不应该利用 getClass 检测， 因为这样不符合置换原则。

有一个应用 AbstractSet 类的 equals 方法的典型例子， 它将检测两个集合是否有相同的元素。
AbstractSet 类有两个具体子类: TreeSet 和 HashSet, 它们分别使用不同的算法实现查找集合元素的操作。 
无论集合采用何种方式实现， 都需要拥有对任意两个集合进行比较的功能。

然而， 集合是相当特殊的一个例子， 应该将 AbstractSetequals 声明为 final , 这是因为没有任何一个子类需要重定义集合是否相等的语义。
事实上， 这个方法并没有被声明为 final, 这样做，可以让子类选择更加有效的算法对集合进行是否相等的检测。


下面可以从两个截然不同的情况看一下这个问题: 
- 如果子类能够拥有自己的相等概念， 则对称性需求将强制采用 getClass 进行检测
- 如果由超类决定相等的概念， 那么就可以使用 instanceof 进行检测， 这样可以在不同子类的对象之间进行相等的比较。

在雇员和经理的例子中（Manager 继承自 Employee，详见 Java核心技术卷1 5.2.3）， 只要对应的域相等， 就认为两个对象相等。 

如果两个 Manager 对象所对应的姓名、 薪水和雇佣日期均相等， 而奖金不相等， 就认为它们是不相同的， 因此， 可以使用 getClass 检测。

但是，假设使用雇员的 ID 作为相等的检测标准， 并且这个相等的概念适用于所有的子类， 就可以使用 instanceof 进行检测， 并应该将 Employee.equals 声明为 final。

*注：本文中的例子为了更好地帮助理解，做了一些简化，假定用ID作为相等的检测标准，且不适用于子类，因此还是用 getClass 做检测。*

##### 常见错误
下面代码有什么问题呢？

```java
public class Employee {
    public boolean equals(Employee other) {
        return other != null
                && getClass() == other.getClass()
                && Objects.equals(name, other.name)
                && salary == other.salary
                && Objects.equals(hireDay, other.hireDay);
    }
}
```

这个方法声明的显式参数类型是 Employee。 
**其结果并没有覆盖 Object 类的 equals 方 法， 而是定义了一个完全无关的方法。**

为了避免发生类型错误， 可以使用 @Override 对覆盖超类的方法进行标记: 

```java
@Override public boolean equals(Object other)
```

如果出现了错误， 并且正在定义一个新方法， 编译器就会给出错误报告。 例如， 假设将下面的声明添加到 Employee 类中:

@Override public boolean equals(Employee other)

就会看到一个错误报告， 这是因为这个方法并没有覆盖超类 Object 中的任何方法。