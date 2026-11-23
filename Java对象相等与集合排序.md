# Java 对象相等与集合排序：equals、hashCode 和 Comparator

[返回目录](README.md) · [上一篇：Set 与 Map](Java集合进阶HashSet与HashMap.md) · [下一篇：迭代器与队列](Java迭代器与队列.md)

集合需要回答两个不同的问题：“两个元素是否相等？”以及“哪个元素排在前面？”本篇分别讲清楚。短代码需要导入相应的 java.util 类型，完整程序放在文末。

## 目录

- [1. == 与 equals 不是一回事](#1--与-equals-不是一回事)
- [2. 自己定义“相同学生”](#2-自己定义相同学生)
- [3. 为什么必须同时考虑 hashCode](#3-为什么必须同时考虑-hashcode)
- [4. 排序与“相等”是两个问题](#4-排序与相等是两个问题)
- [5. Comparable 与 Comparator](#5-comparable-与-comparator)
- [6. 看懂排序里的 Lambda](#6-看懂排序里的-lambda)
- [7. 多条件排序与方法引用](#7-多条件排序与方法引用)
- [8. TreeSet 的重要区别](#8-treeset-的重要区别)
- [9. 完整示例：先去重，再排序](#9-完整示例先去重再排序)

## 1. == 与 equals 不是一回事

对基本类型，== 比较对应的值；对引用类型，== 判断是否引用同一个对象。

~~~java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
~~~

a 和 b 是不同字符串对象，但文字内容相同。String 重写了 equals，因此可以按内容比较。

Object 默认的 equals 采用对象身份判断。如果自定义 Student 没有重写 equals，两个分别 new 出来的学生，即使字段值相同，也不会自动判为相等。

调用 a.equals(b) 前，a 不能为 null。需要兼容 null 时可用 Objects.equals(a, b)。

## 2. 自己定义“相同学生”

假设学号相同就视为同一个学生，可以写：

~~~java
final class Student {
    private final int id;
    private final String name;

    Student(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof Student)) {
            return false;
        }
        Student other = (Student) obj;
        return this.id == other.id;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
~~~

equals 的参数必须是 Object，才能重写 Object.equals。若写成 equals(Student other)，那只是重载，集合调用时可能不按预期工作。

示例把 Student 声明为 final，避免继续继承导致相等规则更加复杂。真实项目还需要根据业务决定哪些字段构成对象身份。

## 3. 为什么必须同时考虑 hashCode

HashSet 和 HashMap 使用散列结构。可以粗略理解为：先根据 hashCode 缩小查找范围，再用 equals 区分候选对象。

必须满足：

- equals 相等的对象，hashCode 必须相同。
- hashCode 相同的对象，未必 equals 相等，可能发生冲突。
- 在参与相等判断的数据没有改变时，相等判断与哈希结果要保持一致。

只重写 equals 却保留不匹配的 hashCode，会破坏散列集合所依赖的契约。契约详见 [Object 官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)。

例如：

~~~java
Set<Student> students = new HashSet<>();
students.add(new Student(1001, "小明"));
students.add(new Student(1001, "另一个姓名"));

System.out.println(students.size()); // 1
~~~

我们定义只看学号，因此这两个对象相等。HashMap 使用自定义对象作键，也需要遵守同样规则；相等键再次 put 会替换值。

### 不要修改作为身份的字段

如果对象放入 HashSet 或作为 HashMap 的键后，修改了参与 equals/hashCode 的字段，集合可能无法按新值正确查找或删除它。

例子把 id 设为 final，避免这种情况。修改不参与相等与哈希的其他状态，不一定影响查找，但仍应注意业务含义。

## 4. 排序与“相等”是两个问题

对整数列表排序：

~~~java
List<Integer> numbers = new ArrayList<>();
numbers.add(3);
numbers.add(1);
numbers.add(2);

Collections.sort(numbers);
System.out.println(numbers); // [1, 2, 3]

numbers.sort(Comparator.reverseOrder());
System.out.println(numbers); // [3, 2, 1]
~~~

Collections 是工具类，Collection 是接口。排序会改变这个列表的元素排列；不会自动去重。

对可修改的 List，sort 是常见排序方法；并非所有 List 都支持修改。

## 5. Comparable 与 Comparator

| 接口 | 方法 | 用途 |
|---|---|---|
| Comparable<T> | 当前对象.compareTo(other) | 类型自己提供自然顺序 |
| Comparator<T> | compare(a, b) | 外部提供一套比较规则 |

返回负数表示前者应排在后者之前，0 表示按该规则比较相等，正数表示前者应排在后者之后。

之前的 MaxPQ<Key extends Comparable<Key>> 要求元素类型具备自然比较能力。使用 Comparator 则可以在不修改元素类的情况下指定不同排序规则。

假设另一个用于本节的 Student 类有 getAge、getId 方法，比较器可以写成：

~~~java
class AgeComparator implements Comparator<Student> {
    @Override
    public int compare(Student a, Student b) {
        return Integer.compare(a.getAge(), b.getAge());
    }
}
~~~

使用：

~~~java
students.sort(new AgeComparator());
~~~

Integer.compare 避免 a.getAge() - b.getAge() 在整数极端值下溢出。比较器还要满足一致性与传递性，不能遇到相等时随意返回正数。

## 6. 看懂排序里的 Lambda

同一比较规则可以简写：

~~~java
Comparator<Student> byAge =
    (a, b) -> Integer.compare(a.getAge(), b.getAge());

students.sort(byAge);
~~~

这是 Lambda 表达式，在此用于提供 Comparator.compare 的实现：

- (a, b)：接收两个学生对象。
- ->：连接参数与执行内容。
- Integer.compare(...)：计算并返回比较结果。

这里不是定义一个学生对象，而是在提供“怎样比较两个学生”的行为。

也可以直接传给 sort：

~~~java
students.sort((a, b) ->
    Integer.compare(a.getAge(), b.getAge())
);
~~~

类型的自然顺序不必覆盖所有需求：今天可以按年龄，明天可以按学号。

## 7. 多条件排序与方法引用

按年龄升序，年龄相同再按学号升序：

~~~java
students.sort(
    Comparator.comparingInt(Student::getAge)
              .thenComparingInt(Student::getId)
);
~~~

Student::getAge 是方法引用。这里可以理解为：给一个 Student，调用它的 getAge 方法，取出排序用的整数。

comparingInt 创建按整数属性升序比较的规则；thenComparingInt 添加相同时的第二规则。

降序：

~~~java
Comparator<Student> byAge =
    Comparator.comparingInt(Student::getAge);

students.sort(byAge.reversed());
~~~

reversed 反转它前面那个比较器的整体顺序。如果对多条件比较器最后调用 reversed，所有比较层次都会被反转，不只是第一项。

字符串排序默认按字符的自然顺序，不是自动按中文拼音。涉及 null 元素或 null 属性时，也需要另行规定规则。

## 8. TreeSet 的重要区别

~~~java
Set<Student> students =
    new TreeSet<>(Comparator.comparingInt(Student::getAge));
~~~

如果两个不同学生年龄相同，比较结果为 0，这个 TreeSet 会把它们视为同一个元素，只保留一个。

如果只是希望所有学生按年龄排列，应使用 List 排序；如果确实需要有序去重，则要设计与 equals 一致的比较规则。

TreeSet/TreeMap 根据比较结果识别相同元素或相同键。要遵守 Set/Map 的一般相等契约，排序规则应与 equals 一致。参见 [TreeSet 官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TreeSet.html)。

“按年龄相同”与“按学号相同”是两套不同的关系，不能无意混用。下一节的完整示例使用 HashSet 按学号去重，再转换为 List 排序。

## 9. 完整示例：先去重，再排序

保存为 Main.java：

~~~java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

final class Student {
    private final int id;
    private final String name;
    private final int age;

    Student(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public int getAge() {
        return age;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof Student)) {
            return false;
        }
        Student other = (Student) obj;
        return id == other.id;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }

    @Override
    public String toString() {
        return id + " " + name + " " + age;
    }
}

public class Main {
    public static void main(String[] args) {
        Set<Student> unique = new HashSet<>();
        unique.add(new Student(1002, "小红", 19));
        unique.add(new Student(1001, "小明", 18));
        unique.add(new Student(1003, "小华", 18));
        unique.add(new Student(1001, "重复记录", 20));

        List<Student> students = new ArrayList<>(unique);
        students.sort(
            Comparator.comparingInt(Student::getAge)
                      .thenComparingInt(Student::getId)
        );

        for (Student student : students) {
            System.out.println(student);
        }
    }
}
~~~

预期输出：

~~~text
1001 小明 18
1003 小华 18
1002 小红 19
~~~

HashSet 不保证遍历顺序，但转换后显式按年龄和学号排序，因此这里的最终输出确定。相同学号的第二次 add 不会替换第一次已经存入的学生对象。
