# Java 异常处理与文件 IO：从 try-catch 到文本和字节读写

[返回学习目录](README.md) · [上一篇：包装类与泛型](Java包装类与泛型基础.md) · [下一篇：ArrayList](Java集合入门ArrayList.md)

本文整理已经学习过的异常处理和文件操作。涉及 Path.of、Files.readString 和 Files.writeString 的示例使用 Java 11 及以上版本。

短代码需要放入合适的方法中，并补齐对应 import。标有完整程序的示例可以分别保存为 Main.java，不要把多个 Main 类拼在一起。

## 目录

1. [异常与 try-catch](#1-异常与-try-catch)
2. [throw 与 throws](#2-throw-与-throws)
3. [路径与文件夹](#3-路径与文件夹)
4. [一次读取和写入文本](#4-一次读取和写入文本)
5. [逐行读取与自动关闭](#5-逐行读取与自动关闭)
6. [BufferedWriter 与缓冲区](#6-bufferedwriter-与缓冲区)
7. [字节流复制文件](#7-字节流复制文件)
8. [常见问题](#8-常见问题)

## 1. 异常与 try-catch

异常表示程序执行过程中出现了不能按正常流程继续处理的问题。例如整数除以零：

~~~java
int a = 10;
int b = 0;
System.out.println(a / b);
System.out.println("计算结束");
~~~

计算 a / b 时出现 ArithmeticException，正常流程被打断。如果没有捕获，最后一行不会执行。

用 try-catch 可以处理匹配的异常：

~~~java
int a = 10;
int b = 0;

try {
    System.out.println("开始计算");
    System.out.println(a / b);
    System.out.println("计算成功");
} catch (ArithmeticException e) {
    System.out.println("计算失败：除数不能为零");
}

System.out.println("程序继续执行");
~~~

输出：

~~~text
开始计算
计算失败：除数不能为零
程序继续执行
~~~

执行顺序是：进入 try → 出现异常 → 跳过 try 剩余语句 → 执行匹配的 catch → 继续执行后面的代码。

`catch (ArithmeticException e)` 中，前面是要捕获的异常类型，e 是引用异常对象的变量，可以用 `e.getMessage()` 读取原因信息。

没有发生异常时，catch 不执行。catch 也不意味着重新执行失败语句；没有匹配的处理时，异常会继续向外传播。

## 2. throw 与 throws

### throw：真正抛出异常

可以主动报告非法参数：

~~~java
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("年龄不能小于 0");
    }
    this.age = age;
}
~~~

`new IllegalArgumentException(...)` 创建异常对象，throw 抛出它。赋值不会执行，这次方法调用以异常方式结束。

调用方处理：

~~~java
try {
    student.setAge(-5);
    System.out.println("设置成功");
} catch (IllegalArgumentException e) {
    System.out.println("设置失败：" + e.getMessage());
}
~~~

### throws：声明可能向外传递的异常

读取文件的方法可以把异常留给调用方：

~~~java
static String readFile() throws IOException {
    return Files.readString(Path.of("note.txt"));
}
~~~

throws 写在方法声明上，本身既不处理异常，也不制造异常。

IOException 是受检异常。对于可能抛出的这类异常，编译器要求捕获，或者通过 throws 声明向外传递。ArithmeticException、IllegalArgumentException 等属于运行时异常，不强制这样声明，但仍可能需要处理。

完整程序：

~~~java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    static String readFile() throws IOException {
        return Files.readString(Path.of("note.txt"));
    }

    public static void main(String[] args) {
        try {
            String content = readFile();
            System.out.println(content);
        } catch (IOException e) {
            System.out.println("读取文件失败：" + e.getMessage());
        }
        System.out.println("程序继续执行");
    }
}
~~~

文件不存在等情况下，IOException 从读取方法传播到 readFile，再被 main 中的 catch 处理。

## 3. 路径与文件夹

### 相对路径

~~~java
Path file = Path.of("note.txt");
~~~

相对路径以程序的当前工作目录为起点，不一定以源文件所在目录为起点。IDE 的运行配置也会影响工作目录。

打印实际位置：

~~~java
System.out.println(Path.of("").toAbsolutePath());
System.out.println(file.toAbsolutePath());
~~~

子目录可以分段指定：

~~~java
Path file = Path.of("data", "students", "note.txt");
~~~

### 绝对路径

Windows 下可以写：

~~~java
Path a = Path.of("C:/data/note.txt");
Path b = Path.of("C:\\data\\note.txt");
~~~

字符串中的反斜杠需要转义。绝对路径依赖具体环境，换电脑时不一定可用。

创建 Path 对象只是在描述位置，不会创建文件或目录。

### 创建目录和文件

~~~java
Path directory = Path.of("data", "students");
Files.createDirectories(directory);

Path file = directory.resolve("note.txt");
Files.writeString(file, "小明，18岁");
~~~

createDirectories 创建缺少的各级目录；目录已经存在时通常可以继续。如果路径中需要作为目录的位置实际是普通文件，就会失败。

resolve 在目录路径后连接相对文件名，这里得到 data/students/note.txt。

如果只需要空文件：

~~~java
Files.createFile(file);
~~~

createFile 要求目标文件不存在，已存在则抛出异常。writeString 默认可以创建新文件，也会覆盖已有文件，所以写文本前不需要先 createFile。

两者都不会自动创建缺失的父目录。

## 4. 一次读取和写入文本

需要导入：

~~~java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
~~~

在处理 IOException 的前提下：

~~~java
Path file = Path.of("note.txt");

// 写入：默认创建或覆盖
Files.writeString(file, "你好，Java");

// 读取：一次得到整个文本
String content = Files.readString(file);
System.out.println(content);
~~~

这两个方法默认使用 UTF-8 编码，并在操作中自行管理所需的文件资源。

连续两次默认写入，后一次会覆盖前一次：

~~~java
Files.writeString(file, "第一段");
Files.writeString(file, "第二段");
// 最终只有“第二段”
~~~

追加内容要指定选项：

~~~java
Files.writeString(
    file,
    "\n新的一行",
    StandardOpenOption.CREATE,
    StandardOpenOption.APPEND
);
~~~

CREATE 表示不存在则创建；APPEND 表示追加到末尾。追加不会自动换行，示例中显式使用了 \n。

完整的创建目录并保存文本示例：

~~~java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) {
        try {
            Path directory = Path.of("data", "students");
            Files.createDirectories(directory);

            Path file = directory.resolve("note.txt");
            Files.writeString(file, "小明，18岁");

            System.out.println("保存成功：" + file.toAbsolutePath());
        } catch (IOException e) {
            System.out.println("保存失败：" + e.getMessage());
        }
    }
}
~~~

## 5. 逐行读取与自动关闭

readString 一次把整个文本放进内存。如果希望逐行处理，可以使用 BufferedReader。

完整程序：

~~~java
import java.io.BufferedReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) {
        try (BufferedReader reader =
                 Files.newBufferedReader(Path.of("note.txt"))) {

            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }

        } catch (IOException e) {
            System.out.println("读取失败：" + e.getMessage());
        }
    }
}
~~~

