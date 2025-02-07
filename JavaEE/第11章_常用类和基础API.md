# 第11章_常用类和基础API



## 1. 字符串相关类之不可变字符序列：String

### 1.1 String的特性

- `java.lang.String`类代表字符串。Java程序中所有的字符串文字（例如`"hello"`）都可以看作是实现此类的实例

- 字符串是常量，用双引号引起来表示。它们的值在创建之后不能更改

- 字符串String类型本身是final声明的，意味着我们不能继承String

- String对象的字符内容是存储在一个字符数组value[]中的。`"abc"`等效于`char[] data = {'a', 'b', 'c'}`

  ![1730543095138](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730543095138.png)

  ```java
  //jdk8中的String源码
  public final class String
      implements java.io.Serializable, Comparable<String>, CharSequence {
      /** The value is used for character storage. */
      private final char value[];
  
      /** Cache the hash code for the string */
      private int hash; // Default to 0
  }
  ```

  - private意味着外面无法直接获取字符数组，而且String没有提供value的get和set方法

  - final意味着字符数组的引用不可改变，而且String也没有提供方法来修改value数组的某个元素值

  - 因此字符串的字符数组内容也是不可变的，即String代表着不可变的字符序列。即，一旦对字符串进行修改，就会产生新对象。

  - jdk9只有，底层使用byte[]数组

    ```java
    public final class String
        implements java.io.Serializable, Comparable<String>, CharSequence,
                   Constable, ConstantDesc {
    
        @Stable
        private final byte[] value;
    }
    
    //官方说明：...that most String objects contain only Latin-1 characters. Such characters require only one byte of storage, hence half of the space in the internal char arrays of such String objects is going unused.
    
    //细节：... The new String class will store characters encoded either as ISO-8859-1/Latin-1 (one byte per character), or as UTF-16 (two bytes per character), based upon the contendts of the String. The encoding flag will indicate which encoding is used.
    ```

- Java语言提供对字符串串联符号（"+"）以及其他对象转换为字符串的特殊支持（toString()方法）

### 1.2 String的内存结构

#### 1.2.1 概述

因为字符串对象设计为不可变，那么字符串有常量池来保存很多常量对象。

JDK6中，字符串常量池在方法区。JDK7开始，就移到堆空间，知道目前JDK23版本。

![1730545232830](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730545232830.png)

栈空间中没有GC（垃圾回收器），方法区和堆空间中都有GC，而堆是频繁进行GC的场所，方法区很少进行GC，主要是加载类的一些信息，为了及时回收这部分内存空间，就转移到了堆空间中。

![1730545125357](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730545125357.png)

![1730545180149](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730545180149.png)

![1730600603938](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730600603938.png)

![1730716316508](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730716316508.png)



#### 1.2.2 练习类型1：拼接

```java
@Test
    public void test3(){
        String s1 = "hello";
        String s2 = "hello";

        s2 += "world";

        System.out.println(s1);   //hello
        System.out.println(s2);   //helloworld
    }
```

![1730717022657](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730717022657.png)

- String连接符：+
  - 情况1：常量 + 常量：结果仍然存储在字符串常量池中。注：此时的常量可能是字面量，也可能是final修饰的常量
  - 情况2：常量 + 变量 或 变量 + 变量：都会通过new的方式创建一个新的字符串，返回堆空间中此字符串对象的地址
  - 情况3： 调用字符串的intern()：返回的是字符串常量池中字面量的地址

```java
/**
     * 测试String的连接符：+
     */
    @Test
    public void test3(){
        String s1 = "hello";
        String s2 = "world";
        
        String s3 = "helloworld";
        String s4 = "hell0" + "world";
        String s5 = s1 + "world";  //通过查看字节码文件发现调用了StringBuilder的toString方法 ---> new String()
        String s6 = "hello" + s2;
        String s7 = s1 + s2;

        System.out.println(s3 == s4);  //true
        System.out.println(s3 == s5);  //false
        System.out.println(s3 == s6);  //false
        System.out.println(s3 == s7);  //false
        System.out.println(s5 == s6);  //false
        System.out.println(s5 == s7);  //false
        //有变量参与的相当于是底层给我们new了一个对象
    }
```

- concat：不管是常量调用此方法，还是变量调用，同样不管参数是常量还是变量，总之调用完concat()方法都返回一个新new的对象。

```java
//concat的使用：不管是常量调用此方法，还是变量调用，同样不管参数是常量还是变量，总之调用完concat()方法都返回一个新new的对象。
@Test
    public void test5(){
        String s1 = "hello";
        String s2 = "world";

        String s3 = s1.concat(s2);
        String s4 = "hello".concat("world");
        String s5 = s1.concat("world");

        System.out.println(s3 == s4);  //false
        System.out.println(s3 == s5);  //false
        System.out.println(s4 == s5);  //false
    }
```



#### 1.2.3 练习类型2：new

- String s = new String("hello");在内存创建了几个对象？  

  两个。一个是堆空间中new的对象，另一个是在字符串常量池中生成的字面量

```java
//replace方法返回的是new String(..)
@Test
    public void test4(){
        String s1 = "hello";
        String s2 = "hello";

        String s3 = s2.replace('l', 'w');
        System.out.println(s1);   //hello
        System.out.println(s2);   //hello
        System.out.println(s3);   //hewwo
    }
```

![1730717675489](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730717675489.png)

```java
public class StringDemo1 {
    @Test
    public void test1(){
        String s1 = "hello";
        String s2 = "hello";

        String s3 = new String("hello");
        String s4 = new String("hello");

        System.out.println(s1 == s2);   //true
        System.out.println(s1 == s3);   //false
        System.out.println(s1 == s4);   //false
        System.out.println(s3 == s4);   //false

        System.out.println(s1.equals(s2));  //true,全是true
    }
    /**
     * String s = new String("hello");的内存解析？或
     * String s = new String("hello");在内存创建了几个对象？2个  
     */
}
```

![1730718574626](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730718574626.png)

```java
@Test
    public void test2(){
        Person p1 = new Person();
        Person p2 = new Person();
        p1.name = "Tom";
        p2.name = "Tom";

        p1.name = "Jerry";
        System.out.println(p2.name);   //Tom
    }
class Person{
    String name;

}
```

![1730719718512](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730719718512.png)

#### 1.2.4 练习类型3：intern()

```java
/**
     * 测试String的连接符：+
     */
    @Test
    public void test3(){
        String s1 = "hello";
        String s2 = "world";
        
        String s3 = "helloworld";
        String s4 = "hell0" + "world";
        String s5 = s1 + "world";  //通过查看字节码文件发现调用了StringBuilder的toString方法 ---> new String()
        
        String s8 = s5.intern();   //intern():返回的是字符串常量池中字面量的地址
        System.out.println(s3 == s8);  //true
    }
```

```java
@Test
    public void test4(){
        final String s1 = "hello";  //加上final之后就相当于变成常量
        final String s2 = "world";

        String s3 = "helloworld";
        String s4 = "hell0" + "world";
        String s5 = s1 + "world";  //通过查看字节码文件发现调用了StringBuilder的toString方法 ---> new String()
        String s6 = "hello" + s2;
        String s7 = s1 + s2;

        System.out.println(s3 == s5);  //true
        System.out.println(s3 == s6);  //true
        System.out.println(s3 == s7);  //true

    }
```



