---
title: page1
date: 2020-07-04 17:06:55
tags:
---
 ### Java爬虫案例之获取视频详细点赞、收藏等参数

最近是b站的11周年庆，相信不少朋友也看过了这个特映视频《喜相逢》了吧？看完视频给个三连支持，衷心祝愿小破站越来越好！不过在长按完大拇指后，看着这极高的点赞和硬币，你是不是有种想了解它的实际数目的好奇心？哈哈，那么咱就立马操练起来，用编程来拿到这些数据吧！
![喜相逢](https://img-blog.csdnimg.cn/20200629174741144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)

 #### 一、运用开发者工具
首先打开b站的首页，搜索框内键入“喜相逢”。
![截图](https://img-blog.csdnimg.cn/20200629171822239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)
（哦豁又有新背景图了，所以。。？）

[上篇文章](https://blog.csdn.net/weixin_46363226/article/details/106340356) 有讲到如何用开发者工具，这里不再赘述。进入视频界面并记得打开开发者工具刷新一下。漫长的筛查过后，只要你英语好一点，有耐心一点，就能左右对比到我们所要的信息了：
![重要信息](https://img-blog.csdnimg.cn/20200629175511329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629172339596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)

注意：如果你自己多找几个视频，会发现每个视频界面对应的referer的不一样的，所以后面代码里构造请求对象时这部分不能写死。

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
// json数据解析
import com.alibaba.fastjson.JSON;
import java.util.Map;
```

##### 2、获得文本数据：
不同于上一次下载图片，这次我们只要求把json格式的文本数据获取来，并进行解析得到我们要的信息即可。

```java
// 上文提到过referer是动态的，所以我们作为参数传入
public String getContent(String url, String referer) {

        // 先实例化一个 OkHttpClient
        OkHttpClient client = new OkHttpClient();
        // 实例化 Request，定义请求的各种参数；header添加以模拟浏览器访问环境
        Request request = new Request.Builder()
                .url(url)
                .addHeader("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36")
                .addHeader("Referer", referer)
                //.addHeader("Host","www.bilibili.com")
                .build();

        String result = null;
        // 调用可能失败并抛出异常，需进行捕捉
        try {
            // 构建调用对象
            Call call = client.newCall(request);
            // 请求执行后的结果
            Response rep = call.execute();
            // 获取状态码
            int code = rep.code();
            System.out.println("状态码为"+ code);
            // 获取调用结果，并返回字符串内容的方法（适用请求字符文本文件）
            result = rep.body().string();
        } catch (IOException e) {
            System.out.println("request" + url + " error . ");
            e.printStackTrace();
        }
        return result;
    }
```
##### 3、main函数打印

```java
import com.alibaba.fastjson.JSON;
import okhttp3.Call;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

import java.io.IOException;
import java.util.Map;

public class TextFileAsker {

    public String getContent(String url, String referer) { ... }

    public static void main(String[] args) {
        TextFileAsker asker = new TextFileAsker();
        String url = "https://api.bilibili.com/x/web-interface/archive/stat?aid=626032318";
        String referer = "https://www.bilibili.com/video/BV1yt4y1X75B";
        // 获得文本数据
        String content = asker.getContent(url, referer);
		// 根据开发者工具里Preview里的数据结构进行json解析
        Map contentObj = JSON.parseObject(content, Map.class);
        Map data = (Map)contentObj.get("data");
		// 依次提取所需信息
        String like = data.get("like").toString();
        String coin = data.get("coin").toString();
        String view = data.get("view").toString();
        String share = data.get("share").toString();
        String favorite = data.get("favorite").toString();
        System.out.println("播放量 点赞 收藏 硬币 转发");
        System.out.println(view + " " +like + " " +favorite+ " " +coin + " " +share + " ");
    }

}

```
运行结果如下：
![运行结果](https://img-blog.csdnimg.cn/20200629175557388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)
（就写了点博客的时间，又多了好多播放量）

以上便是关于视频点赞数等一些参数的详细数据获得方式了，对于类似的情况，都可以自己在开发者工具里进行仔细搜索，细心一点，耐心一点，你总能让它出现在自己的程序输出里的。