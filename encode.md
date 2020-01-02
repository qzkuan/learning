# Java中HTTP网络传输中文编码问题



## 1、java中new String(str.getBytes(“utf-8”),“iso-8859-1”)编码详解

>  前提是str存放的是汉字

1. 如果是`new String(str.getBytes(“gbk”),“gbk”)`时，可以分为两步：

   * 第一步：`byte[] bytes=str.getBytes(“gbk”)`

     告诉java虚拟机将中文以“gbk”的方式转换为字节数组。一个汉字对应两个字节.

   * `String s=new String(bytes,“gbk”)`   // 执行后的s就是第一步的str。

​        告诉虚拟机将字节数组中的字节以“gbk”的方式将每2个字节组装成一个汉字。此汉字s就是第一步str代表的汉字.

2. 如果`new String(str.getBytes(“gbk”),“iso8859-1”)`时

   * 对应的第二步便是：

      `String s=new String(bytes,“iso8859-1”)`时，此时是将每1字节组装成一个“?” 。此时的s是若干个“?”，我们可以把“?”看做是一种特殊的汉字，它代表的信息并没有损失是可以还原回来的。

3. 如果`new String(str.getBytes(“gbk”),“utf-8”)`时

   * 对应的第二步便是：

     `String s=new String(bytes,“utf-8”)`时，此时是将每3字节组装成一个汉字。此汉字s就是第一步str代表的汉字。

> 实际的网络传输的过程中，是将汉字以utf-8编码后在网上传输，此种方式的好处就是节省带宽流量。IE浏览器中的internet选项下高级栏中有说“总是以utf-8传输数据”。
>
> 注意当字节数组用iso8859-1组装成的“?”，用utf-8编码此特殊的汉字时就会变成2个字节。

==getBytes()方法==

* 在Java中,String的getBytes()方法是得到一个操作系统默认的编码格式的字节数组。这表示在不同的操作系统下,返回的东西不一样!

  String.getBytes(Stringdecode)方法会根据指定的decode编码返回某字符串在该编码下的byte数组表示,如:
  byte[] b_gbk = "中".getBytes("GBK");
  byte[] b_utf8 = "中".getBytes("UTF-8");
  byte[] b_iso88591 = "中".getBytes("ISO8859-1");
  将分别返回"中"这个汉字在GBK、UTF-8和ISO8859-1编码下的byte数组表示,此时

  b_gbk的长度为2,

  b_utf8的长度为3,

  b_iso88591的长度为1。

 

==new String(byte[],decode)方法==

* 而与getBytes相对的,可以通过new String(byte[], decode)的方式来还原这个"中"字,

  这个new String(byte[],decode)实际是使用指定的编码decode来将byte[]解析成字符串.
  String s_gbk = new String(b_gbk,"GBK");
  String s_utf8 = new String(b_utf8,"UTF-8");
  String s_iso88591 = new String(b_iso88591,"ISO8859-1");
  通过输出s_gbk、s_utf8和s_iso88591,会发现s_gbk和s_utf8都是"中",而只有s_iso88591是一个不被识别的字符（可以理解为乱码）,为什么使用ISO8859-1编码再组合之后,无法还原"中"字？原因很简单,因为ISO8859-1编码的编码表根本就不包含汉字字符,当然也就无法通过"中".getBytes("ISO8859-1");来得到正确的"中"字在ISO8859-1中的编码值了,所以，再通过newString()来还原就更是无从谈起。
  因此,通过String.getBytes(Stringdecode)方法来得到byte[]时,一定要确定decode的编码表中确实存在String表示的码值,这样得到的byte[]数组才能正确被还原。

  注意：

  有时候,为了让中文字符适应某些特殊要求(如httpheader要求其内容必须为iso8859-1编码),可能会通过将中文字符按照字节方式来编码的情况,如:
  String s_iso88591 = newString("中".getBytes("UTF-8"),"ISO8859-1"),这样得到的s_iso8859-1字符串实际是三个在ISO8859-1中的字符,在将这些字符传递到目的地后,目的地程序再通过相反的方式Strings_utf8 = newString(s_iso88591.getBytes("ISO8859-1"),"UTF-8")来得到正确的中文汉字"中"，这样就既保证了遵守协议规定、也支持中文。

## 2、网络请求中，中文字符的编解码实现：URLEncoder.encode()和URLDecoder.decode()

### demo

```java
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.net.URLEncoder;
 
public class JavaStudy {
    public static void main(String[] args) throws UnsupportedEncodingException {
        //编码
        String strUTF = "上海";
        String encode = URLEncoder.encode(strUTF, "utf-8");
        System.out.println(encode);//%E4%B8%8A%E6%B5%B7
 
        //解码
        String decoStr = "%E4%B8%8A%E6%B5%B7";
        String decode = URLDecoder.decode(decoStr, "utf-8");
        System.out.println(decode);//上海
        
    }
}

```

> 注意事项

1. ==URLEncoder.encode(String s, String enc)==
   使用指定的编码机制，将字符串编码为 `application/x-www-form-urlencoded` 格式 

   发送请求的时候使用。

   ==URLDecoder.decode(String s, String enc)== 
   使用指定的编码机制，对 `application/x-www-form-urlencoded` 字符串解码。 

   接受请求的时候使用。

2. 编码，解码的类型要一致。