### 1.3 String的常用API-1

#### 1.3.1 构造器

- `public String()`：初始化新创建的String对象，以使其表示控制符序列
- `public String(String original)`：初始化一个新创建的`String`对象，使其表示一个与参数相同的字符序列，换句话说，新创建的字符串是该参数字符串的副本
- `public String(char[] value)`：通过当前参数中的字符数组来构造新的String
- `public String(char[] value, int offset, int count)`：通过字符数组的一部分来构建新的String
- `public String(byte[] bytes)`：通过使用平台的**默认字符集**解码当前参数中的字节数组来构造新的String
- `public String(byte[] bytes, String charsetName)`：通过使用指定的字符集解码当前参数中的字节数组来构造新的String

举例：

```java
//字面量定义方式：字符串常量对象
String str = "hello";

//构造器定义方式：无参构造
String str1 = new String();

//构造器定义方式：创建“hello”字符串常量的副本
String str2 = new String("hello");

//构造器定义方式：通过字符数组构造
char[] chars = new char[]{'a', 'b', 'c', 'd', 'e'};
String str3 = new String(chars);
String str4 = new String(chars, 0, 3);    //abc
    
//构造器定义方式：通过字符数组构造
byte[] bytes = {97, 98, 99};
String str5 = new String(bytes);
String str6 = new String(bytes, "gbk");
```

```java
public static void main(String[] args){
    char[] data = {'h', 'e', 'l', 'l', 'o', 'j', 'a', 'v', 'a'};
    String s1 = String.copyValueOf(data);
    String s2 = String.copyValueOf(data, 0, 5);
    int num = 23456;
    String s3 = String.valueOf(num);
    
    System.out.println(s1);
    System.out.println(s2);
    System.out.println(s3);
}
```

#### 1.3.2 String与其他结构间的转换

**字符串 --> 基本数据类型、包装类**

- Integer包装类的public static int parseInt(String s)：可以将由“数字”字符组成的字符串转换为整型
- 类似地，使用java.lang包中的Byte、Short、Long、Float、Double类调用类似的类方法可以将由“数字”字符组成的字符串转化为相应的基本数据类型。

**基本数据类型、包装类 --> 字符串：**

- 调用String类的public String valueOf(int n)可将int型转换为字符串
- 相应的valueOf(byte b)、valueOf(long l)、valueOf(float f)、valueOf(double d)、valueOf(boolean b)可由参数的相应类型转换为字符串

**字符数组 --> 字符串：**

- String类的构造器：String(char[]) 和 String(char[], int offset, int length) 分别用字符数组中的全部字符和部分字符创建字符串对象。

**字符串 --> 字符数组：**

- public char[] toCharArray()：将字符串中的全部字符存放在一个字符数组中
- public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)：将指定索引范围内的字符串存放到数组中

**字符串 --> 字节数组：**

- public byte[] getBytes()：使用平台的默认字符集将此String编码为byte序列，并将结果存储到一个新的byte数组中。
- public byte[] getBytes(String charsetName)：使用指定的字符集将此String编码到byte序列，并将结果存储到新的byte数组

**字节数组 --> 字符串：（解码）**

- String(byte[])：通过使用平台的默认字符集解码指定的byte数组，构造一个新的String。
- String(byte[], int offset, int length)：用指定的字节数组的一部分，即从数组起始位置offset开始取length个字节构造一个字符串对象。
- String(byte[], String charsetName)或new String(byte[], int, int, String charsetName)：解码，按照指定的编码方式进行解码

```java
public class StringMethodTest {
    /**
     * String与常见的其他结构之间的转换
     *
     * 1、String与基本数据类型、包装类之间的转换（复习）
     *
     * 2、String与char[]之间的转换
     *
     * 3、String与byte[]之间的转换（难度）
     */
    @Test
    public void test2(){
        int num = 12;
        //基本String与char[]之间的转换数据类型 ---> String
        //方式1：
        String s1 = num + "";
        //方式2：
        String s2 = String.valueOf(num);

        //String ---> 基本数据类型
        String s3 = "123";
        int i1 = Integer.parseInt(s3);
    }

    //String与char[]之间的转换
    @Test
    public void test3(){
        String str = "hello";
        //String --> char[]:调用String的toCharArray()
        char[] arr = str.toCharArray();
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }

        //char[] --> String：调用String的构造器
        String str1 = new String(arr);
        System.out.println(str1);
    }

    //String与byte[]之间的转换（难度）

    /**
     * 在UTF-8字符集中，一个汉字占用3个字节，一个字母占用1个字节
     * 在gbk字符集中(专门给汉字使用的)，一个汉字占用2个字节，一个字母占用1个字节
     *
     * UTF-8或gbk都向下兼容了ASCII码
     *
     * 编码与解码：
     *   编码：String ---> 字节或字节数组
     *   解码：字节或字节数组 ---> String
     * 要求：解码时使用的字符集必须与编码时使用的字符集一致，不一致就会导致乱码
     */
    @Test
    public void test4() throws UnsupportedEncodingException {
        String str = new String("abc中国");
        //String --> byte[]：调用String的getbytes()方法
        byte[] arr = str.getBytes();    //使用默认的字符集
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }

        //getBytes(String charsetName):使用指定的字符集
        byte[] arr1 = str.getBytes("gbk");
        for (int i = 0; i < arr1.length; i++) {
            System.out.println(arr1[i]);
        }

        //bytes[] ---> String
        String str1 = new String(arr);  //使用默认字符集：UTF-8
        System.out.println(str1);

        String str2 = new String(arr, "utf-8");
        System.out.println(str2);

//        String str3 = new String(arr, "gbk");
//        System.out.println(str3);
        String str4 = new String(arr1, "gbk");
        System.out.println(str4);


    }
}
```



### 1.4 String的常用API-2

`String`类包括的方法可用于检查序列的单个字符、比较字符串、搜索字符串、提取子字符串、创建字符串副本并将所有字符全部转换为大写或小写。

#### 1.4.1 系列1：常用方法

（1） boolean isEmpty()：字符串是否为空

（2） int length()：返回字符串的长度

（3） String concat(xx)：拼接

（4） boolean equals(Object obj)：比较字符串是否相等，区分大小写

（5） boolean equalsIgnoreCase(Object obj)：比较字符串是否相等，不区分大小写

（6） int compareTo(String other)：比较字符串大小，区分大小写，按照Unicode编码值比较大小

（7） int compareToIgnoreCase(String other)：比较字符串大小，不区分大小写

（8） String toLowerCase()：将字符串中大写字母转为小写

（9） String toUpperCase()：将字符串中小写字母转为大写

（10） String trim()：去掉字符串前后空白符

（11） public String intern()：返回字符串常量池中字面量的地址

