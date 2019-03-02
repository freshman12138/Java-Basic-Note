# jAVA IO流 #
2019-03-01 9:21:15 
## 1、分类 ##
### A、根据处理数据类型的不同：字节流、字符流 ###
### B、根据数据流向的不同：输入流、输出流 ###

## 2、区别 ##
### 字节流和字符流的区别：(1).节流能够处理所有类型的数据（包括文字，图片，音频）而字符流只能够处理字符类型的数据. (2).字节流都是直接对数据进行操作的而字符流需要缓冲区通过缓冲区处理数据.  ###
#### 总结:使用的字节流的更为广泛 ####
### 输入流和输出流的区别:输入流只能进行读操作,输出流只能进行写操作... 程序中需要根据待传输数据的不同特性而使用不同的流###

## JAVA流类图结构 ##
![](https://i.imgur.com/EF7gMCe.png)

## 3、JAVA IO流对象 ##
### 1、输入字节流 InputStream ###
InputStream是所有输入字节流的父类，是一个抽象类。ByteArrayInputStream，StringBufferInputStream，FileInputStream是分别从Byte数组，String，文件中读取数据的。
### 2、输出流字节流 OutputStream ###
OutputStream是所有输出字节流的父类，是一个抽象类。
ByteArrayOutputStream，FileOutputStream是分别向Byte数组，文件中写入数据的。

#### 总结：要进行对字符的读写操作可以用字符流reader/writer下的BufferedReader/BufferedWrite或InputStreamReader/OutputStreamWrite进行读写操作。。
#### 
####   但是在对不是字符的文件进行传输时（例：图片，视频，音频）要运用字节流进行读写操作InputStream/OutputStream 或者BufferedInputStream/BufferedOutputStream进行对文件的读写操作  ####
`
package com.JavaIO;

import java.io.*;
 
public class TransferDemo {

    public static void main(String[] args) throws IOException {
        InputStream inputStream = new FileInputStream("D:/女儿国.mp3");
        BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);
        OutputStream outputStream = new FileOutputStream("F:/hello.mp3");
        //BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(outputStream));
        BufferedOutputStream bufferedOutputStream =new BufferedOutputStream(outputStream);
        int temp=0;
        while((temp=bufferedInputStream.read())!=(-1))
        {
            //outputStream.write(temp);
            bufferedOutputStream.write(temp);
        }
        bufferedInputStream.close();
        bufferedOutputStream.close();
        inputStream.close();
        outputStream.close();
    }

}

`
以上就是对mp3文件进行操作的时候必须采用字节流进行读写操作。。 



 
 package com.JavaIO;

import java.io.*;

public class ReaderWriteDemo {

    public static void main(String[] args) throws IOException {

        Reader reader = new FileReader("D:/hello.txt");
        BufferedReader bufferedReader = new BufferedReader(reader);
        //Writer writer = ;
        BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter("D:/copy.txt"));
        String line = null;
        while((line=bufferedReader.readLine())!=null)
        {
            //System.out.println(line);
            bufferedWriter.write(line);
        }
        bufferedReader.close();
        bufferedWriter.close();
        reader.close();
    }}

文档文件就可以使用字符流或者字节流进行读写操作

#### 一般情况下我们采用BufferedWrite/BufferedReader 或者 BufferedInputStream/BufferedOutputStream 进行字符流字节流的读写操作，因为在内存上读写比较快 但是在内存上写入数据后必须刷新一下要不然读取数据会读取不了数据####

### 3、字符输入流Reader ###
Reader是所有的字符输入流的父类，他是一个抽象类。CharArrayReader/StringReader分别读出Char数组，String的数据。
#### 4、字符输出流writer ####
writer是所有输出字符流的父类，它是一个抽象类。
CharArrayWriter/StringWriter分别写入Char数组，String的数据。

## 4.字符流与字节流转换
转换流的特点：

（1）其是字符流和字节流之间的桥梁

（2）可对读取到的字节数据经过指定编码转换成字符

（3）可对读取到的字符数据经过指定编码转换成字节



何时使用转换流？

当字节和字符之间有转换动作时；

流操作的数据需要编码或解码时。

具体的对象体现：
InputStreamReader:字节到字符的桥梁

OutputStreamWriter:字符到字节的桥梁

这两个流对象是字符体系中的成员，它们有转换作用，本身又是字符流，所以在构造的时候需要传入字节流对象进来。 
##

## 5、File类
File类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。 File类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法。
##

## 6、Zip ##
[https://blog.csdn.net/dabing69221/article/details/17074763](https://blog.csdn.net/dabing69221/article/details/17074763)