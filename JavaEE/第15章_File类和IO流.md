[TOC]

# 第15章_File类和IO流

## 1. java.io.File类的使用

### 1.1 概述

- File类及本章下的各种流，都定义在java.io包下
- 一个File对象代表硬盘或网络中可能存在的一个文件或者文件目录（俗称文件夹），与平台无关。（体会万事万物皆对象）
- File能新建、删除、重命名文件和目录，但File不能访问文件内容本身。如果需要访问文件内容本身，则需要使用输入/输出流
  - File对象可以作为参数传递给流的构造器
- 想要在Java程序中表示一个真实存在的文件或目录，那么必须要有一个File对象，但是Java程序中的一个File对象，可能没有一个真实存在的文件或目录。

### 1.2 构造器

- `public File(String pathname)`：以pathname为路径创建File对象，可以是绝对路径或是相对路径，如果pathname是相对路径，则默认的当前路径在系统属性user.dir中存储
- `public File(String parent, String child)`：以parent为父路径，child为子路径创建File对象
- `public File(File parent, String child)`：根据一个父File对象和子文件路径创建File对象

关于路径：

- **绝对路径：**从盘符开始的路径，这是一个完整的路径
- **相对路径：**相对于`项目目录`的路径，这时一个便捷的路径，开发中经常使用
  - IDEA中，main中的文件的相对路径，是相对于“`当前工程`”
  - IDEA中，单元测试方法中的文件的相对路径，是相对于“`当前module`”

举例：

```java
public class FileObjectTest {
    public static void main(String[] args){
        //文件路径名
        String pathname = "D:/aaa.txt";
        File file1 = new File(pathname);
        
        //文件路径名
        String pathname2 = "D:/aaa/bbb.txt";
        File file2 = new File(pathname2);
        
        //通过父路径和子路径字符串
        String parent = "D:/aaa";
        String child = "bbb.txt";
        
        //通过父级File对象和子路径字符串
        File parentDir = new File("D:/aaa");
        String childFile = "bbb.txt";
        File file4 = new File(parentDir, childFile);
        
    }
    @Test
    public void test01() throws IOException {
        File f1 = new File("D:/Material/Java");     //绝对路径
        System.out.println("文件/目录的名称：" + f1.getName());
        System.out.println("文件/目录的构造路径名：" + f1.getPath());
        System.out.println("文件/目录的绝对路径名：" + f1.getAbsolutePath());
        System.out.println("文件/目录的父目录名：" + f1.getParent());
    }
    文件/目录的名称：Java
	文件/目录的构造路径名：D:\Material\Java
	文件/目录的绝对路径名：D:\Material\Java
	文件/目录的父目录名：D:\Material

    @Test
    public void test02() throws IOException{
        File f2 = new File("/用户/86134/aaa.txt");    //绝对路径，从根目录开始
        System.out.println("文件/目录的名称：" + f2.getName());
        System.out.println("文件/目录的构造路径名：" + f2.getPath());
        System.out.println("文件/目录的绝对路径名：" + f2.getAbsolutePath());
        System.out.println("文件/目录的父目录名：" + f2.getParent());
    }
    文件/目录的名称：aaa.txt
    文件/目录的构造路径名：\用户\86134\aaa.txt
    文件/目录的绝对路径名：D:\用户\86134\aaa.txt
    文件/目录的父目录名：\用户\86134

    @Test
    public void test03() throws IOException{
        File f3 = new File("aaa.txt");    //相对路径
        System.out.println("user.dir = " + System.getProperty("user.dir"));
        System.out.println("文件/目录的名称：" + f3.getName());
        System.out.println("文件/目录的构造路径名：" + f3.getPath());
        System.out.println("文件/目录的绝对路径名：" + f3.getAbsolutePath());
        System.out.println("文件/目录的父目录名：" + f3.getParent());
    }
    user.dir = D:\Develop\IDEA\JavaSECode\chapter15_io
    文件/目录的名称：aaa.txt
    文件/目录的构造路径名：aaa.txt
    文件/目录的绝对路径名：D:\Develop\IDEA\JavaSECode\chapter15_io\aaa.txt
    文件/目录的父目录名：null
}
```

> 注意：
>
>  1. 无论该路径下是否存在文件或目录，都不影响File对象的创建
>
>  2. window的路径分隔符使用"\\"，而Java程序中的"\\"表示转义字符，所以在Windows中表示路径，需要用"\\\"。或者直接使用"/"也可以，Java程序支持将"/"当成平台无关的`路径分隔符`。或者直接使用File.seperator常量值表示。比如：
>
>     File file2 = new File("D:" + File.seperator + "Material" + File.seperator + "Java");
>
> 	3. 当构造路径是绝对路径时，那么getPath和getAbsolutePath的结果一样
>
>     当构造路径是相对路径时，那么getAbsolutePath的路径=user.dir的路径+构造路径

### 1.3 常用方法

#### 1、获取文件和目录基本信息