```java
public class StringMethodTest1 {
    @Test
    public void test1(){
        String s1 = "";
        String s2 = new String("");
        String s3 = new String();

        System.out.println(s1.isEmpty());
        System.out.println(s2.isEmpty());
        System.out.println(s3.isEmpty());

//        String s4 = null;
//        System.out.println(s4.isEmpty());//空指针异常

        String s5 = "hello";
        System.out.println(s5.length());
    }

    @Test
    public void test2(){
        String s1 = "hello";
        String s2 = "Hello";
        System.out.println(s1.equals(s2));
        System.out.println(s1.equalsIgnoreCase(s2));

        String s3 = "abcd";
        String s4 = "adef";
        System.out.println(s3.compareTo(s4));

        String s5 = "abcd";
        String s6 = "aBcd";
        System.out.println(s5.compareTo(s6));
        System.out.println(s5.compareToIgnoreCase(s6));

        String s9 = " he  llo   ";
        System.out.println("***" + s9.trim() + "****");
    }

}
```



#### 1.4.2 系列2：查找

（12） boolean contains(xx)：是否包含xx

（13） int indexOf(XX)：从前往后找当前字符串中xx，即如果有返回第一次出现的下标，要是没有返回-1

（14） int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始

（15） int lastIndexOf(xx)：从后往前找当前字符串中xx，即如果有返回最后一次出现的下标，要是没有返回-1

（16） int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索。

```java
@Test
    public void test3(){
        String s1 = "教育尚硅谷教育";
        System.out.println(s1.contains("硅谷"));//true

        System.out.println(s1.indexOf("教育"));//0
        System.out.println(s1.indexOf("教育", 1));//5

        System.out.println(s1.lastIndexOf("教育"));//5
        System.out.println(s1.lastIndexOf("教育", 4));//0
    }
```



#### 1.4.3 系列3：字符串截取

（17） String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串

（18） String substring(int beginIndex, int endIndex)：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex（不包含）的一个子字符串

```java
@Test
    public void test4(){
        String s1 = "教育尚硅谷教育";
        System.out.println(s1.substring(2));
        System.out.println(s1.substring(2, 5));
    }
```



#### 1.4.4 系列4：和字符/字符数组相关

（19） char charAt(int index)：返回[index]位置的字符

（20） char[] toCharArray()：将此字符串转换为一个新的字符数组返回

（21） static String valueOf(char[] data)：返回指定数组中表示该字符序列的String

（22） static String valueOf(char[] data, int offset, int count)：返回指定数组中表示该字符序列的String

（23） staitc String copyValueOf(char[] data)：返回指定数组中表示该字符序列的String

（24） static String copyValueOf(char[], int offset, int count)：返回指定数组中表示该字符序列的String

```java
    @Test
    public void test5(){
        String s1 = "教育尚硅谷教育";
        System.out.println(s1.charAt(2));

        String s2 = String.valueOf(new char[]{'a', 'b', 'c'});
        String s3 = String.copyValueOf(new char[]{'a', 'b', 'c'});
        System.out.println(s2);
        System.out.println(s3);
        System.out.println(s2 == s3); //false
```



#### 1.4.5 系列5：开头与结尾

（25） boolean startsWith(xx)：测试此字符串是否以指定的前缀开始

（26） boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始

（27） boolean endsWith(xx)：测试此字符串是否以指定的后缀结束

```java
    @Test
    public void test5(){
        String s1 = "教育尚硅谷教育";
       
        System.out.println(s1.startsWith("教育"));  //true
        System.out.println(s1.startsWith("教育", 2)); //false
        System.out.println(s1.startsWith("教育", 5)); //true
        System.out.println(s1.endsWith("教育")); //true

    }
```

#### 1.4.6 系列6：替换

（28） String replace(char oldChar, char newChar)：返回一个新的字符串，它是通过用newChar替换词字符串出现的所有oldChar得到的，不支持正则

（29） String replace(CharSequence target, CharSequence replacement)：使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串

（30） String replaceAll(String regex, String replacement)：使用给定的replacement替换此字符串所有匹配给定的正则表达式的子字符串

（31） String replaceFirst(String regex, String replacement)：使用给定的replacement替换此字符串匹配给定的正则表达式的第一个子字符串

```java
@Test
    public void test6(){
        String s1 = "hello";
        String s2 = s1.replace('l', 'w');
        System.out.println(s1);  //hello
        System.out.println(s2);  //hewwo

        System.out.println(s1.replace("ll", "wwww"));//hewwwwo
        
    }
```

### 1.5 常见算法题目

**题目1：**模拟一个trim方法，去除字符串两端的空格

```java

```

**题目2：**将一个字符串进行反转。将字符串中指定部分进行反转

```java

```

**题目3：**获取一个字符串在另一个字符串中出现的次数

```java

```

**题目4：**获取两个字符串中最大相同子串

```java

```

**题目5：**对字符串中字符进行自然顺序排序

```java

```



## 2. 字符串相关类之可变字符序列：StringBuffer、StringBuilder

### 2.1 StringBuffer与StringBuilder的理解

- StringBuilder和StringBuffer非常类似，均代表可变的字符序列，而且提供相关功能的方法也意义
- 区分String、StringBuffer、StringBuilder
  - String：不可变的字符序列；底层使用char[]（jdk8及之前），底层使用bytes[]（jdk9及之后）
  - StringBuffer：可变的字符序列；JDK1.0声明，线程安全的，效率低；底层使用char[]（jdk8及之前），底层使用bytes[]（jdk9及之后）
  - StringBuilder：可变的字符序列；JDK5.0声明，线程不安全的，效率高；底层使用char[]（jdk8及之前），底层使用bytes[]（jdk9及之后）

### 2.2 StringBuilder、StringBuffer的API

StringBuilder、StringBuffer的API是完全一致的，并且很多方法与String相同。

**1、常用API**

（1）StringBuffer append(xx)：提供了很多的append()方法，用于进行字符串追加的方式拼接

（2）StringBuffer delete(int start, int end)：删除[start, end)之间字符

（3）StringBuffer deleteCharAt(int index)：删除[index]位置字符

（4）StringBuffer replace(int start, int end, String str)：替换[start, end)范围的字符序列为str

（5）void setCharAt(int index, char c)：替换[index]位置字符

（6）char charAt(int index)：查找指定index位置上的字符

（7）StringBuffer insert(int index, xx)：在[index]位置上插入xx

（8）int length()：返回存储的字符数据的长度

（9）StringBuffer reverse()：反转

> - 当append和insert时，如果原来的value数组长度不构，可扩容
>
> - 如上(1)(2)(3)(4)(9)这些方法支持`方法链操作`。原理：
>
>   ![1730790580727](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730790580727.png)

**2、其他API**

（1） int indexOf(String str)：在当前字符序列中查找str第一次出现的下标

（2） int indexOf(String str, int fromIndex)：在当前字符序列[fromIndex,最后]中查询str第一次出现的下标

（3） int lastIndexOf(String str)：在当前字符序列中查找str最后一次出现的下标

（4） int lastIndexOf(String str, int fromIndex)：在当前字符序列[fromIndex,最后]中查询str最后一次出现的下标

（5） String substring(int start)：截取当前字符序列[start,最后]

（6） String substring(int beginIndex, int endIndex)：截取当前字符序列[start, end)

（7） String toString()：返回此序列中数据的字符串表示形式

（8） void setLength(int newLength)：设置当前字符序列长度