readLine 每次返回下一行文本，不包含行末换行符：

- 普通行：返回该行字符串。
- 空行：返回空字符串 ""。
- 文件末尾：返回 null。

循环条件先把读取结果赋给 line，再检查是否为 null。

这里的 `try (...)` 叫 try-with-resources。资源打开成功后，无论代码块正常结束还是发生异常，都会自动尝试关闭。无需在循环后手工调用 close。

逐行处理通常可以降低整文件载入的内存需求，但单独一行本身仍会被读取为字符串。

## 6. BufferedWriter 与缓冲区

完整程序：

~~~java
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) {
        try (BufferedWriter writer =
                 Files.newBufferedWriter(Path.of("note.txt"))) {

            writer.write("小明");
            writer.newLine();
            writer.write("小红");
            writer.newLine();
            writer.write("小华");

        } catch (IOException e) {
            System.out.println("写入失败：" + e.getMessage());
        }
    }
}
~~~

默认使用 UTF-8，创建目标文件或清空已有内容，然后写入。父目录需要已经存在。

| 方法 | 用途 |
|---|---|
| write("文字") | 写入文字，不自动换行 |
| newLine() | 写入当前系统的换行符 |
| flush() | 把缓冲的数据向底层写出 |
| close() | 关闭写入器，并刷新缓冲区 |

缓冲区可以暂存数据，减少频繁的底层写入。调用 write 后，不应假定全部数据都已向底层写出。正常关闭会刷新，因此不需要每写一行就 flush。

flush 不等于承诺数据已经持久保存到物理磁盘。这里先掌握文本读写，不涉及更底层的持久化控制。

## 7. 字节流复制文件

Reader、Writer 处理字符；InputStream、OutputStream 处理原始字节。图片、音频等文件不应当通过读成字符串再写回的方式复制。

Input 和 Output 以程序为中心：读入程序叫输入，写出程序叫输出。

完整程序，运行前在工作目录准备 photo.jpg，并确保目标不是源文件本身：

~~~java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) {
        try (
            InputStream input =
                Files.newInputStream(Path.of("photo.jpg"));
            OutputStream output =
                Files.newOutputStream(Path.of("photo-copy.jpg"))
        ) {
            byte[] buffer = new byte[4096];
            int count;

            while ((count = input.read(buffer)) != -1) {
                output.write(buffer, 0, count);
            }
        } catch (IOException e) {
            System.out.println("复制失败：" + e.getMessage());
        }
    }
}
~~~

byte 是基本类型，数组每次最多容纳 4096 个字节。read 返回实际读取数量，不保证填满数组；返回 -1 表示结束。

`output.write(buffer, 0, count)` 表示从下标 0 开始，只写本次有效的 count 个字节。如果最后一次只读到 100 字节，却写入整个数组，就可能把上一轮残留内容也写出去。

默认创建或覆盖目标文件，两个流会在离开 try 时自动关闭。失败的复制可能留下不完整的目标文件，catch 不会自动撤销已经写入的内容。

## 8. 常见问题

| 问题 | 首先检查 |
|---|---|
| 明明有文件却读不到 | 打印 toAbsolutePath，确认工作目录 |
| 写入子目录失败 | 父目录是否存在，是否需要 createDirectories |
| 原内容消失 | 是否使用了默认覆盖模式 |
| 读到空行就停止 | 是否错误地把 "" 当成文件末尾 |
| 文件复制末尾出现多余数据 | 是否只写 read 返回的有效字节数 |
| 写完却看不到完整数据 | 是否正确关闭写入器或按需 flush |
| 使用 Path 就以为创建了文件 | Path 仅表示位置，实际操作由 Files 或流完成 |

学习到这里，已经可以进行基础文本保存、逐行处理和二进制文件复制。后续可以把这些操作与学生对象、集合结合使用。
