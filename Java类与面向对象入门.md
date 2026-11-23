# Java 类与面向对象入门：从创建对象到理解接口

[返回学习目录](README.md) · [下一篇：包装类与泛型基础](Java包装类与泛型基础.md)

已经会写变量、循环和普通方法之后，学习 Java 的下一步，是理解如何用类组织数据和行为。

本文沿着“学生”这个例子逐步介绍。各节的短代码用于单独说明知识点，不要把所有同名类直接拼在一起。文末提供一份可以保存为 Main.java 运行的完整示例。

## 目录

1. [类、对象与引用](#1-类对象与引用)
2. [属性、方法、参数和返回值](#2-属性方法参数和返回值)
3. [默认值](#3-默认值)
4. [构造方法与 this](#4-构造方法与-this)
5. [封装与访问权限](#5-封装与访问权限)
6. [static：属于类的成员](#6-static属于类的成员)
7. [继承与方法重写](#7-继承与方法重写)
8. [多态与类型转换](#8-多态与类型转换)
9. [super：调用父类的实现与构造方法](#9-super调用父类的实现与构造方法)
10. [抽象类](#10-抽象类)
11. [接口](#11-接口)
12. [final](#12-final)
13. [Object 与 toString](#13-object-与-tostring)
14. [完整示例](#14-完整示例)
15. [自测](#15-自测)

## 1. 类、对象与引用

如果分别用变量保存每位学生的姓名和年龄，人数一多就很难管理。类可以把属于同一个学生的数据放在一起：

~~~java
class Student {
    String name;
    int age;
}
~~~

`Student` 是类，规定学生有哪些信息；`name` 和 `age` 是成员变量，也叫字段，初学时常称为属性。

类定义好后，通过 `new` 创建具体的对象：

~~~java
Student s1 = new Student();
s1.name = "小明";
s1.age = 18;

Student s2 = new Student();
s2.name = "小红";
s2.age = 19;
~~~

两个对象分别保存各自的数据。`s1.name` 可以读作“s1 所引用学生的姓名”。

`Student s1 = new Student();` 中，左边声明一个 Student 类型的变量，右边创建对象，再把对象的引用赋给变量。严格来说，变量 s1 保存的是引用，不是把整个对象装进变量中。

因此，下面的赋值不会复制对象：

~~~java
Student other = s1;
other.age = 20;

System.out.println(s1.age); // 20
~~~

s1 和 other 引用同一个对象，通过其中一个变量修改属性，另一个变量也能读到变化。

## 2. 属性、方法、参数和返回值

属性保存信息，实例方法定义对象可以执行的操作：

~~~java
class Student {
    String name;
    int age;

    void introduce() {
        System.out.println("我叫" + name + "，今年" + age + "岁");
    }

    void haveBirthday() {
        age++;
    }

    void grow(int years) {
        age += years;
    }

    int getAgeNextYear() {
        return age + 1;
    }
}
~~~

调用方法时使用点号：

~~~java
Student s = new Student();
s.name = "小明";
s.age = 18;

s.haveBirthday();
s.grow(2);
s.introduce(); // 我叫小明，今年21岁

int result = s.getAgeNextYear();
System.out.println(result); // 22
System.out.println(s.age);  // 21
~~~

这里有三个区别：

- `void` 表示不返回结果值，但方法仍然可以修改属性。
- `int years` 是参数，每次调用时接收传入的整数。
- `return age + 1;` 计算并返回一个结果，没有给 age 重新赋值，所以不会修改年龄。

通过哪个对象调用实例方法，方法中的实例属性就属于哪个对象。

## 3. 默认值

没有显式赋值时，成员变量有默认值：

| 类型 | 默认值 |
|---|---|
| byte、short、int、long | 0 |
| float、double | 0.0 |
| boolean | false |
| char | '\u0000'，不是空格 |
| 引用类型，如 String、数组、自定义类 | null |

`null` 表示没有引用任何对象，与空字符串 `""` 不同。

默认值并不是无参数构造方法特有的。即使使用有参数构造方法，没有被另行赋值的成员变量也会保留默认值。

方法里的局部变量则不同：

~~~java
void test() {
    int number;
    // System.out.println(number); // 编译错误：使用前没有赋值
}
~~~

数组元素也有相应类型的默认值，但数组变量本身仍然要先被赋值，才能使用。

## 4. 构造方法与 this

构造方法负责在创建对象时进行初始化：

~~~java
class Student {
    String name;
    int age;

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
~~~

现在可以直接写：

~~~java
Student s = new Student("小明", 18);
~~~

构造方法的名称必须与类名相同，不写返回类型，连 void 也不写。

`this` 表示当前对象。因此：

~~~java
this.name = name;
~~~

左边是对象的属性，右边是参数。参数与属性同名时，直接写 name 会优先使用参数。

下面是常见错误：

~~~java
Student(String name, int age) {
    name = name;
    age = age;
}
~~~

这只是把参数赋值给自己，对象属性没有被修改。

### 默认构造方法

只有当类没有声明任何构造方法时，Java 才会提供一个无参数的默认构造方法。自己声明了有参数版本后，想使用 `new Student()`，就需要另外定义无参数版本。

### 构造方法重载

一个类可以提供参数不同的构造方法：

~~~java
Student(String name, int age) {
    this.name = name;
    this.age = age;
}

Student(String name) {
    this(name, 0);
}

Student() {
    this("未命名", 0);
}
~~~

`this(name, 0)` 调用本类的另一个构造方法，复用初始化代码，整个过程仍然只创建一个对象。

本文统一采用把 this(...) 或 super(...) 放在构造方法第一条语句的常见兼容写法；它们不能形成循环调用。

重载靠参数数量、类型或类型顺序区分。仅修改参数名，或者仅修改返回类型，不能构成重载。普通方法也可以重载。

## 5. 封装与访问权限

如果外部可以直接写 `s.age = -10;`，就可能破坏年龄的规则。

可以把属性设为 private，通过方法检查：

~~~java
class Student {
    private int age;

    Student(int age) {
        this.age = checkedAge(age);
    }

    private static int checkedAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("年龄不能小于 0");
        }
        return age;
    }

    public void setAge(int age) {
        this.age = checkedAge(age);
    }

    public int getAge() {
        return age;
    }
}
~~~

`private` 限制外部直接访问属性，`public` 方法向外提供受控的操作。构造时也要检查，不能只检查 setter。

这里的 throw 表示报告非法参数，异常处理可以后续单独学习。检查逻辑使用私有静态方法，避免构造时调用可能被子类重写的实例方法。

封装的重点是维护规则，不是机械地给所有字段配 getter 和 setter。只读信息可能只需要 getter。

常见访问权限如下：

| 修饰符 | 基本含义 |
|---|---|
| public | 允许其他代码访问，仍受所属类型等访问规则限制 |
| protected | 同包可以访问；跨包的子类可以按继承访问规则使用 |
| 不写修饰符 | 包访问权限，只能在同一个包中访问 |
| private | 入门时理解为只在声明它的类内部直接访问 |

跨包使用 protected 还有引用类型等限制，不应简单理解为“任何地方的子类都能随意访问任意父类对象”。

## 6. static：属于类的成员

普通实例属性每个对象一份；static 属性由同一个类的对象共享：

~~~java
class Student {
    String name;
    private static int count = 0;

    Student(String name) {
        this.name = name;
        Student.count++;
    }

    public static int getCount() {
        return Student.count;
    }
}
~~~

~~~java
System.out.println(Student.getCount()); // 0
Student s1 = new Student("小明");
Student s2 = new Student("小红");
System.out.println(Student.getCount()); // 2
~~~

推荐通过类名访问静态成员。count 记录的是累计创建次数，不是当前存活对象数。

如果构造方法中声明同名局部变量：

~~~java
int count = 10;
count++;         // 局部变量
Student.count++; // 静态属性
~~~

直接写 count 使用局部变量；用类名可以明确访问静态属性。同名实例属性则用 this.count 区分。

静态方法不依附于某一个当前对象，因此不能使用 this，也不能直接访问实例字段。但可以通过明确提供的对象访问实例成员：

~~~java
static void printName(Student student) {
    System.out.println(student.name);
}
~~~

## 7. 继承与方法重写

学生是一种人，所以可以写：

~~~java
class Person {
    String name;

    public void introduce() {
        System.out.println("我叫" + name);
    }
}

class Student extends Person {
    public void study() {
        System.out.println("正在学习");
    }

    @Override
    public void introduce() {
        System.out.println("我叫" + name + "，我是一名学生");
    }
}
~~~

extends 表示继承。Person 是父类，Student 是子类。学生可以使用继承来的适当成员，也可以增加新方法或重写已有方法。

`@Override` 让编译器检查这个方法是否真的构成重写。

重写要求方法签名匹配；返回类型必须符合重写规则，引用返回类型允许协变。初学时先保持父子类返回类型一致即可。访问权限不能缩小。

为什么不能把父类的 public 方法重写成 private？因为父类承诺这个功能可以被外部调用，子类作为父类的一种，也要遵守这个承诺。

继承还需要注意：

- 一个类只能直接继承一个类。
- 构造方法不会被子类继承。
- 父类的 private 字段不能被子类直接访问，应通过父类提供的适当方法使用。
- “学生有一本书”适合让学生保存一个 Book 引用，称为组合；它不是“学生是一种书”。

## 8. 多态与类型转换

看这行代码：

~~~java
Person p = new Student();
~~~

变量声明类型是 Person，实际对象类型是 Student。

对于被重写的实例方法，记住：

> 编译时能通过变量调用哪些方法，看声明类型；运行时执行哪个重写版本，看实际对象类型。

~~~java
p.introduce(); // 执行 Student 重写的版本
// p.study(); // 编译错误：Person 中没有 study()
~~~

这条规则主要讨论实例方法重写，不要套用到字段和静态方法上。

多态可以让同一方法处理不同子类：

~~~java
static void welcome(Person person) {
    person.introduce();
}
~~~

只要 Teacher 也继承 Person，就能传入学生或老师，由各自对象决定如何介绍自己。

### 向下转型

如果需要调用学生特有的方法，可以先检查再转换：

~~~java
if (p instanceof Student) {
    Student s = (Student) p;
    s.study();
}
~~~

转换没有创建对象，也没有把普通人变成学生，只是用子类类型使用原有引用。

~~~java
Person p = new Person();
Student s = (Student) p; // 运行时抛出 ClassCastException
~~~

另外，`null instanceof Student` 的结果是 false。

## 9. super：调用父类的实现与构造方法

### super.方法名(...)

子类重写后，父类实现不会自动执行。需要时可以明确调用：

~~~java
@Override
public void introduce() {
    super.introduce();
    System.out.println("我是一名学生");
}
~~~

关键问题是：调用 super 后，操作的是谁？

**仍然是同一个学生对象。super 选择父类的实现，不会切换到另一个对象。父类方法中的 this 也仍然是当前学生对象。**

如果在重写后的 introduce 中直接调用 introduce()，会再次调用自己，可能无限递归。super.introduce() 才是明确调用父类版本。

### super(...)

它用于调用父类构造方法：

~~~java
class Person {
    private String name;

    Person(String name) {
        this.name = name;
    }
}

class Student extends Person {
    private int studentId;

    Student(String name, int studentId) {
        super(name);
        this.studentId = studentId;
    }
}
~~~

父类负责姓名初始化，子类负责学号初始化，共同完成一个学生对象的构造。

如果既没有显式调用 super(...)，也没有通过 this(...) 委托给其他构造方法，通常会隐式调用 super()。因此父类没有可访问的无参数构造方法时，子类需要明确调用可用的父类构造方法。

## 10. 抽象类

如果父类只想规定“必须能自我介绍”，不提供具体实现，可以声明抽象方法：

~~~java
abstract class Person {
    public abstract void introduce();
}
~~~

抽象方法没有方法体，以分号结尾。包含抽象方法的类必须是抽象类。

~~~java
class Student extends Person {
    @Override
    public void introduce() {
        System.out.println("我是一名学生");
    }
}
~~~

具体子类必须实现继承下来的抽象方法；也可以继续声明为抽象类，把实现留给后续子类。

抽象类不能直接 new，但可以用作变量类型：

~~~java
Person p = new Student();
p.introduce();
~~~

抽象类可以有属性、构造方法、普通方法，也可以没有抽象方法。

## 11. 接口

接口用于约定能力。学生和机器人都可以唱歌，不必因为这项共同能力而继承同一个具体父类：

~~~java
interface Singable {
    void sing();
}

class Student implements Singable {
    @Override
    public void sing() {
        System.out.println("学生在唱歌");
    }
}

class Robot implements Singable {
    @Override
    public void sing() {
        System.out.println("机器人在唱歌");
    }
}
~~~

接口中的这种无方法体的普通实例方法，默认是 public abstract，所以实现时必须写 public。

~~~java
static void perform(Singable singer) {
    singer.sing();
}
~~~

现在学生和机器人都能参加：

~~~java
perform(new Student());
perform(new Robot());
~~~

参数写 Student，要求传入学生；参数写 Singable，要求对象所属类型实现这个接口。仅仅写一个同名 sing() 方法，没有实现接口，并不足以作为 Singable 传入。

### 一个类实现多个接口

~~~java
interface Swimmable {
    void swim();
}

class Student implements Singable, Swimmable {
    public void sing() {
        System.out.println("唱歌");
    }

    public void swim() {
        System.out.println("游泳");
    }
}
~~~

~~~java
Student s = new Student();
Singable singer = s;
Swimmable swimmer = s;

singer.sing();
// singer.swim(); // 编译错误：Singable 没有声明 swim()

swimmer.swim();
// swimmer.sing(); // 编译错误：Swimmable 没有声明 sing()
~~~

这些变量引用同一个对象。通过 singer 不能调用 swim，不代表学生失去游泳能力，而是这个变量的声明类型没有提供该操作。

### default 方法

接口也可以提供默认实现：

~~~java
interface Singable {
    void sing();

    default void thankAudience() {
        System.out.println("谢谢大家");
    }
}
~~~

实现类可以直接使用 thankAudience，也可以重写。默认方法不会自动在 sing 后执行，仍然需要明确调用。

### 与抽象类配合

~~~java
class Student extends Person implements Singable, Swimmable {
    // 实现所有需要的方法
}
~~~

抽象类适合相关类型共享状态和基础实现；接口适合约定不同类型共同具备的能力。两者可以配合，并不需要二选一。

## 12. final

| 位置 | 含义 |
|---|---|
| final 变量 | 赋值后不能重新赋值 |
| final 实例方法 | 不能被子类重写 |
| final 类 | 不能被继承 |

最容易混淆的是 final 引用：

~~~java
final Student s = new Student();
s.name = "小明";    // 若 name 可写，则允许修改对象内部状态
// s = new Student(); // 不允许让变量引用另一个对象
~~~

数组同理：

~~~java
final int[] numbers = {1, 2, 3};
numbers[0] = 100;        // 可以
// numbers = new int[5]; // 不可以
~~~

final 限制引用变量重新赋值，不自动让对象不可变。final 类也不意味着属性自动变为只读。

## 13. Object 与 toString

没有显式指定父类的普通类，默认继承 Object。Object 自己没有父类。

如果 Student 继承 Person，而 Person 默认继承 Object，那么 Student 也间接继承 Object。

Object 提供 toString()，用于返回对象的文字表示。默认结果类似 Student@2f92e0f4，后半部分是哈希码的十六进制表示，不应当作内存地址。

可以重写：

~~~java
@Override
public String toString() {
    return "姓名：" + name + "，年龄：" + age;
}
~~~

直接打印一个非 null 对象时，会使用它的 toString：

~~~java
System.out.println(s);
~~~

toString 返回字符串，打印工作由 println 完成。

## 14. 完整示例

下面把构造方法、封装、继承、抽象方法、接口、多态和静态计数放在同一个程序中。

保存为 `Main.java`。安装 JDK 后执行 `javac -encoding UTF-8 Main.java`，再执行 `java Main`。

~~~java
interface Singable {
    void sing();

    default void thankAudience() {
        System.out.println("谢谢大家");
    }
}

abstract class Person {
    private final String name;
    private int age;

    protected Person(String name, int age) {
        this.name = name;
        this.age = checkedAge(age);
    }

    private static int checkedAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("年龄不能小于 0");
        }
        return age;
    }

    public final String getName() {
        return name;
    }

    public final int getAge() {
        return age;
    }

    public final void setAge(int age) {
        this.age = checkedAge(age);
    }

    public abstract void introduce();

    @Override
    public String toString() {
        return "姓名：" + name + "，年龄：" + age;
    }
}

class Student extends Person implements Singable {
    private static int count = 0;

    public Student(String name) {
        this(name, 0);
    }

    public Student(String name, int age) {
        super(name, age);
        Student.count++;
    }

    public static int getCount() {
        return Student.count;
    }

    @Override
    public void introduce() {
        System.out.println("我叫" + getName() + "，我是一名学生");
    }

    @Override
    public void sing() {
        System.out.println(getName() + "在唱歌");
    }

    public void study() {
        System.out.println(getName() + "在学习");
    }
}

public class Main {
    public static void welcome(Person person) {
        person.introduce();
    }

    public static void perform(Singable singer) {
        singer.sing();
        singer.thankAudience();
    }

    public static void main(String[] args) {
        Student student = new Student("小明", 18);
        student.setAge(20);

        Person person = student;
        Singable singer = student;

        welcome(person);
        perform(singer);
        student.study();

        System.out.println(student);
        System.out.println("累计创建学生数：" + Student.getCount());
    }
}
~~~

预期输出：

~~~text
我叫小明，我是一名学生
小明在唱歌
谢谢大家
小明在学习
姓名：小明，年龄：20
累计创建学生数：1
~~~

这里 student、person 和 singer 指向同一个学生对象。不同的声明类型决定可调用的接口，实际对象决定重写实例方法的执行版本。

## 15. 自测

可以先自己回答，再展开答案。

<details>
<summary>1. Student b = a; 会复制一个学生对象吗？</summary>

不会，只是复制引用。a 和 b 会引用同一个对象。

</details>

<details>
<summary>2. Person p = new Student(...); 调用重写方法时执行谁的版本？</summary>

执行实际对象 Student 的版本。如果 Student 没有重写相应方法，则使用继承来的实现。这个规则讨论的是可被重写的实例方法。

</details>

<details>
<summary>3. 执行 super.introduce() 时，父类方法中的 this 指向谁？</summary>

仍然指向当前学生对象。super 选择父类实现，不会新建父类对象。

</details>

<details>
<summary>4. static 属性与局部变量同名，怎样明确访问静态属性？</summary>

使用类名，例如 Student.count。访问同名实例属性则使用 this.属性名。

</details>

<details>
<summary>5. final Student s 为什么仍然可能允许修改年龄？</summary>

final 限制 s 重新赋值，不限制对象本身的内部状态。能否修改年龄，还取决于对象提供的字段和方法。

</details>

<details>
<summary>6. 一个具体类实现了两个接口，需要实现几个方法？</summary>

需要满足两个接口的全部方法契约。已有继承实现或 default 实现的方法不一定要重新实现；默认方法冲突时还需要明确解决冲突。

</details>