```java
@Test
    public void test3(){
        StringBuilder sBuilder = new StringBuilder("hello");
        sBuilder.setLength(2);    //
        System.out.println(sBuilder);
        System.out.println(sBuilder.length());

        System.out.println(sBuilder.append('c'));

        sBuilder.setLength(5);
        System.out.println(sBuilder);
        System.out.println(sBuilder.length());
        System.out.println(sBuilder.append('d'));
        
        sBuilder.setLength(10);
        System.out.println(sBuilder.charAt(7)==0);
    }
//结果：
he
2
hec
hec  
5
hec  d
true
```



### 2.3 效率测试

```java
/**
     * 测试String、StringBuffer、StringBuilder在操作数据方面的效率
     */
    @Test
    public void test4(){
        //初始设置
        long startTime = 0L;
        long endTime = 0L;
        String text = "";
        StringBuffer buffer = new StringBuffer("");
        StringBuilder builder = new StringBuilder("");

        //开始对比
        startTime = System.currentTimeMillis();
        for (int i = 0; i < 20000; i++) {
            buffer.append(String.valueOf(i));
        }
        endTime = System.currentTimeMillis();
        System.out.println("StringBuffer的执行时间：" + (endTime - startTime));

        startTime = System.currentTimeMillis();
        for (int i = 0; i < 20000; i++) {
            builder.append(String.valueOf(i));
        }
        endTime = System.currentTimeMillis();
        System.out.println("StringBuilder的执行时间：" + (endTime - startTime));

        startTime = System.currentTimeMillis();
        for (int i = 0; i < 20000; i++) {
            text += i;
        }
        endTime = System.currentTimeMillis();
        System.out.println("String的执行时间：" + (endTime - startTime));
    }
//结果：
StringBuffer的执行时间：2
StringBuilder的执行时间：1
String的执行时间：170
```



### 2.4 练习

```java
public class InterviewTest1 {
    public static void main(String[] args) {
        StringBuffer a = new StringBuffer("A");
        StringBuffer b = new StringBuffer("B");

        operate(a, b);  //a指向的区域执行完变为ABx，b指向的区域没有发生变化

        System.out.println(a + "," + b);
    }

    public static void operate(StringBuffer x, StringBuffer y){
        //传递后，x和y分别指向"A"和"B，
        x.append(y);  //x指向"AB"
        y = x;  //y指向"AB"，x、y指向相同的地址
        y.append("x");  //y指向的追加x
        //x、y指针释放
    }
}
//ABx,B
```

```java
public class InterviewTest2 {
    public static void stringReplace(String text){
        text = text.replace('j', 'i');
    }

    public static void bufferReplace(StringBuffer text){
        text.append("C");
        text = new StringBuffer("Hello");
        text.append("World!");
    }

    public static void main(String[] args) {
        String textString = new String("java");
        StringBuffer textBuffer = new StringBuffer("java");

        stringReplace(textString);
        bufferReplace(textBuffer);

        System.out.println(textString + textBuffer);
    }
}
//javajavaC
```

```java
public class InterviewTest3 {
    private static void change(String s, StringBuffer sb){
        s = "aaaa";
        sb.setLength(0);
        sb.append("aaaa");
    }

    public static void main(String[] args) {
        String s = "bbbb";
        StringBuffer sb = new StringBuffer("bbbb");
        change(s, sb);
        System.out.println(s+sb);
    }
//bbbbaaaa
```

```java
public class InterviewTest4 {
    public static void main(String[] args) {
        String str = null;
        StringBuffer sb = new StringBuffer();
        sb.append(str);

        System.out.println(sb.length());  //4

        System.out.println(sb);  //"null"

        StringBuffer sb1 = new StringBuffer(str);
        System.out.println(sb1);   //空指针异常
    }
}
```

![1730803649301](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730803649301.png)

![1730803663784](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730803663784.png)

## 3. JDK8之前：日期时间API

### 3.1 java.lang.System类的方法

- System类提供的public static long currentTimeMillis()：用来返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差。

  - 此方法适于计算时间差
  - 通常称为时间戳，long类型

- 计算世界时间的主要标准有：

  - UTC（Coordinated Universal Time）
  - GMT（Greenwich Mean Time）
  - CST（Central Standard Time）

  > 在国际无线电通信场合，为了统一起见，使用一个统一的时间，称为通用协调时（UTC）。UTC与格林尼治平均时（GMT）一样，都与英国伦敦的本地时相同。这里，UTC与GMT含义完全相同

### 3.2 java.util.Date

表示特定的瞬间，精确到毫秒。

- 构造器：

  - Date()：使用无参构造器创建的对象可以获取当前系统时间
  - Date(long 毫秒数)：把该毫秒数值换算成日期时间对象

- 常用方法：

  - getTime()：返回自1970年1月1日00:00:00 GMT以来，此对象表示的毫秒数
  - toString()：把此Date对象转换成以下形式的String：dow mon dd hh:mm:ss zzz yyyy 其中：dow是一周中的某一天(Sun, Mon, Tue, Wed, Thu, Fri, Sat)，zzz是时间标准
  - 其它很多方法都过时了。

- 举例：

  ```java
  public class DateTimeTest {
      /**
       * Date类的使用
       *
       * |--java.util.Date
       *  > 两个构造器的使用
       *  > 两个方法的使用：①tuString() ②long getTime()
       *      |--java.sql.Date：对应着数据库中的date类型(子类)
       */
      @Test
      public void test1(){
          Date date1 = new Date();  //创建一个基于当前系统时间的Date实例
          System.out.println(date1.toString());
  
          long milliTimes = date1.getTime();
          System.out.println("对应的毫秒数：" + milliTimes);
  
          Date date2 = new Date(milliTimes);  //创建一个基于指定时间戳的Date的实例
          System.out.println(date2.toString());
      }
  
      @Test
      public void test2(){
          java.sql.Date date1 = new java.sql.Date(1730807341999L);
          System.out.println(date1.toString());
          System.out.println(date1.getTime()); //也包含具体到毫秒级，只是toString只显示到日
      }
  }
  ```

### 3.3 java.text.SimpleDateFormat

- java.text.SimpleDateFormat类是一个不与语言环境有关的方式来格式化和解析日期的具体类

- 可以进行格式化：日期 ---> 文本

- 可以进行解析：文本 ---> 日期

- **构造器：**

  - SimpleDateFormat()：默认的模式和语言环境创建对象
  - public SimpleDateFormat(String pattern)：该构造方法可以用参数pattern指定的格式创建一个对象

- **格式化：**

  - public String format(Date date)：方法格式化时间对象date

- **解析：**

  - public Date parse(String source)：从给定的字符串的开始解析文本，以生成一个日期

  ![1730808733613](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730808733613.png)

  ```java
  @Test
      public void test4() throws ParseException {
          SimpleDateFormat sdf = new SimpleDateFormat("EEE, d MMM yyyy HH:mm:ss Z");
          sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
          //格式化：日期 ---> 字符串
          Date date1 = new Date();
          String strDate = sdf.format(date1);
          System.out.println(strDate); //2024-11-05 20:16:35
  
          //解析：字符串 ---> 日期
          Date date2 = sdf.parse("2024-11-05 20:16:35"); //要匹配指定的格式
          System.out.println(date2.toString());
      }
  ```

  

### 3.4 java.util.Calendar（日历）

