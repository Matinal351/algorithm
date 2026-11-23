# Java 包装类与泛型基础：看懂 MaxPQ 的类型声明

[返回学习目录](README.md) · [上一篇：类与面向对象](Java类与面向对象入门.md) · [下一篇：异常处理与文件 IO](Java异常处理与文件IO.md)

本文整理包装类、泛型、Comparable 和泛型约束。先弄清 Integer 是什么，再逐层理解下面的声明：

~~~java
public class MaxPQ<Key extends Comparable<Key>>
~~~

各节的短代码是独立片段，同名类不要直接拼接。文末提供可单独运行的泛型盒子示例。

## 1. Integer 是什么

int 是基本类型；Integer 是与 int 对应的包装类，可以用对象的形式表示整数。

~~~java
int a = 10;
Integer b = 10;
~~~

两者都能参与常见整数运算，但类型不同：int 是基本类型，Integer 是引用类型。

| 基本类型 | 包装类 |
|---|---|
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| boolean | Boolean |
| char | Character |

有些 Java 功能要求使用引用类型，泛型类型实参就是其中之一：

~~~java
// Box<int> box; // 不允许
Box<Integer> box; // 允许，前提是已经定义 Box<T>
~~~

## 2. 自动装箱与拆箱

Java 可以在基本类型与包装类之间自动转换：

~~~java
int a = 10;
Integer b = a; // 自动装箱
int c = b;     // 自动拆箱
~~~

装箱可以理解为把整数交给对应的包装类表示；拆箱则是取出其中的基本类型值。不需要使用已经不推荐的 `new Integer(10)` 写法。

Integer 可以是 null，但 int 不可以：

~~~java
Integer number = null;
// int value = number; // 运行时抛出 NullPointerException
// int value = null;   // 编译错误
~~~

number 没有引用整数对象，因而无法拆箱。

另外，不要用两个 Integer 引用之间的 == 判断一般意义上的数值相等：它比较的是引用，某些值还可能复用缓存对象。确认非 null 后可以用 equals 比较内容；这一话题会在对象比较中进一步展开。

## 3. 泛型：先给类型留一个位置

假设盒子只保存整数：

~~~java
class Box {
    private Integer value;

    public void set(Integer value) {
        this.value = value;
    }

    public Integer get() {
        return value;
    }
}
~~~

如果还要保存字符串，不想为每种类型重写一个盒子，就可以引入类型参数：

~~~java
class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
~~~

T 是类型参数，相当于“使用时再指定的某种类型”，不是固定关键字。也可以命名为 Value、E 或其他合法名称。

~~~java
Box<Integer> numberBox = new Box<Integer>();
numberBox.set(123);
Integer number = numberBox.get();

Box<String> textBox = new Box<String>();
textBox.set("你好");
String text = textBox.get();
~~~

对 numberBox 而言，T 对应 Integer；对 textBox 而言，T 对应 String。

编译器会检查不匹配的使用：

~~~java
// numberBox.set("你好"); // 编译错误：要求 Integer
~~~

泛型使同一份代码能够处理不同类型，同时保留编译时的类型检查。它不意味着 Java 为每个类型都生成一份独立的类定义。

## 4. 右边的 <> 是什么

下面两种写法都可以：

~~~java
Box<String> box = new Box<String>();
Box<String> another = new Box<>();
~~~

右边的空尖括号叫菱形语法，编译器根据上下文推断类型。它不是“不限制类型”，another 仍然只能按 Box<String> 的规则使用。

## 5. Comparable：约定比较大小的接口

一个类型实现 Comparable<T>，就提供与 T 类型对象比较的方法：

~~~java
int compareTo(T other);
~~~

返回值的含义是：

| 返回值 | 当前对象相对 other |
|---|---|
| 小于 0 | 更小 |
| 等于 0 | 比较结果相等 |
| 大于 0 | 更大 |

不要假设它只返回 -1、0、1。

Integer 已经实现 Comparable<Integer>：

~~~java
Integer a = 3;
System.out.println(a.compareTo(5)); // 负数
System.out.println(a.compareTo(3)); // 0
System.out.println(a.compareTo(1)); // 正数
~~~

compareTo 返回 0 表示按这套顺序比较相等，不是在判断两个引用是否指向同一对象。

## 6. 泛型约束

先看没有约束的声明：

~~~java
class MaxPQ<Key> {
    // Key 暂时没有比较能力方面的限制
}
~~~

如果内部要比较两个 Key：

~~~java
// a.compareTo(b);
~~~

编译器不能保证任意类型都具备 compareTo。

所以增加限制：

~~~java
public class MaxPQ<Key extends Comparable<Key>> {
    // 此处 Key 满足 Comparable<Key> 的要求
}
~~~

逐层理解：

- public class MaxPQ：定义一个公开类。
- Key：元素类型的占位名称。
- Comparable<Key>：能够与 Key 类型元素比较的接口。
- Key extends Comparable<Key>：Key 必须满足这一接口约束。

在泛型类型参数的上界中统一使用 extends，即使右边是接口；普通类实现接口时仍使用 implements：

~~~java
class Student implements Comparable<Student> {
    private int age;

    Student(int age) {
        this.age = age;
    }

    @Override
    public int compareTo(Student other) {
        return Integer.compare(this.age, other.age);
    }
}
~~~

这里按年龄比较。Integer.compare 避免直接用整数相减时可能发生的溢出。

于是 Student 满足 MaxPQ<Student> 的类型要求。Integer 也满足 MaxPQ<Integer> 的要求；没有实现相应比较接口的普通类则不满足。

声明末尾的两个 >，分别结束 Comparable<Key> 和整个类型参数列表。

这个约束只是说明类型要求，不会自动实现最大优先队列的入队、出队等逻辑。

## 7. 完整示例：泛型盒子

保存为 Main.java：

~~~java
class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<Integer> numbers = new Box<>();
        numbers.set(123);

        Box<String> texts = new Box<>();
        texts.set("你好");

        int number = numbers.get(); // 自动拆箱
        String text = texts.get();

        System.out.println(number);
        System.out.println(text);
    }
}
~~~

预期输出：

~~~text
123
你好
~~~

编译运行：

~~~text
javac -encoding UTF-8 Main.java
java Main
~~~

## 8. 容易混淆的地方

| 概念 | 含义 |
|---|---|
| Integer | int 对应的包装类 |
| Box<T> 中的 T | 定义时的类型参数 |
| Box<String> 中的 String | 使用时指定的类型实参 |
| new Box<>() | 利用上下文推断类型 |
| Comparable<T> | 比较能力的接口 |
| T extends Comparable<T> | 对类型参数的上界约束 |
