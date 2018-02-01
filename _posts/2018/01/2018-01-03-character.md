---
layout: post
title: 微服务部署：蓝绿部署、滚动部署、灰度发布、金丝雀发布
category: other
tags: [other]
---

字节流与和字符流的使用非常相，两者除了操作代码上的不同之外，是否还有其他的不同呢？实际上字节流在操作时本身不会用到缓冲区（内存），是文件本身直接操作的，而字符流在操作时使用了缓冲区，通过缓冲区再操作文件，如图12-6所示。

![](http://images.51cto.com/files/uploadimg/20090731/162655699.jpg)

下面以两个写文件的操作为主进行比较，但是在操作时字节流和字符流的操作完成之后都不关闭输出流。

范例：使用字节流不关闭执行

```java
packageorg.lxh.demo12.byteiodemo;
importjava.io.File;
importjava.io.FileOutputStream;
importjava.io.OutputStream;
publicclassOutputStreamDemo05{
publicstaticvoidmain(String[]args)throwsException{//异常抛出，不处理
//第1步：使用File类找到一个文件
Filef=newFile(&quot;d:&quot;&#43;File.separator&#43;&quot;test.txt&quot;);//声明File对象
//第2步：通过子类实例化父类对象
 OutputStreamout=null;
 //准备好一个输出的对象
 out=newFileOutputStream(f);
 //通过对象多态性进行实例化
 //第3步：进行写操作
 Stringstr=&quot;HelloWorld!!!&quot;;
 //准备一个字符串
 byteb[]=str.getBytes();
 //字符串转byte数组
 out.write(b);
 //将内容输出
 //第4步：关闭输出流
 //out.close();
 //此时没有关闭
 }
 }
```


程序运行结果：

![](http://images.51cto.com/files/uploadimg/20090731/162806761.jpg)

此时没有关闭字节流操作，但是文件中也依然存在了输出的内容，证明字节流是直接操作文件本身的。而下面继续使用字符流完成，再观察效果。

范例：使用字符流不关闭执行

```java
packageorg.lxh.demo12.chariodemo;
importjava.io.File;
importjava.io.FileWriter;
importjava.io.Writer;
publicclassWriterDemo03{
publicstaticvoidmain(String[]args)throwsException{//异常抛出，不处理
//第1步：使用File类找到一个文件
Filef=newFile(&quot;d:&quot;&#43;File.separator&#43;&quot;test.txt&quot;);//声明File对象
//第2步：通过子类实例化父类对象
 Writerout=null;
 //准备好一个输出的对象
 out=newFileWriter(f);
 //通过对象多态性进行实例化
 //第3步：进行写操作
 Stringstr=&quot;HelloWorld!!!&quot;;
 //准备一个字符串
 out.write(str);
 //将内容输出
 //第4步：关闭输出流
 //out.close();
 //此时没有关闭
 }
 }
```


程序运行结果：

![](http://images.51cto.com/files/uploadimg/20090731/162913379.jpg)

程序运行后会发现文件中没有任何内容，这是因为字符流操作时使用了缓冲区，而 在关闭字符流时会强制性地将缓冲区中的内容进行输出，但是如果程序没有关闭，则缓冲区中的内容是无法输出的，所以得出结论：字符流使用了缓冲区，而字节流没有使用缓冲区。

提问：什么叫缓冲区？

在很多地方都碰到缓冲区这个名词，那么到底什么是缓冲区？又有什么作用呢？

回答：缓冲区可以简单地理解为一段内存区域。

可以简单地把缓冲区理解为一段特殊的内存。

某些情况下，如果一个程序频繁地操作一个资源（如文件或数据库），则性能会很低，此时为了提升性能，就可以将一部分数据暂时读入到内存的一块区域之中，以后直接从此区域中读取数据即可，因为读取内存速度会比较快，这样可以提升程序的性能。

在字符流的操作中，所有的字符都是在内存中形成的，在输出前会将所有的内容暂时保存在内存之中，所以使用了缓冲区暂存数据。

如果想在不关闭时也可以将字符流的内容全部输出，则可以使用Writer类中的flush()方法完成。

范例：强制性清空缓冲区

```java
packageorg.lxh.demo12.chariodemo;
importjava.io.File;
importjava.io.FileWriter;
importjava.io.Writer;
publicclassWriterDemo04{
publicstaticvoidmain(String[]args)throwsException{//异常抛出不处理
//第1步：使用File类找到一个文件
Filef=newFile(&quot;d:&quot;&#43;File.separator&#43;&quot;test.txt&quot;);//声明File
对象
 //第2步：通过子类实例化父类对象
 Writerout=null;
 //准备好一个输出的对象
 out=newFileWriter(f);
 //通过对象多态性进行实例化
 //第3步：进行写操作
 Stringstr=&quot;HelloWorld!!!&quot;;
 //准备一个字符串
 out.write(str);
 //将内容输出
 out.flush();
 //强制性清空缓冲区中的内容
 //第4步：关闭输出流
 //out.close();
 //此时没有关闭
 }
 }
```


程序运行结果：

![](http://images.51cto.com/files/uploadimg/20090731/163055734.jpg)

此时，文件中已经存在了内容，更进一步证明内容是保存在缓冲区的。这一点在读者日后的开发中要特别引起注意。

提问：使用字节流好还是字符流好？

学习完字节流和字符流的基本操作后，已经大概地明白了操作流程的各个区别，那么在开发中是使用字节流好还是字符流好呢？

回答：使用字节流更好。

在回答之前，先为读者讲解这样的一个概念，所有的文件在硬盘或在传输时都是以字节的方式进行的，包括图片等都是按字节的方式存储的，而字符是只有在内存中才会形成，所以在开发中，字节流使用较为广泛。

字节流与字符流主要的区别是他们的的处理方式

流分类：

1.Java的字节流

 InputStream是所有字节输入流的祖先，而OutputStream是所有字节输出流的祖先。

2.Java的字符流

 Reader是所有读取字符串输入流的祖先，而writer是所有输出字符串的祖先。

InputStream，OutputStream,Reader,writer都是抽象类。所以不能直接new 

字节流是最基本的，所有的InputStream和OutputStream的子类都是,主要用在处理二进制数据，它是按字节来处理的

但实际中很多的数据是文本，又提出了字符流的概念，它是按虚拟机的encode来处理，也就是要进行字符集的转化

这两个之间通过 InputStreamReader,OutputStreamWriter来关联，实际上是通过byte[]和String来关联

在实际开发中出现的汉字问题实际上都是在字符流和字节流之间转化不统一而造成的

在从字节流转化为字符流时，实际上就是byte[]转化为String时，

public String(byte bytes[], String charsetName)

有一个关键的参数字符集编码，通常我们都省略了，那系统就用操作系统的lang

而在字符流转化为字节流时，实际上是String转化为byte[]时，

byte[] String.getBytes(String charsetName)

也是一样的道理至于java.io中还出现了许多其他的流，按主要是为了提高性能和使用方便，如BufferedInputStream,PipedInputStream等

                    