- Date类的API大部分被废弃了，替换为Calendar。

- `Calendar`类是一个抽象类，主要用于完成日期字段之间相互操作的功能。

- 获取Calendar实例的方法

  - 使用`Calendar.getInstance()`方法

    ![1730868435659](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730868435659.png)

  - 调用它的子类GregorianCalendar（公历）的构造器。

    ![1730868589373](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\1730868589373.png)

- 一个Calendar的实例是系统时间的抽象表示，可以修改或获取YEAR、MONTH、DAY_OF_WEEK、HOUR_OF_DAY、MINUTE、SECOND等`日历字段`对应的时间值。

  - public int get(int field)：返回给定日历字段的值
  - public void set(int field, int value)：将给定的日历字段设置为指定的值
  - public void add(int field, int amount)：根据日历的规则，为给定的日历字段添加或者减去指定的时间量
  - public final Date getTime()：将Calendar转成Date对象
  - public final void setTime()：使用指定的Date对象重置Calendar的时间

- 常用字段

  

### 3.5 练习



## 4. JDK8：新的日期时间API

如果我们可以跟别人说：“我们在1502643933071见面，别晚了！”那么久再简单不过了。但是我们希望时间与昼夜和四季有关，于是事情就变复杂了。JDK1.0中包含了一个java.util.Date类，但是它的大多数方法已经再JDK1.1引入Calendar类之后被弃用了。而Calendar并不比Date好多少。它们面临的问题是：

- 可变性：像日期和时间这样的类应该是不可变的

- 偏移性：Date中的年份是从1900开始的的，而月份都是从0开始

- 格式化：格式化只对Date有用，而Calendar则不行

- 此外，它们也不是线程安全的；不能处理闰秒等。

  > 闰秒，是指为保持协调世界时接近于世界时时刻，由国际计量局统一规定再年底或年中（也可能在季末）对协调世界时增加或减少一秒的调整。由于地球自转的不均匀性和长期变慢性（主要由潮汐摩擦引起的），会使世界时（民用时）和原子时之间相差超过±0.9秒时，就把协调世界时向前拨1秒（负闰秒，最后一分钟为59秒）或向后拨1秒（正闰秒，最后一分钟为61秒）；闰秒一般加在公历年末或公历六月末。
  >
  > 目前，全球已经进行了27次闰秒，均为正闰秒。

总结：`对日期和时间进行操作一直是Java程序员最痛苦的地方之一`。

第三次引入的API是成功的，并且Java 8中引入的java time API已经纠正了过去的缺陷，将来很长一段时间内它都会为我们服务

Java 8以一个新的开始为Java创建优秀的API。新的日期时间API包含：

- `java.time` - 包含值对象的基础包
- `java.time.chrono` - 提供对不同日历系统的访问
- `java.time.format` - 格式化和解析时间和日期
- `java.time.temporal` - 包括底层框架和拓展特性
- `java.time.zone` - 包含时区支持的类

说明：新的java.time中包含了所有关于时钟（clock）、本地日期（LocalDate）、本地时间（LocalTime）、本地日期时间（LocalDateTime）、时区（ZonedDateTime）和持续时间（Duration）的类。

尽管有68个新的公开类型，但是大多数开发者智慧用到基础包和format包，大概占总书的三分之一。

### 4.1 本地日期时间：LocalDate、LocalTime、LocalDateTime

| **<font color="black">方法</font>**                          | **描述**                                                |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| `now()`/now(Zoneld zone)                                     | 静态方法，根据当前时间创建对象/指定时区的对象           |
| `of(xx,xx,xx,xx,xx,xxx)`                                     | 静态方法，根据指定日期/时间创建对象                     |
| getDayOfMonth()/getDayOfYear()                               | 获得月份天数(1-31)/获得年份天数(1-366)                  |
| getDayOfWeek()                                               | 获得星期几(返回一个DayOfWeek枚举类)                     |
| getMonth()                                                   | 获得月份，返回一个Month枚举类                           |
| getMonthValue()/getYear()                                    | 获得月份(1-12)/获得年份                                 |
| getHours()/getMinute()/getSecond()                           | 获得当前对象对应的小时/分钟/秒                          |
| withDayOfMonth()/withDayOfYear()/withMonth()/withYear()      | 将月份天数/年份天数/月份/年份修改为指定值并返回新的对象 |
| with(TemporalAdjuster t)                                     | 将当前日期时间设置为校对器指定的日期时间                |
| plusDays()/plusWeeks()/plusMonths()/plusYears()/plusHours()  | 向当前对象添加几天/几周/几个月/几年                     |
| minusDays()/minusWeeks()/minusMonths()/minusYears()/minusHours() | 向当前对象减去几天/几周/几个月/几年                     |
| plus(TemporalAmount t)/minus(TemporalAmount t)               | 添加或减少一个Duration或Period                          |
| isBefore()/isAfter()                                         | 比较两个LocalDate                                       |
| isLeapYear()                                                 | 判断是否是闰年(在LocalDate类中声明)                     |
| format(DateTImeFormatter t)                                  | 格式化本地日期、时间，返回一个字符串                    |
| parse(Charsequence text)                                     | 将指定格式的字符串解析为日期、时间                      |
|                                                              |                                                         |



### 4.2 瞬时：Instant

- Instant：时间线上的一个瞬时点。这可能被用来记录应用程序中的事件时间戳

  - 时间戳是指格林威治时间1970年1月1日00时00分00秒(北京时间1970年1月1日08时00分00秒)起至今的总秒数。

- `java.time.Instant`表示时间线上的一点，而不需要任何上下文信息，例如时区。概念上讲，`它只是简单地表示自1970年1月1日00时00分00秒（UTC）开始的秒数`

  | **方法**                        | **描述**                                                     |
  | ------------------------------- | ------------------------------------------------------------ |
  | `now()`                         | 静态方法，返回默认UTC时区的Instant类的对象                   |
  | `ofEpochMilli(long epochMilli)` | 静态方法，返回在1970-01-01 00:00:00基础上加上指定毫秒数之后Instant类的对象 |
  | atOffset(ZoneOffset offset)     | 结合即时的偏移来创建一个OffsetDateTime                       |
  | `toEpochMilli()`                | 返回1970-01-01 00:00:00到当前时间的毫秒数，即为时间戳        |

  > 东八区与UTC的时差均为+8，也就是UTC+8.
  >
  > instant.atOffset(ZoneOffset.ofHours(8))

```java
/**
     * JDK8的api：Instant
     */

    @Test
    public void test4(){
        //now():
        Instant instant = Instant.now();
        System.out.println(instant);  //2024-11-06T06:34:19.567121700Z得到格林尼治时间
        //了解
        OffsetDateTime instant1 = instant.atOffset(ZoneOffset.ofHours(8));
        System.out.println(instant1);

        Instant instant2 = Instant.ofEpochMilli(24123123312L);
        System.out.println(instant2);  //1970-10-07T04:52:03.312Z

        long milliTime = instant.toEpochMilli();
        System.out.println(milliTime);
        System.out.println(new Date().getTime());
    }
```



### 4.3 日期时间格式化：

该类提供了三种格式化方法：

