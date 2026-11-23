# Java 集合进阶：HashSet 去重与 HashMap 查找

[返回目录](README.md) · [上一篇：ArrayList](Java集合入门ArrayList.md) · [下一篇：相等与排序](Java对象相等与集合排序.md)

已经会用 ArrayList 保存多个元素后，接下来考虑两个问题：怎样自动去重？怎样根据学号查到姓名？

本文短代码默认放在方法内，并导入对应的 java.util 类型。示例使用 Java 8 及以上语法；文末完整程序可单独运行。

## 目录

- [1. 先分清接口与实现类](#1-先分清接口与实现类)
- [2. HashSet：重复元素只保存一份](#2-hashset重复元素只保存一份)
- [3. HashMap：根据键找到值](#3-hashmap根据键找到值)
- [4. 遍历 Map](#4-遍历-map)
- [5. 用 Map 计数](#5-用-map-计数)
- [6. 有序版本怎么选择](#6-有序版本怎么选择)
- [7. 完整示例：去重与计数](#7-完整示例去重与计数)

## 1. 先分清接口与实现类

常见写法：

~~~java
List<String> names = new ArrayList<>();
Set<String> uniqueNames = new HashSet<>();
Map<Integer, String> students = new HashMap<>();
~~~

左边的 List、Set、Map 是接口，右边是具体实现类。与 Person p = new Student() 一样，是通过共同类型使用具体对象。

| 接口 | 主要用途 | 常见实现 |
|---|---|---|
| List | 按位置保存元素，允许重复 | ArrayList、LinkedList |
| Set | 保存不重复的元素 | HashSet、LinkedHashSet、TreeSet |
| Map | 保存键与值的对应关系 | HashMap、LinkedHashMap、TreeMap |

Map 不是 Collection 的子接口；List、Set 则属于 Collection 体系。这里的 Collection 是集合接口，Collections 是工具类，不要混淆。

## 2. HashSet：重复元素只保存一份

~~~java
Set<String> names = new HashSet<>();
System.out.println(names.add("小明")); // true
System.out.println(names.add("小红")); // true
System.out.println(names.add("小明")); // false

System.out.println(names.size());         // 2
System.out.println(names.contains("小明")); // true
System.out.println(names.remove("小红"));  // true
~~~

add 返回是否真的加入新元素。contains 判断是否存在，remove 删除匹配元素，size 查看数量。

HashSet 没有下标，不能 get(0)。遍历时：

~~~java
for (String name : names) {
    System.out.println(name);
}
~~~

HashSet 不保证遍历顺序。偶然看到与添加顺序相同的输出，也不能依赖这一点。“不保证顺序”并不等于每次都随机打乱。

### 从列表去重

~~~java
List<String> input = new ArrayList<>();
input.add("小明");
input.add("小红");
input.add("小明");

Set<String> unique = new HashSet<>(input);
~~~

如果希望去重后保留第一次出现的先后顺序：

~~~java
Set<String> uniqueInOrder = new LinkedHashSet<>(input);
System.out.println(uniqueInOrder); // [小明, 小红]
~~~

这里使用默认的插入顺序。重新添加已存在元素，不会把它移动到最后。

如果希望按字符串的自然顺序遍历，可用 TreeSet。自然顺序不等于中文拼音顺序。

## 3. HashMap：根据键找到值

把学号作为键，把姓名作为值：

~~~java
Map<Integer, String> students = new HashMap<>();

students.put(1001, "小明");
students.put(1002, "小红");

System.out.println(students.get(1001)); // 小明
~~~

Map<K, V> 中，K 是键类型，V 是值类型。泛型不能直接写 int，所以学号用 Integer；传入 1001 时会自动装箱。

键不能重复，值可以重复：

~~~java
students.put(1001, "小华");
System.out.println(students.get(1001)); // 小华
System.out.println(students.size());   // 2
~~~

同一个键再次 put，会替换对应的值，不会多出一条记录。

| 操作 | 方法 |
|---|---|
| 添加或替换 | put(key, value) |
| 按键读取 | get(key) |
| 判断键是否存在 | containsKey(key) |
| 判断是否包含某个值 | containsValue(value) |
| 按键删除 | remove(key) |
| 映射数量 | size() |
| 清空 | clear() |

containsValue 通常需要遍历查找，不能认为与按键 get 一样高效。

HashMap 允许一个 null 键，也允许 null 值，但不保证遍历顺序。基础 put/get 在散列分布良好时具有预期常数时间表现，不能对所有情况承诺最坏 O(1)。参见 [HashMap 官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html)。

### 查不到时怎么办

~~~java
String name = students.get(9999); // null
~~~

如果 Map 允许保存 null，get 返回 null 既可能表示键不存在，也可能表示键存在但值为 null。判断是否存在用 containsKey。

~~~java
if (students.containsKey(9999)) {
    System.out.println(students.get(9999));
} else {
    System.out.println("没有这个学号");
}
~~~

也可以为缺失键提供默认值：

~~~java
String name = students.getOrDefault(9999, "未知学生");
~~~

getOrDefault 只在没有该键时使用默认值；已有键对应 null 时，仍返回 null。

不要直接把可能为 null 的结果拆箱：

~~~java
Map<String, Integer> scores = new HashMap<>();
// int score = scores.get("小明"); // 缺失时拆箱会抛出 NullPointerException
int score = scores.getOrDefault("小明", 0);
~~~

这个 scores 示例约定不存 null 分数。

## 4. 遍历 Map

只需要键：

~~~java
for (Integer id : students.keySet()) {
    System.out.println(id);
}
~~~

只需要值：

~~~java
for (String name : students.values()) {
    System.out.println(name);
}
~~~

同时需要键和值：

~~~java
for (Map.Entry<Integer, String> entry : students.entrySet()) {
    System.out.println(entry.getKey() + "：" + entry.getValue());
}
~~~

Map.Entry<Integer, String> 表示一组键值对；getKey 取键，getValue 取值。通常优先用 entrySet 遍历键值对。

keySet、values、entrySet 是与原 Map 关联的视图，不是独立副本。不要一边普通增强 for 遍历，一边随意对原 Map 增删键。

## 5. 用 Map 计数

统计每个姓名出现的次数：

~~~java
Map<String, Integer> counts = new HashMap<>();

for (String name : names) {
    int oldCount = counts.getOrDefault(name, 0);
    counts.put(name, oldCount + 1);
}
~~~

names 在这里应是保留重复项的列表。第一次出现时从 0 开始，再加一；之后读取旧次数再累加。

可缩写为：

~~~java
counts.put(name, counts.getOrDefault(name, 0) + 1);
~~~

## 6. 有序版本怎么选择

| 实现 | 常见遍历顺序 |
|---|---|
| HashSet / HashMap | 不保证顺序 |
| 默认 LinkedHashSet / LinkedHashMap | 插入顺序 |
| TreeSet / TreeMap | 自然顺序或指定比较器的顺序 |

TreeMap 按键排序，不按值排序。TreeSet 和 TreeMap 的基本查找、增删通常为 O(log n)，适合有序查找。TreeMap 还可以用 floorKey、ceilingKey 查找边界键：

~~~java
TreeMap<Integer, String> table = new TreeMap<>();
table.put(10, "A");
table.put(20, "B");

System.out.println(table.floorKey(15));   // 10，最大的 <=15 的键
System.out.println(table.ceilingKey(15)); // 20，最小的 >=15 的键
~~~

自然顺序的 TreeSet/TreeMap 不接受 null 元素或 null 键；如果自行提供支持 null 的比较器，规则另论。初学时统一不要向树结构放 null。

HashSet 的去重与 equals/hashCode 相关，TreeSet 则根据比较结果是否为 0 区分元素。下一篇会详细解释。

## 7. 完整示例：去重与计数

保存为 Main.java：

~~~java
import java.util.ArrayList;
import java.util.LinkedHashSet;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("小明");
        names.add("小红");
        names.add("小明");

        Set<String> unique = new LinkedHashSet<>(names);
        System.out.println("去重：" + unique);

        Map<String, Integer> counts = new LinkedHashMap<>();
        for (String name : names) {
            counts.put(name, counts.getOrDefault(name, 0) + 1);
        }

        for (Map.Entry<String, Integer> entry : counts.entrySet()) {
            System.out.println(entry.getKey() + "：" + entry.getValue());
        }
    }
}
~~~

预期输出：

~~~text
去重：[小明, 小红]
小明：2
小红：1
~~~

这里特意使用 LinkedHashSet/LinkedHashMap，使示例输出顺序确定；换成 HashSet/HashMap 后不要依赖这个顺序。
