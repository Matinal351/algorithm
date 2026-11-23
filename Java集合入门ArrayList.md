# Java 集合入门：ArrayList 的增删改查与对象引用

[返回学习目录](README.md) · [上一篇：异常处理与文件 IO](Java异常处理与文件IO.md) · [下一篇：HashSet 与 HashMap](Java集合进阶HashSet与HashMap.md)

本文整理 ArrayList 的创建、增删改查、遍历、遍历中删除，以及保存学生对象时的引用行为。理解泛型可先读 [包装类与泛型基础](Java包装类与泛型基础.md)。

短代码默认位于方法中，并已导入 `java.util.ArrayList`。文末给出独立完整程序。

## 目录

1. [为什么需要 ArrayList](#1-为什么需要-arraylist)
2. [增删改查](#2-增删改查)
3. [遍历](#3-遍历)
4. [遍历时删除元素](#4-遍历时删除元素)
5. [保存学生对象](#5-保存学生对象)
6. [修改对象与替换元素](#6-修改对象与替换元素)
7. [完整示例](#7-完整示例)
8. [接下来学什么](#8-接下来学什么)

## 1. 为什么需要 ArrayList

数组创建后长度固定：

~~~java
String[] names = new String[3];
~~~

如果人数不断增加，固定长度不够方便。ArrayList 是 Java 提供的列表类，能够随着添加元素扩展容量：

~~~java
ArrayList<String> names = new ArrayList<>();
names.add("小明");
names.add("小红");
names.add("小华");

System.out.println(names.get(0)); // 小明
System.out.println(names.size()); // 3
~~~

`<String>` 规定保存字符串，右边 <> 让编译器根据上下文推断类型。

ArrayList 保持元素的位置顺序，允许重复，也允许 null。它不是会根据姓名自动排序的结构。

## 2. 增删改查

| 操作 | 方法 |
|---|---|
| 添加到末尾 | add(元素) |
| 获取指定位置 | get(下标) |
| 替换指定位置 | set(下标, 新元素) |
| 删除指定位置 | remove(下标) |
| 删除第一个匹配元素 | remove(元素) |
| 当前元素数量 | size() |

对于 get、set 和按下标删除的 remove，有效下标是 0 到 size() - 1。空列表没有第 0 个元素。

~~~java
ArrayList<String> names = new ArrayList<>();
names.add("小明");
names.add("小红");
names.add("小华");

names.set(1, "小丽");
System.out.println(names); // [小明, 小丽, 小华]

names.remove(0);
System.out.println(names); // [小丽, 小华]
System.out.println(names.get(0)); // 小丽
~~~

set 是替换，不改变元素数量。删除后，后面的元素会向前移动，下标随之变化。

按内容删除时，只删除第一个匹配项：

~~~java
ArrayList<String> names = new ArrayList<>();
names.add("小明");
names.add("小红");
names.add("小明");

names.remove("小明");
System.out.println(names); // [小红, 小明]
~~~

没有匹配元素时，这种按内容删除不会修改列表，并返回 false；删除成功返回 true。按内容匹配涉及 equals，字符串按内容比较。

### 整数列表的 remove 要注意

因为 remove 有按下标和按对象两种形式，整数列表容易混淆：

~~~java
ArrayList<Integer> numbers = new ArrayList<>();
numbers.add(10);
numbers.add(20);
numbers.add(30);

numbers.remove(1);                  // 删除下标 1，移除 20
numbers.remove(Integer.valueOf(10)); // 删除数值 10
System.out.println(numbers);        // [30]
~~~

Integer.valueOf(10) 提供一个表示 10 的 Integer，明确选择按元素删除。它与 remove(10) 不同，后者表示删除下标 10，可能越界。

## 3. 遍历

### 普通 for

~~~java
for (int i = 0; i < names.size(); i++) {
    String name = names.get(i);
    System.out.println(name);
}
~~~

条件是 i < size()，不是 <=，因为最后一个元素下标是 size() - 1。

需要下标或替换指定位置时，这种写法方便：

~~~java
for (int i = 0; i < names.size(); i++) {
    names.set(i, "未知");
}
~~~

### 增强 for

只需要依次使用每个元素时：

~~~java
for (String name : names) {
    System.out.println(name);
}
~~~

可以读作“从 names 依次取出字符串，交给本轮变量 name”。

变量名不固定，但给这个局部变量重新赋值，不会替换列表元素：

~~~java
for (String name : names) {
    name = "未知";
}
~~~

这段代码不改变 names 中保存的字符串引用。替换列表元素需要 set。

## 4. 遍历时删除元素

假设要删除所有“小明”：

~~~java
ArrayList<String> names = new ArrayList<>();
names.add("小明");
names.add("小明");
names.add("小红");
~~~

下面从前往后的写法会漏删：

~~~java
for (int i = 0; i < names.size(); i++) {
    if ("小明".equals(names.get(i))) {
        names.remove(i);
    }
}
~~~

删除下标 0 后，第二个小明移动到下标 0，但 i 随后变成 1，跳过了它。

字符串内容比较用 equals。这里把固定的非 null 字符串放在前面，即使列表元素为 null，也不会因为调用方为 null 而出错。

一种容易理解的方式是从后往前删除：

~~~java
for (int i = names.size() - 1; i >= 0; i--) {
    if ("小明".equals(names.get(i))) {
        names.remove(i);
    }
}
System.out.println(names); // [小红]
~~~

删除 i 时，未处理的 0 到 i - 1 位置不受影响，因此不会跳过待检查元素。

不要在增强 for 中直接用列表的 add 或 remove 改变长度：

~~~java
for (String name : names) {
    if ("小明".equals(name)) {
        names.remove(name); // 错误用法，可能抛出 ConcurrentModificationException
    }
}
~~~

增强 for 内部依赖迭代器，直接对列表做结构性修改会干扰遍历。即使某些情况下未报错，也不应该依赖这种行为。这里先掌握倒序删除，迭代器删除、removeIf 可以以后再学。

倒序删除避免漏删，但 ArrayList 每次删除仍可能移动元素；大量删除时最坏总成本可能达到 O(n²)。

## 5. 保存学生对象

泛型元素类型也可以是自己定义的类：

~~~java
class Student {
    private String name;
    private int age;

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
~~~

为突出集合用法，这个小例子省略了年龄合法性检查，实际需要时可沿用上一篇面向对象文章中的校验方式。

~~~java
ArrayList<Student> students = new ArrayList<>();
students.add(new Student("小明", 18));
students.add(new Student("小红", 19));

Student first = students.get(0);
System.out.println(first.getName()); // 小明
~~~

遍历：

~~~java
for (Student student : students) {
    System.out.println(
        student.getName() + "：" + student.getAge() + "岁"
    );
}
~~~

ArrayList 保存的是对象引用，不会因为 add 或 get 自动复制学生对象。

## 6. 修改对象与替换元素

~~~java
Student s = students.get(0);
s.setAge(20);

System.out.println(students.get(0).getAge()); // 20
~~~

s 与列表第一个位置引用同一个学生，因此通过 s 修改年龄，之后从列表也能读到变化。

增强 for 也可以修改对象状态：

~~~java
for (Student student : students) {
    student.setAge(student.getAge() + 1);
}
~~~

这不是增删列表元素，不会改变列表长度。

但重新给局部变量赋值是另一回事：

~~~java
for (Student student : students) {
    student = new Student("小华", 18);
}
~~~

这只是让本轮 student 引用新对象，没有替换列表中的引用。

要替换第一个学生：

~~~java
students.set(0, new Student("小华", 18));
~~~

| 操作 | 结果 |
|---|---|
| student.setAge(20) | 修改引用对象的状态 |
| student = new Student(...) | 修改局部变量所引用的对象 |
| students.set(0, new Student(...)) | 替换列表第 0 个位置的引用 |

同一个对象也可以被添加多次。这些位置仍然引用同一对象，不会变成独立副本。

## 7. 完整示例

保存为 Main.java：

~~~java
import java.util.ArrayList;

class Student {
    private String name;
    private int age;

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return name + "：" + age + "岁";
    }
}

public class Main {
    public static void main(String[] args) {
        ArrayList<Student> students = new ArrayList<>();
        students.add(new Student("小明", 18));
        students.add(new Student("小红", 19));
        students.add(new Student("小华", 17));

        Student first = students.get(0);
        first.setAge(20);

        // 删除未满 18 岁的学生
        for (int i = students.size() - 1; i >= 0; i--) {
            if (students.get(i).getAge() < 18) {
                students.remove(i);
            }
        }

        for (Student student : students) {
            System.out.println(student);
        }

        System.out.println("人数：" + students.size());
    }
}
~~~

预期输出：

~~~text
小明：20岁
小红：19岁
人数：2
~~~

编译运行：

~~~text
javac -encoding UTF-8 Main.java
java Main
~~~

## 8. 接下来学什么

集合部分已经补成以下续篇，建议依次阅读：

1. [HashSet 与 HashMap](Java集合进阶HashSet与HashMap.md)：去重、按键查找、计数、插入顺序与有序映射。
2. [对象相等与集合排序](Java对象相等与集合排序.md)：equals/hashCode、Comparable、Comparator、Lambda 和多条件排序。
3. [迭代器与队列](Java迭代器与队列.md)：遍历删除、Queue、Deque、PriorityQueue、复制与集合选型。

之后还可以单独学习 Stream、泛型通配符和多线程等专题。