- （了解）预定义的标准格式。如：ISO_LOCAL_DATE_TIME、ISO_LOCAL_DATE、ISO_LOCAL_TIME

- （了解）本地化相关的格式。如：ofLocalizedDate(FormatStyle.LONG)

  ```java
  //本地化相关的格式。如：ofLocalizedDateTime()
  // FormatStyle.MEDIUM / FormatStyle.SHORT：适用于LocalDateTime
  
  //本地化相关的格式。如：ofLocalizedDate()
  // FormatStyle.FULL / FormatStyle.LONG /FormatStyle.MEDIUM / FormatStyle.SHORT：适用于LocalDate
  ```

- 自定义的格式。如：ofPattern("yyyy-MM-dd hh:mm:ss")

| **方 法**                     | **描 述**                                           |
| ----------------------------- | --------------------------------------------------- |
| **ofPattern(String pattern)** | 静态方法，返回一个指定字符串格式的DateTimeFormatter |
| **format(TemporalAccessor)**  | 格式化一个日期、时间，返回字符串                    |
| **parse(charSequence test)**  | 将指定格式的字符序列解析为一个日期、时间            |

```java
/**
     * JDK8的api：DateTimeFormatter
     */
    @Test
    public void test5(){
        //自定义的格式。如：ofPattern("yyyy-MM-dd hh:mm:ss")
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

        //格式化：日期、时间 ---> 字符串
        LocalDateTime localDateTime = LocalDateTime.now();
        String strDateTime = dateTimeFormatter.format(localDateTime);
        System.out.println(strDateTime);//2024-11-06 15:01:02

        //解析：字符串 ----> 日期、时间
        TemporalAccessor temporalAccessor = dateTimeFormatter.parse("2024-11-06 17:01:02");
        LocalDateTime localDateTime1 = LocalDateTime.from(temporalAccessor);
        System.out.println(localDateTime1);
    }
```



## 5. Java比较器

我们知道基本数据类型的数据（除boolean类型外）需要比较大小的话，直接运用比较运算符即可，但是引用数据类型时不能直接运用比较运算发来比较大小的。那么，如何解决这个问题呢？

- 在Java中经常会涉及到对象数组的排序问题，那么就涉及到对象之间的比较问题
- Java实现对象排序的方法有两种：
  - 自然排序：java.lang.Comparable
  - 定制排序：java.lang.Comparator



### 5.1 自然排序：java.lang.Comparable

- Comparable接口强行对实现它的每个类的对象进行整体排序。这种排序称为类的自然排序

- 实现Comparable的类必须实现`compareTo(Object obj)`方法，两个对象通过 compareTo(Object obj)方法的返回值来比较大小。如果当前对象this大于形参对象obj，则返回正整数，如果当前对象this小于形参对象obj，则返回负整数，如果当前对象this等于形参对象obj，则返回零。

  ```java
  package java.lang;
  
  public interface Comparable{
      int compareTo(Object o);
  }
  ```

- 实现Comparable接口的对象列表（和数组）可以通过Collection.sort 或 Arrays.sort进行自动排序。实现此接口的对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

- 对于类C的每一个e1和e2来说，当且仅当e1.compareTo(e2)==0与e1.equals(e2)具有相同的boolean值时，类C的自然排序才叫做与equals一致。建议（虽然不是必需的）`最好使自然排序与equals一致`

- Comparale的典型实现：（`默认都是从小大大排序时`）

  - String：按照字符串中字符的Unicode值进行比较
  - Character：按照字符的Unicode值来进行比较
  - 数值类型对应的包装类以及BigInteger、BigDecimal：按照它们对应的数值大小进行比较
  - Boolean：true对应的包装类实例大于false对应的包装类实例
  - Date、Time等：后面的日期时间比前面的日期时间大

```java
public class Product implements Comparable{  //商品类
    private String name;
    private double price;

    public Product() {
    }

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }
    @Override
    public String toString() {
        return "Product{" +
                "name='" + name + '\'' +
                ", price=" + price +
                '}';
    }

    /**
     * 当前的类需要实现Comparable中的抽象方法：compareTo(Object o)
     * 在此方法中指明如何判断当前类的对象的大小。比如：按照价格高低进行排序
     * @param o the object to be compared.
     * @return 正数，当前对象大；负数，当前对象小；0，一样大
     */
//    @Override
//    public int compareTo(Object o) {
//        if (o == this){
//            return 0;
//        }
//        if (o instanceof Product){
//            Product p = (Product) o;
//
//            return Double.compare(this.price, p.price);
//        }
//
//        throw new RuntimeException("类型不匹配");
//    }

    //比较的标准：先比较价格（从大到小），价格相同，再进行名字的比较（从大到小）
    @Override
    public int compareTo(Object o) {
        if (o == this){
            return 0;
        }
        if (o instanceof Product){
            Product p = (Product) o;

            int value = Double.compare(this.price, p.price);
            if (value != 0){
                return -value;
            }else {
                return -this.name.compareTo(p.name);
            }
        }

        throw new RuntimeException("类型不匹配");
    }
}

public class ComparableTest {
    @Test
    public void test1(){
        String[] arr = new String[]{"Tom", "Jerry", "Tony", "Rose", "Jack", "Lucy"};

        Arrays.sort(arr);

        //排序后：遍历
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
    }

    @Test
    public void test2(){
        Product[] products = new Product[5];
        products[0] = new Product("HuaweiMate50pro", 6299);
        products[1] = new Product("Xiaomi13pro", 4999);
        products[2] = new Product("VivoX90pro", 5999);
        products[3] = new Product("Iphone14ProMax", 9999);
        products[4] = new Product("HonorMagic4", 6299);

        Arrays.sort(products);
        for (int i = 0; i < products.length; i++) {
            System.out.println(products[i]);
        }
    }

    @Test
    public void test3(){
        Product p1 = new Product("HuaweiMate50pro", 6299);
        Product p2 = new Product("Xiaomi13pro", 4999);
        System.out.println(p1.compareTo(p2));
    }
}
```



### 5.2 定制排序：java.lang.Comparator

- 思考
  - 当元素的类型没有实现Comparable接口而又不方便修改代码（例如：一些第三方的类，你只有.class文件，没有源文件）
  - 如果有一个类，实现了Comparable接口，也指定列两个对象的比较大小的规则，但是此时此刻我不想按照它预定义的方法比较大小，但是我又不能随意修改，因为会影响其他地方的使用，怎么办？
- JDK在设计类库之初，也考虑到这种情况，所以又增加了一个java.lang.Comparator接口。强行对多个对象进行整体排序的比较
  - 重写compare(Object o1, Object o2)方法，比较o1和o2的大小：如果返回正整数，则表示o1大于o2；如果返回0，表示相等；返回负整数，表示o1小于o2
  - 可以将Comparator传递给sort方法（如Colleactions.sort或Arrays.sort），从而允许在排序顺序上实现精确控制

```java
package java.lang;

public interface Comparator{
    int compare(Onject o1, Object o2);
}
```

举例：

