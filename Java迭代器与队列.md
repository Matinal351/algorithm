# Java 迭代器与队列：Iterator、Deque、PriorityQueue 和集合选型

[返回目录](README.md) · [上一篇：相等与排序](Java对象相等与集合排序.md)

本篇补齐常用集合的遍历删除、队列、栈和优先队列，并整理如何选择集合。短代码默认在方法中执行，按需导入 java.util 中对应的类和接口；示例采用 Java 8 及以上语法。

## 目录

- [1. Iterator：控制遍历过程](#1-iterator控制遍历过程)
- [2. Queue：先进先出](#2-queue先进先出)
- [3. Deque：两端都可以操作](#3-deque两端都可以操作)
- [4. PriorityQueue：按优先级取元素](#4-priorityqueue按优先级取元素)
- [5. 常见操作的成本](#5-常见操作的成本)
- [6. 复制、视图与可修改性](#6-复制视图与可修改性)
- [7. 完整示例：删除、排队与优先处理](#7-完整示例删除排队与优先处理)
- [8. 选型速查与练习](#8-选型速查与练习)

## 1. Iterator：控制遍历过程

增强 for 背后使用迭代器。手工写出来：

~~~java
Iterator<String> iterator = names.iterator();

while (iterator.hasNext()) {
    String name = iterator.next();
    System.out.println(name);
}
~~~

names 是一个 List<String>。hasNext 判断是否还有元素，next 取出下一个元素，并前进。

当需要边遍历边删除时，使用当前迭代器自己的 remove：

~~~java
Iterator<String> iterator = names.iterator();

while (iterator.hasNext()) {
    String name = iterator.next();
    if ("小明".equals(name)) {
        iterator.remove();
    }
}
~~~

remove 删除最近一次 next 返回的元素。不能在第一次 next 之前 remove，也不能对同一次 next 连续 remove 两次。并非所有迭代器都支持删除，例如某些不可修改集合的迭代器不支持。

对于 ArrayList 等常用可修改集合，这种写法可避免直接 names.remove 干扰迭代器。它保证删除语义正确，但不意味着 ArrayList 大量删除一定高效。

### removeIf

也可以用条件删除：

~~~java
names.removeIf(name -> "小明".equals(name));
~~~

Lambda 接收一个元素，返回 true 表示应删除。这个参数对应 Predicate<T> 接口，其核心方法 test 返回 boolean。

HashMap 遍历时删除也可以通过条目迭代器：

~~~java
Iterator<Map.Entry<Integer, String>> iterator =
    students.entrySet().iterator();

while (iterator.hasNext()) {
    Map.Entry<Integer, String> entry = iterator.next();
    if (entry.getKey() < 1000) {
        iterator.remove();
    }
}
~~~

这里 students 是可修改的 HashMap。不要把 ConcurrentModificationException 当作只会在多线程中出现的异常；单线程里错误修改集合也可能触发它，检测本身也不是线程安全保证。

## 2. Queue：先进先出

队列适合排队处理任务，先加入的先处理：

~~~java
Queue<String> queue = new ArrayDeque<>();

queue.offer("小明");
queue.offer("小红");

System.out.println(queue.peek()); // 小明，只看队首
System.out.println(queue.poll()); // 小明，取出并删除
System.out.println(queue.poll()); // 小红
System.out.println(queue.poll()); // null，队列为空
~~~

Queue 是接口，ArrayDeque 是具体实现。用 ArrayDeque 时，这种尾部加入、头部取出的方式构成普通先进先出队列；并不是所有 Queue 实现都按先进先出，PriorityQueue 就按优先级。

| 用途 | 常用方法 | 空或失败时 |
|---|---|---|
| 入队 | offer(e) | 有容量限制的队列无法接收时可返回 false |
| 取出队首 | poll() | 空队列返回 null |
| 查看队首 | peek() | 空队列返回 null |
| 取出队首 | remove() | 空队列抛出异常 |
| 查看队首 | element() | 空队列抛出异常 |

初学时优先使用 offer、poll、peek。没有参数的 remove() 与 remove(Object) 不是同一个操作。

ArrayDeque 不允许 null，因此 poll 返回 null 可以清楚表示队列为空。

## 3. Deque：两端都可以操作

Deque 是双端队列，ArrayDeque 也实现它：

~~~java
Deque<String> deque = new ArrayDeque<>();
deque.offerLast("小明");
deque.offerLast("小红");
deque.offerFirst("小华");

// 当前从头到尾：[小华, 小明, 小红]
System.out.println(deque.pollFirst()); // 小华
System.out.println(deque.pollLast());  // 小红
~~~

| 队头 | 队尾 |
|---|---|
| offerFirst | offerLast |
| pollFirst | pollLast |
| peekFirst | peekLast |

需要普通队列时可用 offerLast + pollFirst，便于明确方向。

### 用作栈

栈是后进先出：

~~~java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(10);
stack.push(20);

System.out.println(stack.peek()); // 20
System.out.println(stack.pop());  // 20
System.out.println(stack.pop());  // 10
~~~

push、pop 在同一端工作。pop 在空栈时抛出异常，调用前可以检查 isEmpty。需要空时返回 null，可以使用 pollFirst。

新代码中一般优先使用 Deque/ArrayDeque 实现栈，而不是旧的 Stack 类。

## 4. PriorityQueue：按优先级取元素

普通队列看进入顺序，优先队列看比较规则。

~~~java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5);
pq.offer(2);
pq.offer(8);

System.out.println(pq.peek()); // 2
System.out.println(pq.poll()); // 2
System.out.println(pq.poll()); // 5
System.out.println(pq.poll()); // 8
~~~

Java 的 PriorityQueue 默认按自然顺序让最小元素先出队，即小顶堆。

需要最大元素先出队：

~~~java
PriorityQueue<Integer> pq =
    new PriorityQueue<>(Comparator.reverseOrder());

pq.offer(5);
pq.offer(2);
pq.offer(8);

System.out.println(pq.poll()); // 8
~~~

这与某些 C++ priority_queue 的默认行为不同，不要直接套用。

### 不保证遍历有序

~~~java
for (Integer value : pq) {
    System.out.println(value);
}
~~~

这种遍历不保证从小到大或从大到小。需要按优先级依次取出时，反复 poll：

~~~java
while (!pq.isEmpty()) {
    System.out.println(pq.poll());
}
~~~

这会清空队列。如果想保留原队列，可先创建副本，再从副本中取：

~~~java
PriorityQueue<Integer> copy = new PriorityQueue<>(pq);
while (!copy.isEmpty()) {
    System.out.println(copy.poll());
}
~~~

PriorityQueue 不允许 null；比较相等的元素允许重复，先后顺序没有保证。它也不自动提供同优先级任务先进先出的稳定性，需要时把入队序号纳入比较规则。参见 [PriorityQueue 官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/PriorityQueue.html)。

### 对象优先级

如果 Task 有 getPriority 方法：

~~~java
PriorityQueue<Task> tasks = new PriorityQueue<>(
    Comparator.comparingInt(Task::getPriority).reversed()
);
~~~

这里把较大的 priority 视为更高优先级。比较器根据业务定义即可。

对象入堆后，不要直接修改用于排序的字段并期待堆自动重排。可在仍能正确找到对象时先移除，再修改、重新加入；或者设计为不可变任务并加入新记录。

## 5. 常见操作的成本

设 n 为元素数量，以下为常见实现的基本表现：

| 实现 | 典型成本 |
|---|---|
| ArrayList | 按下标读取 O(1)，尾部添加摊还 O(1)，中间插删 O(n) |
| LinkedList | get(i) 通常 O(n)；找到节点/持有相应迭代器后局部插删 O(1) |
| HashSet / HashMap | 散列良好时基本增删查预期 O(1) |
| TreeSet / TreeMap | 基本增删查 O(log n) |
| ArrayDeque | 两端添加删除通常为摊还 O(1) |
| PriorityQueue | offer/poll O(log n)，peek O(1)，contains/remove(Object) O(n) |

不要认为 LinkedList 在任何“经常删除”的场景中都更快：如果要先按下标找到位置，查找本身可能需要线性时间。

PriorityQueue 的 remove(Object) 是按对象匹配删除，不是“高效删除任意堆节点”的接口。

## 6. 复制、视图与可修改性

### 容器复制不是对象深拷贝

~~~java
List<Student> copy = new ArrayList<>(students);
~~~

这会创建一个独立列表，但内部仍然引用相同学生。给 copy 增删元素不改变原列表结构；修改同一个学生对象的属性，则两边都能看到。

### Arrays.asList

~~~java
List<String> names = Arrays.asList("小明", "小红");
names.set(0, "小华"); // 可以
// names.add("小丽"); // 不支持改变长度
~~~

它是固定大小、由数组支持的列表，不是普通可扩容 ArrayList。想要可扩容：

~~~java
List<String> names =
    new ArrayList<>(Arrays.asList("小明", "小红"));
~~~

如果传入 int[]，Arrays.asList 得到的是保存这个数组的单元素 List<int[]>，不是自动拆成 List<Integer>。

### 不可修改列表

Java 9 及以上可使用：

~~~java
List<String> names = List.of("小明", "小红");
// names.add("小华"); // 不支持
// names.set(0, "小丽"); // 也不支持
~~~

List.of 也不允许 null。不可修改容器不意味着其引用的每个对象都不可变。

需要从原列表获得一段范围时，subList 返回关联视图，不是独立副本。若想独立保存该范围：

~~~java
List<String> part = new ArrayList<>(names.subList(0, 1));
~~~

范围左闭右开，即包含 0，不包含 1。

## 7. 完整示例：删除、排队与优先处理

保存为 Main.java：

~~~java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.Deque;
import java.util.Iterator;
import java.util.List;
import java.util.PriorityQueue;
import java.util.Queue;

public class Main {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(
            Arrays.asList("小明", "小红", "小明")
        );

        Iterator<String> iterator = names.iterator();
        while (iterator.hasNext()) {
            if ("小明".equals(iterator.next())) {
                iterator.remove();
            }
        }
        System.out.println(names);

        Queue<String> queue = new ArrayDeque<>();
        queue.offer("任务A");
        queue.offer("任务B");
        while (!queue.isEmpty()) {
            System.out.println(queue.poll());
        }

        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(10);
        stack.push(20);
        System.out.println("栈顶：" + stack.pop());

        PriorityQueue<Integer> pq =
            new PriorityQueue<>(Comparator.reverseOrder());
        pq.offer(3);
        pq.offer(1);
        pq.offer(5);
        while (!pq.isEmpty()) {
            System.out.println("优先处理：" + pq.poll());
        }
    }
}
~~~

预期输出：

~~~text
[小红]
任务A
任务B
栈顶：20
优先处理：5
优先处理：3
优先处理：1
~~~

## 8. 选型速查与练习

| 需求 | 通常先考虑 |
|---|---|
| 按位置保存、遍历、允许重复 | ArrayList |
| 去重，不关心顺序 | HashSet |
| 去重并保留首次出现顺序 | LinkedHashSet |
| 根据学号查学生 | HashMap |
| 键值映射按默认插入顺序遍历 | LinkedHashMap |
| 有序去重、查相邻边界 | TreeSet / TreeMap |
| 先进先出排队 | ArrayDeque 作为 Queue |
| 后进先出栈 | ArrayDeque 作为 Deque |
| 总是先处理最小或最大元素 | PriorityQueue |

这些常见可修改实现通常不是线程安全的；并发集合与多线程可以另作一个主题学习。

练习建议：

1. 用 Map<Integer, Student> 建立按学号查找的学生表。
2. 用 LinkedHashSet 对姓名去重，并保持输入顺序。
3. 把学生保存到 List，按年龄升序、学号升序排列。
4. 用 Queue 模拟排队，用 PriorityQueue 模拟按分数优先处理。
5. 用 Iterator 或 removeIf 删除不满足条件的记录。

Stream、泛型通配符和并发集合还可以继续深入，但它们属于后续专题。本系列至此覆盖日常入门最常用的集合接口、实现、相等规则、排序和安全遍历方式。