- public String getName()：获取名称
- public String getPath()：获取路径
- `public String getAbsolutePath()`：获取绝对路径
- public File getAbsoluteFile()：获取绝对路径表示的文件
- `public String getParent()`：获取上层文件目录路径。若无，返回null
- public long length()：获取文件长度（即：字节数）。不能获取目录的长度
- public long lastModified()：获取最后一次的修改时间，毫秒值

> 如果File对象代表的文件或目录存在，则File对象实例初始化时，就会用硬盘中对应文件或目录的属性信息（例如，时间、类型等）为File对象的属性赋值，否则除了路径和名称，File对象的其他属性值将会保留默认值

举例：

```java
@Test
public void test1(){
    File f1 = new File("hello.txt");    //相对路径
    System.out.println("user.dir = " + System.getProperty("user.dir"));
    System.out.println("文件/目录的名称：" + f1.getName());
    System.out.println("文件/目录的构造路径名：" + f1.getPath());
    System.out.println("文件/目录的绝对路径名：" + f1.getAbsolutePath());
    System.out.println("文件/目录的父目录名：" + f1.getParent());
    System.out.println(f1.getAbsoluteFile().getParent());
    System.out.println("文件长度：" + f1.length());
    System.out.println("文件最后修改时间：" + f1.lastModified());
    System.out.println("文件最后修改时间：" + 						LocalDateTime.ofInstant(Instant.ofEpochMilli(f1.lastModified()),ZoneId.of("Asia/Shanghai")));
    }

user.dir = D:\Develop\IDEA\JavaSECode\chapter15_io
文件/目录的名称：hello.txt
文件/目录的构造路径名：hello.txt
文件/目录的绝对路径名：D:\Develop\IDEA\JavaSECode\chapter15_io\hello.txt
文件/目录的父目录名：null
D:\Develop\IDEA\JavaSECode\chapter15_io
文件长度：6
文件最后修改时间：1732239889705
文件最后修改时间：2024-11-22T09:44:49.705
```

#### 2、列出目录的下一级

- public String[] list()：返回一个String数组，表示该File目录中的所有子文件或目录
- public File[] listFiles()：返回一个File数组，表示该File目录中的所有子文件或目录

举例：

```java
@Test
public void test2(){
    File f1 = new File("D:/Material");
    String[] fileArr = f1.list();
    for (String s : fileArr){
        System.out.println(s);
    }

    System.out.println();

    File[] files = f1.listFiles();
    for (File f : files){
        System.out.println(f);
    }
}

hadoop
Java
LaTeX
Linux.md
Mindspore
Python
python数学建模材料
R
SQL Course Materials
Untitled-2.ipynb
us_states.json
大数据实践A
泰迪杯

D:\Material\hadoop
D:\Material\Java
D:\Material\LaTeX
D:\Material\Linux.md
D:\Material\Mindspore
D:\Material\Python
D:\Material\python数学建模材料
D:\Material\R
D:\Material\SQL Course Materials
D:\Material\Untitled-2.ipynb
D:\Material\us_states.json
D:\Material\大数据实践A
D:\Material\泰迪杯
```

#### 3、File类的重命名功能

- public boolean renameTo(File dest)：把文件重命名为指定的文件路径

#### 4、判断功能的方法

- `public boolean exists()`：此File表示的文件或目录是否实际存在
- `public boolean isDirectory()`：此File表示的是否为目录
- `public boolean isFile()`：此File表示的是否为文件
- public boolean canRead()：判断是否可读
- public boolean canWrite()：判断是否可写
- public boolean isHidden()：判断是否隐藏

举例：

```java
@Test
public void test4(){
    File file1 = new File("D:/Material/Java/abc.txt");
    System.out.println(file1.exists());
    System.out.println(file1.isDirectory());
    System.out.println(file1.isFile());
    System.out.println(file1.canRead());
    System.out.println(file1.canWrite());
    System.out.println(file1.isHidden());

    System.out.println();
    File file2 = new File("D:/Material/Java");
    System.out.println(file2.exists());
    System.out.println(file2.isDirectory());
    System.out.println(file2.isFile());
    System.out.println(file2.canRead());
    System.out.println(file2.canWrite());
    System.out.println(file2.isHidden());
}

true
false
true
true
true
false

true
true
false
true
true
false
```

#### 5、创建、删除功能

- `public boolean createNewFile()`：创建文件，若文件存在，则不创建，返回false

- `public boolean mkdir()`：创建文件目录，如果此文件目录存在，就不创建。如果此文件目录的上层目录不存在，也不创建

- `public boolean mkdirs()`：创建文件目录，如果上层文件目录不存在，一并创建

- `public boolean delete()`：删除文件或文件夹

  删除注意事项：①Java中的删除不走回收站。 ②要删除一个文件目录，请注意该文件目录内不能包含文件或文件目录

举例：