```java
public class ComparatorTest {
    @Test
    public void test1(){
        Product[] products = new Product[5];
        products[0] = new Product("HuaweiMate50pro", 6299);
        products[1] = new Product("Xiaomi13pro", 4999);
        products[2] = new Product("VivoX90pro", 5999);
        products[3] = new Product("Iphone14ProMax", 9999);
        products[4] = new Product("HonorMagic4", 6299);

        //创建一个实现类Comparator接口的实现类的对象

        Comparator comparator = new Comparator() {
            @Override
            //如果判断两个对象o1、o2的大小，其标准就是此方法的方法体要编写的逻辑
            //比如：按照价格从低到高排序
            public int compare(Object o1, Object o2) {
                if (o1 instanceof Product && o2 instanceof Product){
                    Product p1 = (Product) o1;
                    Product p2 = (Product) o2;
                    return Double.compare(p1.getPrice(), p2.getPrice());
                }
                throw new RuntimeException("类型不匹配");
            }
        };
        Comparator comparator1 = new Comparator() {
            @Override
            //如果判断两个对象o1、o2的大小，其标准就是此方法的方法体要编写的逻辑
            //比如：按照名字从低到高排序
            public int compare(Object o1, Object o2) {
                if (o1 instanceof Product && o2 instanceof Product){
                    Product p1 = (Product) o1;
                    Product p2 = (Product) o2;
                    return p1.getName().compareTo(p2.getName());
                }
                throw new RuntimeException("类型不匹配");
            }
        };
        Arrays.sort(products, comparator1);
        for (int i = 0; i < products.length; i++) {
            System.out.println(products[i]);
        }
    }

    @Test
    public void test2(){
        String[] arr = new String[]{"Tom", "Jerry", "Tony", "Rose", "Jack", "Lucy"};

        Arrays.sort(arr, new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                if (o1 instanceof String && o2 instanceof String){
                    String s1 = (String) o1;
                    String s2 = (String) o2;
                    return -s1.compareTo(s2);
                }
                throw new RuntimeException("类型不匹配");
            }
        });

        //排序后：遍历
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
    }
}
```

### 5.3 两种方式对比

- 角度1：
  - 自然排序：单一的，唯一的
  - 定制排序：灵活的，多样的
- 角度2：
  - 自然排序：一劳永逸的，实现了之后以后总是能比较大小的
  - 定制排序：临时的，每次都要临时地指定一下
- 角度3：
  - 自然排序：对应的接口是Comparable， 对应的抽象方法是compareTo(Onject obj)
  - 定制排序：对应的接口是Comparator，对应的抽象方法是compare(Object o1, Object o2)

## 6. 系统相关类

### 6.1 java.lang.System类

- System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。该类位于`java.lang包`。

- 由于该类的构造器是private的，所以无法创造该类的对象。其内部的成员变量和成员方法都是`static`的，所以也可以很方便地进行调用

- 成员变量 Scanner scanner = new Scanner(System.in);

  - System类内部包含`in`、`out`和`err`三个成员变量，分别代表标准输入流（键盘输入）、标准输出流（显示器）和标准错误输出流（显示器）

- 成员方法

  - `native long currentMillis()`：

    该方法的作用是返回当前的计算机时间，时间的表达格式伪当前计算机时间与GMT时间（格林威治时间）1970年1月1日00:00:00所差的毫秒数

  - `void exit(int status)`：

    该方法的作用是退出程序。其中status的值为0代表正常退出，非零代表异常退出。使用该方法可以在图形界面编程中实现程序的退出功能等

  - `void gc()`：

    该方法的作用是请求系统进行垃圾回收。至于系统是否立刻回收，则取决于系统中垃圾回收算法的实现以及系统执行时的情况

  - `String getProperty(String key)`：

    该方法的作用是获得系统中属性名为key的属性对应的值，系统中常见的属性名以及属性的作用如下表所示：

    | 属性名       | 属性说明           |
    | ------------ | ------------------ |
    | java.version | Java运行时环境版本 |
    | java.home    | Java安装目录       |
    | os.home      | 操作系统的名称     |
    | os.version   | 操作系统的版本     |
    | user.name    | 用户的账户名称     |
    | user.home    | 用户的主目录       |
    | user.dir     | 用户的当前工作目录 |

```java
public class OtherAPITest {
    @Test
    public void test1(){
        String javaVersion = System.getProperty("java.version");
        System.out.println("java的version: " + javaVersion);

        String javaHome = System.getProperty("java.home");
        System.out.println("java的home: " + javaHome);

        String osName = System.getProperty("os.name");
        System.out.println("os的name: " + osName);

        String osVersion = System.getProperty("os.version");
        System.out.println("os的version: " + osVersion);

        String userName = System.getProperty("user.name");
        System.out.println("user的name: " + userName);

        String userHome = System.getProperty("user.home");
        System.out.println("user的home: " + userHome);

        String userDir = System.getProperty("user.dir");
        System.out.println("user的dir: " + userDir);
    }

    @Test
    public void test2() throws InterruptedException {
        for (int i = 1; i <=10 ; i++) {
            MyDemo my = new MyDemo(i);
            //每一次循环my就会指向新的对象，那么上次的对象久没有变量引用它了，就成垃圾对象
        }

        //为了看到垃圾回收器工作，要加下面的代码，让main方法不那么快结束，因为main结束就会导致JVM退出，GC也会跟着结束
        System.gc();  //如果不调用这句代码，GC可能不工作，因为当前内存很充足，GC就觉得不着急回收垃圾对象
                      //调用这句代码，会让GC尽快来工作
        Thread.sleep(5000);
    }
}

class MyDemo{
    private int value;

    public MyDemo(int value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "MyDemo{" +
                "value=" + value +
                '}';
    }

    //重写finalize方法

    @SuppressWarnings("removal")
    @Override
    protected void finalize() throws Throwable {
        //正常重写，这里是编写清理系统内存的代码
        //这里输出语句是为了看到finalize()方法被调用的效果
        System.out.println(this + "轻轻地我走了，不带走一段代码....");
    }
}
```

- `static void arraycopy(Object src, int sr`

- `cPos, Object dest, int destPos, int length)`：

  从指定源数组中复制一个数组，复制从指定的位置开始，到目标数组的指定位置结束。常用于数组的插入和删除

  ```java
  public class SystemArrayCopyTest {
      @Test
      public void test1(){
          int[] arr1 = {1, 2, 3, 4, 5};
          int[] arr2 = new int[10];
          System.arraycopy(arr1, 0, arr2, 3, arr1.length);
          System.out.println(Arrays.toString(arr1));
          System.out.println(Arrays.toString(arr2));
      }
  
      @Test
      public void test2(){
          int[] arr = {1, 2, 3, 4, 5};
          System.arraycopy(arr, 0, arr, 1, arr.length-1);
          System.out.println(Arrays.toString(arr));
      }
  
      @Test
      public void test3(){
          int[] arr = {1, 2, 3, 4, 5};
          System.arraycopy(arr, 1, arr, 0, arr.length-1);
          System.out.println(Arrays.toString(arr));
      }
  }
  ```

  

### 6.2 java.lang.Runtime类

每个Java程序都有一个`Runtime`类实例，使应用程序能够与其运行的环境相连接。

`public static Runtime getRuntime()`：返回与当前Java应用程序相关的运行时对象。应用程序不能创建自己的Runtime类实例。

