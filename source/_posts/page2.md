---
title: page2
date: 2020-07-04 20:51:18
tags:
---

 ### Java爬虫案例之获取网站图片

最近逛b站，发现导航栏上的背景图有点好看。

![导航图](https://img-blog.csdnimg.cn/20200531154812549.png#pic_center)
于是点击右键另存为，却发现得到的是b站页面的html文件，根本不是图片。原来这张图并不能直接获取到，那么为了得到这张背景图，用Java网络编程我们可以怎么做？
 
 #### 一、运用开发者工具
 图虽然没法直接拿到，但好歹也是这个页面元素的一部分嘛，所以我们可以打开开发者工具或者右键检查，得到对应图片的相关信息：
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200531161256238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70#pic_center)

为了从网页上获取到这张图，我们需要获取相关的信息，我们在上张图的基础上点击Headers界面：
![headers](https://img-blog.csdnimg.cn/20200531161541598.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)
有用的信息：

 - Request URL: "https://i0.hdslb.com/bfs/archive/ed92db305ae43c7fc8a59b1789934caa2636b876.png"
   （这是我们要请求的网址）
 - Request Method，请求方式为GET
 - Http Headers中的 User-Agent、Referer、Host
 （这里 Referer: https://www.bilibili.com/，域名即为 Host: www.bilibili.com）

拿到有用的消息之后，我们就可以用在我们的代码上了。

#### 二、代码实现
##### 1、准备：
安装okhttp库，能实现简单快速的http调用：
```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.1.0</version>
</dependency>
```
导包：

```java
// 异常
import java.io.IOException;
// http请求
import okhttp3.Request;
import okhttp3.Call;
import okhttp3.OkHttpClient;
// 文件下载
import java.io.File;
import java.io.FileOutputStream;
```
##### 2、获取二进制编码
图片内容属于二进制编码数据，并非可直接阅读的文本字符，所以我们需要首先得到这个图片的二进制编码数据并返回。详情请看代码：

```java
public byte[] getBytes(String url) {

        // okHttpClient 实例
        OkHttpClient okHttpClient = new OkHttpClient();
        // 定义一个的request，并添加headers模拟请求该网址的浏览器环境
        Request request = new Request.Builder()
                .url(url)
                .addHeader("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36")
                .addHeader("Referer", "https://www.bilibili.com/")
                .addHeader("Host","www.bilibili.com")
                .build();
		byte[] bytes = null;
		// 调用过程可能失败，在try中运行
        try {
            // 构建调用对象
            Call call = okHttpClient.newCall(request);
            // 执行请求，获得请求结果对象
            Response response = call.execute();
            // 请求结果对象的操作如下：
            // 获取响应码
            int code = response.code();
            System.out.println("响应码为：" + code);
            // 非字符文本文件，用 bytes()获取二进制编码内容
            bytes = response.body().bytes();
            System.out.println("图片大小为：" + bytes.length + "字节");
        } catch (IOException e) {
            // 抓取异常
            System.out.println("request " + url + " error . ");
            e.printStackTrace();
        }
		return bytes;
    }
```
##### 3、文件下载
获取到对应的二进制编码内容后，我们就可以将它写入文件啦：

```java
public void getPicture(byte[] bytes) {

        try {
            // 写文件，文件名命名
            File file = new File("b站导航栏背景图");
            FileOutputStream fos = new FileOutputStream(file);
            // 将二进制编码内容写入文件
            fos.write(bytes);
            // 文件刷新和关闭
            fos.flush();
            fos.close();

            System.out.println("下载成功");
        } catch (IOException e) {
            e.printStackTrace();
        }

}
```

##### 3、main函数测试
接下来，我们就用已经封装好的方法，进行图片的下载吧：

```java
import okhttp3.Call;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

public class PictureAsker {

    public byte[] getBytes(String url) { ... }

    public void getPicture(byte[] bytes) { ... }

    public static void main(String[] args) {
        PictureAsker picAsker = new PictureAsker();
        String url = "https://i0.hdslb.com/bfs/archive/ed92db305ae43c7fc8a59b1789934caa2636b876.png";
        byte[] bytes = picAsker.getBytes(url);
        picAsker.getPicture(bytes);
    }

}
```
运行结果如下：

![运行结果](https://img-blog.csdnimg.cn/20200531184515719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)
下载的图片如下：
![获得图片](https://img-blog.csdnimg.cn/20200531184607905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)

以上便是Java网络编程的爬图技巧，学到了的话不妨自己找几个网站的图片试试手？

（ 补充：上面的运行结果，实际上是在创建Request的时候删去Host header才运行成功的，目前还未能查到原因，有大神了解的话希望在评论区指出啊！）