```java
@Test
public void test5() throws IOException {
    File file1 = new File("D:/hello.txt");
    //测试文件的创建、删除
    if (!file1.exists()){
        boolean isSuccessed = file1.createNewFile();
        if (isSuccessed){
            System.out.println("创建成功");
        }
    }else {
        System.out.println("此文件已存在");
        System.out.println(file1.delete()?"文件删除成功":"文件删除失败");
    }
}

@Test
public void test6(){
    File file1 = new File("D:/io");

    file1.mkdir();     //上层目录存在的情况下，mkdir和mkdirs都可以；否则，mkdir不成功
}

@Test
public void test7(){
    File file1 = new File("D:/io");
    System.out.println(file1.delete());    //只有在文件目录中没有任何文件或文件目录的情况下才能删除
}
```

### 1.4 练习



## 2. IO流原理及流的分类

### 2.1 Java IO原理

- Java程序中，对于数据的输入/输出操作以“`流(stream)`”的方式进行，可以看做是一种数据的流动

   ![Java输入输出流 - 知乎](https://pic1.zhimg.com/v2-29f652abce63d9a68eb2bfcaaa8c8bb3_r.jpg) 

   ![javaIO--数据流之IO流与字节流 - wqbin - 博客园](https://img2018.cnblogs.com/blog/1353331/201907/1353331-20190722223956211-245619041.png) 

- I/O流中的I/O是 `Input/Output` 的缩写，I/O技术是非常实用的技术，用于处理设备之间的数据传输。如读/写文件，网络通讯等。

  - `输入input`：读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中
  - `输出output`：将程序（内存）数据输出到磁盘、光盘等存储设备中

### 2.2 流的分类

`java.io` 包下提供了各种”流“类和接口，用以获取不同种类的数据，并通过 `标准的方法` 输入或输出数据

- 按数据的流向不同分为：**输入流**和**输出流**

  - **输入流：**把数据从`其他设备`上读取到`内存`中的流		
    - 以InputStream、Reader结尾
  - **输出流：**把数据从`内存`中写出到`其他设备`上的流
    - 以OutputStream、Writer结尾

- 按操作数据单位的不同分为：**字节流（8bit）**和**字符流（16bit）**

  - **字节流：**以字节为单位，读写数据的流
    - 以InputStream、OutputStream结尾
  - **字符流：**以字符为单位，读写数据的流
    - 以Reader、Writer结尾

- 根据IO流的角色不同分为：**节点流**和**处理流**

  - **节点流：**直接从数据源或目的地读写数据

     ![IO流——节点流和处理流_节点流和处理流览图-CSDN博客](https://img-blog.csdnimg.cn/e9c2b3c37ad8488d879f513a48a77d31.png) 

  - **处理流：**不直接连接到数据源或目的地，而是“连接”在已存在的流（节点流或数据流）之上，通过对数据的处理为程序提供更为强大的读写功能

     ![Java IO节点流和处理流_节点流和处理流的构建-CSDN博客](https://img-blog.csdnimg.cn/1888e2f837dc4fb98c49468600624697.png) 

小结：图解

 ![IO流、File类、节点流、缓冲流、转换流 | 极客之音](https://img-blog.csdnimg.cn/14c3e55ea0324a0a8cba1f4c4f82ac72.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5omY5q-ULemprOWljuWwlA==,size_16,color_FFFFFF,t_70,g_se,x_16) 

### 2.3 流的API

- Java的IO流涉及40多个类，实际上是非常规则，都是从如下4个抽象基类派生的

| （抽象基类） | 输入流      | 输出流       |
| ------------ | ----------- | ------------ |
| 字节流       | InputStream | OutputStream |
| 字符流       | Reader      | Writer       |

- 由这四个类派生出来的子类名称都是一起父类名作为子类名后缀

| 分类       | 字节输入流                                          | 字节输出流                                           | 字符输入流                                     | 字符输出流                                     |
| ---------- | --------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- |
| 抽象基类   | <font color="purple">InputStream</font>             | <font color="purple">OutputStream</font>             | <font color="purple">Reader</font>             | <font color="purple">Writer</font>             |
| 访问文件   | **<font color="black">FileInputStream</font>**      | **<font color="black">FileOutputStream</font>**      | **<font color="black">FileReader</font>**      | **<font color="black">FileWriter</font>**      |
| 访问数组   | **<font color="black">ByteArrayInputStream</font>** | **<font color="black">ByteArrayOutputStream</font>** | **<font color="black">CharArrayReader</font>** | **<font color="black">CharArrayWriter</font>** |
| 访问管道   | **<font color="black">PipedInputStream</font>**     | **<font color="black">PipedOutputStream</font>**     | **<font color="black">PipedReader</font>**     | **<font color="black">PipedWriter</font>**     |
| 访问字符串 |                                                     |                                                      | **<font color="black">StringReader</font>**    | **<font color="black">StringWriter</font>**    |
| 转换流     |                                                     |                                                      | InputStreamReader                              | OutputStreamWriter                             |
| 对象流     | ObjectInputStream                                   | ObjectOutputStream                                   |                                                |                                                |
| 过滤流     | <font color="purple">FilterInputStream</font>       | <font color="purple">FilterOutputStream</font>       | <font color="purple">FilterReader</font>       | <font color="purple">FilterWriter</font>       |
| 缓冲流     | BufferedInputStream                                 | BufferedOutputStream                                 | BufferedReader                                 | BufferedWriter                                 |
| 打印流     |                                                     | PrintStream                                          |                                                | PrintWriter                                    |
| 推回输入流 | PushbackInputStream                                 |                                                      | PushbackReader                                 |                                                |
| 数据流     | DataInputStream                                     | DataOutputStream                                     |                                                |                                                |

其中：

1. <font color="purple">紫色</font>的是抽象基类，是无法new出对象的
2. **<font color="black">粗体</font>**的流称为节点流，必须直接连接真实的物理节点；其他的称为处理流，作用是连接另一个流对其进行包装

**常用的节点流：**

- 文件流：FileInputStream、FileOutputStream、FileReader、FIleWriter
- 字节/字符数组流：ByteArrayInputStream、ByteArrayOutputStream、CharArrayReader、CharArrayWriter
- 

## 3. 节点流之一：FileReader/FileWriter

### 3.1 Reader与Writer



> 常见的文本文件有如下的格式：.txt、.java、.c、.cpp、.py等
>
> 注意：.doc、.xls、.ppt这些都不是文本文件。

#### 3.1.1 字符输入流：Reader

`java.io.Reader` 抽象类是表示用于读取字符流的所有类的父类，可以读取字符信息到内存中。它定义了字符输入流的基本共性功能方法

- `public int read()`：从输入流读取一个字符。虽然读取了一个字符，但是会自动提升为int类型，返回该字符的Unicode编码值。如果已经到达流末尾了，则返回-1
- `public int read(char[] cbuffer)`：从输入流读取一些字符，并将它们存储到字符数组cbuffer中。每次最多读取cbuffer.length个字符。返回实际读取的字符个数。如果已经到达流末尾，没有数据可读，则返回-1
- `public int read(char[] cbuffer, int off, int len)`：从输入流读取一些字符，并将它们存储到字符数组cbuffer中，从cbuffer[off]开始的位置存储。每次最多读取len个字符。返回实际读取的字符个数，如果已经到达流末尾，没有数据可读，则返回-1
- `public void close()`：关闭此流并释放与此流相关联的任何系统资源

> 注意：当完成流的操作时，必须调用close()方法，释放系统资源，否则会造成内存泄漏

#### 3.1.2 字符输出流：Writer

`java.io.Writer` 抽象类是表示用于写入字符流的所有类的父类，将指定的字符信息写到目的地。它定义了字符输出流的基本共性功能方法

- `public void write(int c)`：写入单个字符
- `public void write(char[] cbuffer)`：写出字符数组
- `public void write(char[] cbuffer, int off, int len)`：写入字符数组的某一部分。off：数组的开始索引；len：写入的字符个数
- `public void write(String str)`：写入字符串
- `public void write(String str, int off, int len)`：写入字符串的某一部分。off：字符串的开始索引；len：写入的字符个数
- `public void flush()`：刷新该流的缓冲
- `public void close()`：关闭此流并释放与此流相关联的任何系统资源

> 注意：当完成流的操作时，必须调用close()方法，释放系统资源，否则会造成内存泄漏



### 3.2 FileReader与FileWriter

#### 3.2.1 FileReader

`java.io.FileReader` 类用于读取字符文件，构造是使用系统默认的字符编码和默认字节缓冲区

- `FileReader(File file)`：创建一个新的FileReader，给定要读取的File对象
- `FileReader(String fileName)`：创建一个新的FileReader，给定要读取的文件的名称

**举例：**读取hello.txt文件的字符数据，并显示在控制台上

```java
/**
 * 需求：读取hello.txt中的内容，显示在控制台上
 * 对test2进行优化，每次读取多个字符存放到字符数组中，减少了与磁盘交互的次数，提升效率
 */
@Test
public void test3() {
    FileReader fr = null;
    try {
        //1.创建File类的对象，对应着hello.txt文件
        File file = new File("hello.txt");

        //2.创建输入型的字符流，用于读取数据
        fr = new FileReader(file);

        //3.读取数据，并显示在控制台上
        char[] cbuffer = new char[5];
        int len;
        while ((len = fr.read(cbuffer)) != -1){
            //遍历数组
            for (int i = 0; i < len; i++) {     //这里注意循环条件是len，不能是数组长度
                System.out.print(cbuffer[i]);
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //4.流资源的关闭操作（必须要关闭，否则会内存泄漏）
        try {
            if (fr != null)
                fr.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}

```

#### 3.2.2 FileWriter

`java.io.FileWriter` 类用于写入字符到文件，构造时使用系统默认的字符编码和默认字节缓冲区

- `FileWriter(File file)`：创建一个新的FileWriter，给定要读写入的File对象
- `FileWriter(String fileName)`：创建一个新的FileWriter，给定要写入的文件的名称
- `FileWriter(File file, boolean append)`：创建一个新的FileWriter，指明是否在现有文件末尾追加内容

举例：

```java
/**
 * 需求：将内存中的数据写出到指定的文件中
 */
@Test
public void test4(){
    FileWriter fw = null;
    try {
        //1.创建File两类的对象，指明要写出的文件的名称
        File file = new File("twins.txt");

        //2.创建输出流
        fw = new FileWriter(file);     //这种情况下，若原文件存在，则覆盖原文件
        //            fw = new FileWriter(file, false);   //不追加，若原文件存在，则覆盖原文件
        //            fw = new FileWriter(file, true);  //直接在原文件后追加

        //3.写出的具体过程
        //输出的方法：write(String str) / write(char[] cdata)
        fw.write("Twins forever!\n");
        fw.write("Twins 24th Anniversary happy!\n");
        fw.write("Sa happy birthday!");

        System.out.println("输出成功");

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //4.关闭资源
        try {
            if (fw != null)
                fw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 需求：复制一份hello.txt文件，命名为hello_copy.txt
 */
@Test
public void test5(){
    File src = new File("hello.txt");
    File dest = new File("hello_copy.txt");

    FileReader fr = null;
    FileWriter fw = null;
    try {
        fr = new FileReader(src);
        fw = new FileWriter(dest);

        int len;
        char[] cbuffer = new char[5];
        while ((len = fr.read(cbuffer)) != -1){
            //write(char[] cbuffer, int fromIndex, int len)
            fw.write(cbuffer, 0, len);
        }
        System.out.println("成功");
    }catch (IOException e){
        e.printStackTrace();
    } finally {
        //注意两个要并列写，不能放一起，因为放一起的话，第一个出现异常后，第二个不会关闭
        try {
            if (fw != null)
                fw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if (fr != null) {
                fr.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

## 4. 节点流之二：InputStream/OutputStream

### 4.1 InputStream和OutputStream

#### 4.1.1 字节输入流：InputStream

#### 4.1.2 字节输出流：OutputStream

### 4.2 FileInputStream与FileOutputStream

#### 4.2.1 FileInputStream

#### 4.2.2 FileOutputStream

### 4.3 练习



## 5. 处理流之一：缓冲流

### 5.1 构造器

### 5.2 效率测试

### 5.3 字符缓冲流特有方法

### 5.4 练习



## 6. 处理流之二：转换流

### 6.1 问题引入

**引入情况1：**

使用 `FileReader` 读取项目中的文本文件。由于IDEA设置中针对项目设置了UTF-8编码，当读取Windows系统中创建的文本文件时，如果Windows系统默认的是GBK编码，则读入内存中会出现乱码。

```java
public class Problem{
    public static void main(String[] args) throws IOException {
        FileReader fr = new FileReader("File_GBK.txt");
        int data;
        while ((data = fr.read()) != -1){
            System.out.print((char)data);
        }
        fr.close();
    }
}
输出结果：������
```

那么如何读取GBK编码的文件呢？

**引入情况2：**

针对文本文件，现在使用一个字节流进行数据的读入，希望将数据显示在控制台上。此时针对包含中文的文本数据，可能会出现乱码。

### 6.2 转换流的理解

**作用：转换流是字节与字符间的桥梁**

 ![【Java】缓冲流、转换流、序列化流-云社区-华为云](https://img-blog.csdnimg.cn/dbe48da0fedc460baeb5a5e04ae98564.png) 

具体来说：

 ![【Java】—— File类与IO流：处理流之转换流-CSDN博客](https://i-blog.csdnimg.cn/direct/0e799800717c48e98867e2b99324ebe9.png) 

### 6.3 InputStreamReader 与 OutputStreamWriter

- **InputStreamReader**
  - 转换流 `java.io.InputStreamReader` ，是Reader的的子类，是从字节流到字符流的桥梁。它读取字节，并使用指定的字符集将其解码为字符。它的字符集可以由名称指定，也可以使用平台的默认字符集
  - 构造器



### 6.4 字符编码和字符集

#### 6.4.1 编码与解码



#### 6.4.2 字符集

- **字符集Charset：**也叫编码表。是一个系统支持的所有字符的集合，包括各国文字、标点符号、图形符号、数字等
- 计算机要准确地存储和是被各种字符集符号，需要进行字符编码，一套字符集必然至少有一套字符编码。常见字符集有ASCII字符集、GBK字符集、Unicode字符集等。

可见，当指定了**编码**，它对应的**字符集**自然就指定了，所以**编码**才是我们最终要关心的

- **ASCII字符集**：
  - ASCII码（American Standard Code for Information Interchange，美国信息交换标准代码）：上个世纪60年代，美国指定列一套字符编码，对 `英语字符` 与二进制之间的关系，做了统一规定，这被称为ASCII码
  - ASCII码用于显示现代英语，主要包括控制字符（回车键、退格、换行符等）和可显示字符（英文带下斜字符、阿拉伯数字和西文符号）
  - 基本的ASCII码字符集，使用7位（bits）表示一个字符（最前面的1位统一规定为0），共 `128` 个字符。比如：空格“SPACE”是32（二进制00100000），大写的字母A是65（二进制01000001）.
  - 缺点：不能表示所有字符
- **ISO-8859-1字符集**
  - 拉丁码表，别名Latin-1，用于显示欧洲使用的语言，包括荷兰语、德语、意大利语、葡萄牙语等
  - ISO-8859-1使用单字节编码，兼容ASCII编码
- **GBxxx字符集**
  - GB就是国标的意思，是为了 `显示中文` 而设计的一套字符集
  - **GB2312：**简体中文码表。一个小于127的字符的意义与原来想通过，即向下兼容ASCII码。但大于127的字符连在一起时，就表示一个汉字，这样大约可以组合了包括 `7000多个简体汉字` ，此外数学符号、罗马希腊的字母、日文的假名们都编进去了，这就是常说的”全角“字符，而原来在127以下的那些符号就叫”半角“字符了
  - **GBK**：最常用的中文码表。是在GB2312标准基础上的扩展规范，使用了 `双字节` 编码方案，共收录了 `21003个` 汉字，完全兼容GB2312标准，同时支持 `繁体汉字` 以及日韩汉字等。
  - **GB18030：**最新的中文码表。收录汉字 `70244个` ，采用 `多字节` 编码，每个字可以由1个、2个或4个字节组成。支持中国国内少数民族的文字，同时支持繁体汉字以及日韩汉字等
- **Unicode字符集**
  - Unicode编码为表达 `任意语言的任意字符` 而设计，也称为统一码、标准万国码。Unicode将世界上所有的文字用 `2个字节` 统一进行编码，为每个字符设定唯一的二进制编码，以满足跨语言、跨平台进行文本处理的要求
  - Unicode的缺点：这里有三个问题：
    - 第一，英文字母只用一个字节表示就够了，如果使用更多的字节存储是 `极大的浪费`
    - 第二，如何才能 `区别Unicode和ASCII` ？计算机怎么知道两个字节表示一个符号，而不是分别表示两个符号呢
    - 第三，如果和GBK等双字节编码方式一样，最高位是1或0表示两个字节和一个字节，就缺少了很多值无法用于表示字符，`不够白哦是所有字符`
  - Unicode在很长一段时间内无法将进行推广，知道互联网的出现，为接近Unicode如何在网络上传输的问题，于是面向传输的众多UTF（UCS Transfer Format）标准出现。具体来说，有三种编码方案，UTF-8、UTF-16和UTF-32
- **UTF-8字符集**
  - Unicode是字符集，UTF-8、UTF-16、UTF-32是三种 `将数字转换到程序数据` 的编码方案。顾名思义，UTF-8就是每次8个位传输数据，而UTF-16就是每次16个位。其中，UTF-8是在互联网上 `使用最广` 的一种Unicode实现方式
  - 互联网工程工作小组（IETF）要求所有互联网协议都必须支持UTF-8编码。所以，我们开发Web应用，也要使用UTF-8编码。UTF-8是一种 `变长的编码方式` 。它使用1-4个字节为每个字符编码，编码规则：
    1. 128个US-ASCII字符，只需1个字节编码
    2. 拉丁文等字符，需要2个字节编码
    3. 大部分常用字（含中文），使用3个字节编码
    4. 其他极少使用的Unicode辅助字符，使用4个字节编码
- 举例

Unicode符号范围|UTF-8编码方式

```
(十六进制)			  |  (二进制)
---------------------|--------------------
0000 0000-0000 007F  | 0xxxxxxx (兼容原来的ASCII)
0000 0000-0000 07FF  | 110xxxxx 10xxxxxx
0000 0000-0000 FFFF  | 1110xxxx 10xxxxxx 10xxxxxx
0001 0000-0010 FFFF  | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
```



## 7. 处理流之三/四：数据流、对象流

### 7.1 数据流与对象流说明

如果需要将内存中定义的变量（包括基本数据类型和引用数据类型）保存在文件中，那怎么办呢？

```java
int age = 300;
char gender = '男';
int energe = 5000;
double price = 75.5;
boolean relive = true;

String name = "巫师";
Student stu = new Student("张三", 23, 89);
```

Java提供了数据流和对象流来处理这些类型的数据：

- **数据流：DataOutputStream、DataInputStream**
  - DataOutputStream：允许应用程序将基本数据类型、String类型的变量写入输入流中
  - DataInputStream：允许应用程序以与机器无关的方式从底层输入流中读取基本数据类型、String类型的变量
- 对象流DataInputStream中的方法：

```java
byte readByte() 				short readShort()
int readInt()					long readLong()
float readFloat()				double readDouble()
char readChar()					boolean readBoolean()
String readUTF()				void readFully(byte[] b)
```

- 对象流DataInputStream中的方法：将上述的方法的read改为相应的write即可
- 数据流的弊端：只支持Java基本数据类型和字符串的读写，而不支持其他Java对象的类型。而ObjectOutputStream和ObjectInputStream既支持Java基本数据类型的数据读写，又支持Java对象的读写，所以重点介绍对象流ObjectOutputStream和ObjectInputStream
- **对象流：ObjectOutputStream、ObjectInputStream**
  - ObjectOutputStream：将Java基本数据类型和对象写入字节输出流中。通过在流中使用文件可以实现Java各种基本数据类型的数据以及对象的持久存储
  - ObjectInputStream：ObjectInputStream对以前使用的 ObjectOutputStream 写出的基本数据类型和对象进行读入操作，保存在内存中

> 说明：对象流的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来

### 7.2 对象流API

**ObjectOutputStream中的构造器：**

`public ObjectOutputStream(OutputStream out)`：创建一个指定的ObjectOutputStream。

```java
FileOutputStream fos = new FileOutputStream("game.dat");
ObjectOutputStream oos = new ObjectOutputStream(fos);
```

**ObjectOutputStream中的方法：**

- public void writeBoolean(boolean val)：写出一个boolean值
- public void writeByte(int val)：写出一个8位字节
- public void writeShort(int val)：写出一个16位的short值
- public void writeChar(int val)：写出一个16位的char值
- public void writeInt(int val)：写出一个32位的int值
- public void writeLong(long val)：写出一个64位的long值
- public void writeFloat(float val)：写出一个32位的float值
- public void writeDouble(double val)：写出一个64位的double值
- public void writeUTF(String str)：将表示长度信息的两个字节写入数据流，后跟字符串s中每个字符的UTF-8修改版表示形式。根据字符的值，将字符串s中每个字符转换为一个字节、两个字节或三个字节的数组。注意，将String作为基本数据写入流中与将它作为Object写入流中明显不同，如果s为null，则抛出NullPointerException
- `public void writeObject(Object obj)`：写出一个obj对象
- public void close()：关闭此输出流并释放与此流相关联的任何系统资源

**ObjectInputStream中的方法：**

- public void readBoolean(boolean val)：读取一个boolean值
- public void readByte(int val)：读取一个8位字节
- public void readShort(int val)：读取一个16位的short值
- public void readwriteChar(int val)：读取一个16位的char值
- public void readInt(int val)：读取一个32位的int值
- public void readLong(long val)：读取一个64位的long值
- public voidreadFloat(float val)：读取一个32位的float值
- public void readDouble(double val)：读取一个64位的double值
- public void readUTF(String str)：将表示长度信息的两个字节写入数据流，后跟字符串s中每个字符的UTF-8修改版表示形式。根据字符的值，将字符串s中每个字符转换为一个字节、两个字节或三个字节的数组。注意，将String作为基本数据写入流中与将它作为Object写入流中明显不同，如果s为null，则抛出NullPointerException
- `public void writeObject(Object obj)`：写出一个obj对象
- public void close()：关闭此输出流并释放与此流相关联的任何系统资源

### 7.3 认识对象序列化机制

**1、何为对象序列化机制？**

`对象序列化机制` 允许把内存中的Java对象转换成与平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这个二进制流传递到另一个网络节点。当其他程序获得了这种二进制流，就可以复原成原来的Java对象。

- 序列化过程：用一个字节序列可以表示一个对象，该字节序列包含该 `对象的类型` 和 `对象中存储的属性` 等信息。字节序列写出到文件之后，相当于文件中 `持久保存` 了一个对象的信息
- 反序列化过程：该字节序列还可以从文件中读取回来，重构对象，对它进行 `反序列化` 。`对象的数据`、`对象的类型` 和`对象中存储的数据` 信息，都可以用来在内存中创建对象。

 ![Java并发编程：Java 序列化的工作机制 - 知乎](https://picx.zhimg.com/v2-b4e7789fa3942814ea80de8c17e3584f_720w.jpg?source=172ae18b) 

**2、序列化机制的重要性**

序列化是RMI（Remote Method Invoke 远程方法调用）过程的参数和返回值都必须实现的机制，而RMI是JavaEE的基础。因此序列化机制是JavaEE平台的基础

序列化的好处，在于可将任何实现了Serializable接口的对象转化为**字节数据**，使其在保存和传输时可被还原

**3、实现原理**

- 序列化：用ObjectOutputStream类保存基本数据类型或对象的机制，方法为：
  - `public final void writeObject(Object obj)`：将指定的对象写出
- 反序列化：用ObjectInputStream类读取基本数据类型或对象的机制，方法为：
  - `public final Object readObject()`：读取一个对象

 ![【JavaSE】Java序列化详解_java序列化过程-CSDN博客](https://img-blog.csdnimg.cn/img_convert/b6190ce847a46da397c38bd70c845e3b.png#pic_center) 

### 7.4 如何实现序列化机制

如果需要让某个对象支持序列化机制，则必须让对象所属的类及其属性是可序列化的，为了让某个类是可序列化的，该类必须实现 `Serializable` 接口。 `Serializable` 是一个 `标记接口` ，不实现此接口的类将不回使任何状态序列化或反序列化，会抛出 `NotSerializableException`

- 如果对象的某个属性也是引用数据类型，那么如果该属性也要序列化的化，也要实现 `Serializable` 接口
- 该类的所有属性必须是可序列化的，如果有一个属性不需要可序列化的，则该属性必须注明是瞬态的，使用 `transient` 关键词修饰
- `静态（static）变量` 的值不会序列化，因为静态变量的值不属于某个对象

### 7.5 反序列化失败问题

接口的类都应该有一个表示序列化版本标识符的静态变量：

```java
static final long serialVersionUID = 54359490235L;
```

- serialVersionUIO用来表明类的不同版本间的兼容性。简单来说，Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本的一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体类的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常（InvalidClassException）
- 如果类没有显示定义这个静态变量，它的值是Java运行时环境根据类的内部细节 `自动生成` 的。若类的实例变量做了修改，serivalVersionUID `可能发生变化` 。因此，建议显式声明
- 如果声明了serivalVersionUID，即使在序列化完成之后修改了类导致类重新编译，原来的数据也能正常反序列化，只是新增的字段值是默认值而已



## 8. 其他流的使用

### 8.1 标准输入、输出流

- System.in和System.out分别代表了系统标准的输入和输出设备
- 默认输入设备是：键盘，输出设备是：显示器
- System.in的类型是InputStream
- System.out的类型是PrintStream，其是OutputStream的子类FilterOutputStream的子类
- 重定向：通过System类的setIn，setOut方法对默认设备进行修改
  - public static void setIn(InputStream in)
  - public static void setOut(PrintStream out)

**举例：**

从键盘输入字符串，要求将读取到的整行字符串转成大写输出。然后继续进行输入操作，直至当输入”e“或者”exit“时，退出程序

```java
@Test
    public void test1(){
        System.out.println("请输入信息(退出输入e或exit)");
        //把标准输入流这个字节流包装成字符流，再包装成缓冲流
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String s = null;
        try {
            while ((s = br.readLine()) != null){  //读取用户输入的一行数据  ---> 阻塞程序
                if ("e".equalsIgnoreCase(s) || "exit".equalsIgnoreCase(s)){
                    System.out.println("安全退出！！");
                    break;
                }
                // 将读取到的整行字符串转成大写输出
                System.out.println("--->:" + s.toUpperCase());
                System.out.println("继续输入信息");
            }
        } catch (IOException e){
            e.printStackTrace();
        } finally {
            try {
                if (br != null){
                    br.close();  // 关闭过滤流时，会自动关闭它包装的底层节点流
                }
            } catch (IOException e){
                e.printStackTrace();
            }
        }
    }
```

**练习：**

Create a program named MyInput.java: Contain the methods for reading int, double, float, boolean, short, byte and String values from the keyboard.

```java
public class MyInput {
    public static String readString() {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String str = "";
        
        try {
            str = br.readLine();
        } catch (IOException e){
            e.printStackTrace();
        }
        
        return str;
    }
    
    public static int readInt(){
        return Integer.parseInt(readString());
    }
    
    public static double readDouble(){
        return Double.parseDouble(readString());
    }
    
    public static byte readByte(){
        return Byte.parseByte(readString());
    }
    
    public static Short readShort(){
        return Short.parseShort(readString());
    }
    
    public static Float readFloat(){
        return Float.parseFloat(readString());
    }
}
```



### 8.2 打印流

- 实现将基本数据类型的数据格式转化为字符串输出

- 打印流：`PrintStream` 和 `PrintWriter`

  - 提高功率一系列重载的print()和println()方法，用于多种数据类型的输出

  ![1732802116930](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1732802116930.png)

  ![1732802138852](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1732802138852.png)

举例：可以自定义一个日志工具

```java
public class Logger {
    /*
    记录日志的方法
     */
    
    public static void log(String msg){
        try {
            //指向一个日志文件
            PrintStream ps = new PrintStream(new FileOutputStream("log.txt", true));
            //改变输出方向
            System.setOut(ps);
            //日期当前时间
            Date nowTime = new Date();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss SSS");
            String strTime = sdf.format(nowTime);

            System.out.println(strTime + ": " + msg);
        } catch (FileNotFoundException e){
            e.printStackTrace();
        }
    }
}
```

```java
public class LogTest {
    public static void main(String[] args) {
        //测试工具类是否好用
        Logger.log("调用了System类的gc()方法，建议启动垃圾回收");
        Logger.log("调用了TeamView的addMember()方法");
        Logger.log("用户尝试进行登录，验证失败");
    }
}
```

### 8.3 Scanner类





## 9. apache-common包的使用

### 9.1 介绍

IO技术开发中，代码量很大，而且代码的重复率较高，为此Apache基金会，开发了IO技术的工具类`commonsIO`，大大简化了IO开发

Apache基金会属于第三方，（Oracle公司第一方，我们自己第二方，其他都是第三方）我们要使用第三方开发好的工具，需要添加jar包

### 9.2 导包及举例

- 在导入commons-io-2.5.jar包之后，内部的API都可以调用
- IOUtils类的使用