`public long totalMemory()`：返回Java虚拟机中初始化时的内存总量。此方法返回的值可能随时间的推移而变化，这取决于主机环境。默认为物理电脑内存的1/64

`public long maxMemory()`：返回Java虚拟机中最大程度能使用的内存总量，默认为物理电脑内存的1/4

`public long freeMemory()`：返回虚拟机中的空闲内存量。调用gc方法可能导致freeMemory返回值的增加

```java
public class RuntimeTest {
    public static void main(String[] args) {
        Runtime runtime = Runtime.getRuntime();
        long initialMemory = runtime.totalMemory();  //获取虚拟机初始化时堆内存总量
        long maxMemory = runtime.maxMemory();  //获取虚拟机最大堆内存总量
        String str = "";
        //模拟占用内存
        for (int i = 0; i < 10000; i++) {
            str += i;
        }
        long freeMemory = runtime.freeMemory();  //获取空闲堆内存总量
        System.out.println("总内存" + initialMemory / 1024 / 1024 * 64 + "MB");
        System.out.println("总内存" + maxMemory / 1024 / 1024 * 4 + "MB");
        System.out.println("空闲内存" + freeMemory / 1024 / 1024 + "MB");
        System.out.println("已用内存" + (initialMemory-freeMemory) / 1024 / 1024 + "MB");
    }
}
```



## 7. 和数学相关的类

### 7.1 java.lang.Math

`java.lang.Math`类包含用于执行基本数学运算的方法，如初等指数、对数、平方根和三角函数。类似这样的工具类，其所有方法均为静态方法，并且不会创造对象，调用起来非常简单。

- `public staitc doublee abs(double a)`：返回double值得绝对值

```java
double d1 = Math.abs(-5);
double d2 = Math.abs(5)
```

- `public static double ceil(double a)`：返回大于等于参数的最小整数

- `public static double floor(double a)`：返回小于等于参数的最大整数

- `public static long round(double a)`：返回最接近参数的long。（相当于四舍五入方法）

  ```java
  //技巧：round(x) = floor(x+0.5)
  System.out.println(Math.round(12.3));  //12
  System.out.println(Math.round(12.5));  //13
  System.out.println(Math.round(-12.3)); //-12
  System.out.println(Math.round(-12.6)); //-13
  System.out.println(Math.round(-12.5)); //-12 *******
  ```

- public static double pow(double a, double b)：返回a的b次幂

- public static double sqrt(double a)：返回a的平方根

- `public static double random()`：返回[0,1)的随机数

- public static final double PI：返回圆周率

- public static double max(double x, double y)：返回x，y中的最大值

- public static double min(double x, double y)：返回x，y中的最小值

- 其他：acos,asin,atan,cos,sin.tan三角函数

### 7.2 java.math包

#### 7.2.1 BigInteger

- Integer类作为int的包装类，能存储的最大整型值为2\^31+1，Long类也是有限的，最大为2\^63-1。如果要表示再大的整数，不管是基本数据类型还是它们的包装类都无能为力，更不用说进行运算了。

- java.math包的igInteger可以表示`不可变的任意精度的整数`。BigInteger提供所有Java的基本整数操作符的对应物，并提供java.lang.Math的所有相关方法。另外，BigInteger还提供一下运算：模算术、GCD计算、质数测试、素数生成、位操作以及一些其他操作。

- 构造器

  - BigInteger(String val)：根据字符串构建BigInteger对象

- 方法

  - public BigInteger `abs`()：返回此BigInteger的绝对值的BigInteger
  - BigInteger `add`(BigInteger val)：返回其值位（this+val）的BigInteger
  - BigInteger `subtract`(BigInteger val)：返回其值为（this-val）的BigInteger
  - BigInteger `divide`(BigInteger val)：返回其值为（this/val）的BigInteger。整数相除只保留整数部分
  - BigInteger `remainder`(BigInteger val)：返回其值为（this%val）的BigInteger
  - BigInteger[] `divideAndRemainder`(BigInteger val)：返回包含（this/val）后跟（this%val）的两个BigInteger的数组
  - BigInteger `pow`(int exponent)：返回其值为（this^exponent）的BigInteger

  ```java
  @Test
  public void test1(){
      //long bigNum = 123456789123456789123456789L;
      
      BigInteger b1 = new BigInteger("123456789123456789123456789");
      BigInteger b2 = new BigInteger("78923456789123456789123456789");
      
      //System.out.println("和：" + (b1+b2));//错误的，无法直接用+进行求和
      System.out.println("和：" + b1.add(b2));
      System.out.println("减：" + b1.subtract(b2));
      System.out.println("乘：" + b1.multiply(b2));
      System.out.println("除：" + b1.divide(b2));
      System.out.println("余：" + b1.remainder(b2));
  }
  ```

#### 7.2.2 BigDecimal

- 一般的Float类和Double类可以用来做科学计算活工程计算，但在**商业计算中，要求数字精度比较高，故用到java.math.BigDecimal类。**

- BigDecimal类支持不可变的、任意精度的有符号的十进制定点数

- 构造器

  - public BigDecimal(double val);
  - public BigDecimal(String val);  --->推荐

- 常用方法

  - public BigDecimal `add`(BigDecimal augend)
  - public BigDecimal `subtract`(BigDecimal subtrahend)
  - public BigDecimal `multiply`(BigDecimal multiplicand)
  - public BigDecimal `divide`(BigDecimal divisor, int scale, int roundingMode)：divisor是除数，scale指明保留几位小数，roundingMode指明舍入模式（ROUND_UP:向上加1、ROUND_DOWN:直接舍去、ROUND_HALF_UP:四舍五入）

  ```java
  @Test
  public void test1(){
      BigInteger b1 = new BigInteger("12433241123");
      BigDecimal bd = new BigDecimal("12435.351");
      BigDecimal bd2 = new BigDecimal("11");
      System.out.println(b1);
  //    System.out.println(bd.divide(bd2));  //报错：除不尽
      System.out.println(bd.divide(bd2, BigDecimal.ROUND_HALF_UP));
      System.out.println(bd.divide(bd2, 15, BigDecimal.ROUND_HALF_UP));
      }
  ```

  

### 7.3 java.util.Random

用于产生随机数

- `boolean nextBoolean()`：返回下一个伪随机数，它是取自此随机数生成器序列的均匀分布的boolean值。
- `void nextBytes(byte[] bytes)`：生成随机数字节并将其置于用户提供的byte数组中
- `double nextDouble()`：返回下一个伪随机数，它是取自此随机数生成器序列的、在0.0和1.0之间均匀分布的double值
- `float nextFloat()`：返回下一个伪随机数，它是取自随机数生成器序列的、在0.0和1.0之间均匀分布的float值
- `double nextGaussian()`：返回下一个伪随机数，它是取自此随机数产生序列的、呈高斯（正态）分布的double值，其平均值是0.0，标准差是1.0
- `int nextInt()`：返回下一个伪随机数，它是此随机数生成器序列中均匀分布的int值
- `int nextInt(int n)`：返回下一个伪随机数，它是此随机数生成器序列的、在0（包括）和指定值（不包括）之间均匀分布的int值
- `long nextLong()`：返回下一个伪随机数，它是此随机数生成器序列中均匀分布的long值

```java

```

