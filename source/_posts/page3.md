---
title: page3
date: 2020-07-04 20:51:34
tags:
---


**什么是异常？**

异常，就是程序运行过程中出现的错误。
区别于编译错误，异常通常是发生在程序运行过程中的，比如因为思维的不够缜密，导致一些难以检查到的问题出现。

***
**Java的异常机制**

作为一个面向对象语言，Java将异常单独定义为class类，不同类型的异常情况用不同的类来表示，比如说下面这段代码，以及即将出现的 NullPointerException：

```java
public class ExceptionTest {

    public static void main(String[] args){

      int len = getStrLength(null);
      System.out.println(len);
    }
    // 获取字符串的长度
    public static int getStrLength(String str){
        return str.length();
    }

}
```
结果如下，我们称之为**异常堆栈**，其中蕴含了很多关于异常的信息，如异常出现的文件、方法、甚至到哪一行。看到这个 java.lang.NullPointerException 类了吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200425233906262.png)
（当然这个故意简单的出错代码，还是容易查找的啦）

Java通过类把所有的异常当作对象进行处理，而所有的异常类都继承于一个已经提前定义的基类 java.lang.Throwable，如下图：

![异常类](https://img-blog.csdnimg.cn/20200425193138483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)
我们遇到的 **异常 Exceptions** 和 **错误 Errors** 都继承自 Throwable 这个基类。而异常又可分为运行时异常 Runtime Exceptions 和 非运行时异常 Other Exceptions。我们简单说说他们的区别：

 **1. Errors & Exceptions**
 
 前者是程序员无法处理的非代码性错误，一般会发生在运行应用的层面，这时JVM，即Java虚拟机便会终止线程，歇菜不干；而后者则是程序员可以动手操作处理的问题
 	
 **2. Runtime Exceptions & Other Exceptions**
 
 运行时异常都是 RuntimeExceptions 及其子类，例如空指针 NullPointerException、下标越界 ArrayIndexOutOfBoundsException 等。属于不检查异常，可捕获可不捕获，一般是由程序员的逻辑出错造成，比较难检查出来，所以敲代码时就得谨慎考虑。
 	
 非运行时异常属于 RuntimeExceptions 异常以外的，但也都在 Exceptions之中，例如 IOExpection输入输出流异常和用户自定义异常。属于检查型异常，若不处理则程序无法编译通过。所以通常使用 try-catch 捕获，或者 throw/throws 抛出。
 ***

**异常的处理：**

**一、捕获处理：** 使用关键字 try catch finally

```java
int len = 0; // 提升作用域
try {
	len = getStrLength(null); // 还是上文那个方法，不再详写
} catch (NullPointerException e) {
	// 可以记录一些异常信息或执行其他特定代码
	System.out.println("something wrong");
} finally {
	//无论try中是否有异常，都会执行的代码
	System.out.println("got you bro");
}
```
**try：** 尝试运行其中的代码块，收到异常监控。若无异常则跳过 catch 进入 finally 代码块；否则会抛出异常对象
**catch：** 捕获 try 中出现的异常，并将其与括号中的 Throwable 类型参数进行比对，如果异常类型匹配，则执行代码块，并将参数指向抛出的异常对象。
**finally：** 紧随 catch 之后，总在方法返回前执行。

**使用注意：** 

 - 不可单独使用，一段完整的捕获，try 必须有一个，catch 可零或多个，finally 至多一个。
 - 三个代码块内的变量作用域仅限其内部，相互独立不可访问。
 - 多个 catch 时会自上而下匹配异常类型，匹配成功即执行，并再不执行其他的 catch 代码块。
 

```java
public class ExceptionTest {

    public static void main(String[] args) {
        int number = 0;
        try {
          num = getInt("气密呛噶");
        } catch (NumberFormatException e) {
        } catch (NullPointerException e) {
        }

        System.out.println(num);
    }

    // 字符串转换为数字
    public static int getInt(String str) {
        return Integer.parseInt(str);
    }

}
```

***

**二、抛出处理：** 使用关键字 throw throws

```java
public class ExceptionTest {

    public static void main(String[] args) {
        int num = 0;
        try {
            num = getInt(null);
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        System.out.println(num);
    }

    public static int getInt(String str) throws Exception {
        if (str == null) {
            throw new Exception("不能为空");
        }
        return Integer.parseInt(str);
    }
}
```

**throw：** 用于方法体内部，来抛出 Throwable 异常，并且在方法名头部进行异常类型的声明。同时，异常会上抛给方法的调用者，由其进行检查处理，即可运用 try-catch-finally 进行捕获。
**throws：** 用于方法名头部的异常类型声明，即方法内可能抛出的异常类型。只有抛出了这个类型的检查异常，方法调用者才会处理或继续抛出。

***

**自定义异常**

当然，Throwable提供的异常已经足够多，但还是会有一些异常情况无法被清晰地表达。因此，我们可以通过创建自己的异常类，即自定义异常来进行异常的表达。前面说过，异常都是 Exceptions 的子类，我们只需将自己创建的异常类继承 Exception 即可。

在这里我们创建一个自定义异常类 UnknownException 并进行测试
```java
package exception;

public class UnknownException extends Exception{ // 不要忘记继承 Exception
  	// 一个空的自定义异常类构造函数
    public UnknownException(){
	
	}
    // 父类构造函数，后面会简说一下
    public UnknownException(String msg) {
      super(msg);
    }
    
}
```

```java
package test;

import exception.UnknownException;

public class ExceptionTest {
    
    public static void main(String[] args){
        try{
          createAccount( "伍六七", null);
        } catch (UnknownException e) {
          System.out.println(e.getMessage()); // 后面也会说这个东西的
        } finally {
          System.out.println("恭迎首席！！");
        }
        
    }
    
    // 注意方法声明的后面，同时声明了我们自创的异常类
    public static String createAccount(String nickname, String password) throws UnknownException{
    	// 根据条件抛出异常
        if (nickname == null){
          throw new UnknownException("还不上号？");
        } else if (password == null){
          throw new UnknownException("密码忘了？");
        }
        return "那个男人回来了！";
    }
    
}
```
下面这个就是运行结果啦，小伙伴们可以自己试试！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426135245468.png)

那么，我们是怎么做到在抛出异常的时候输出 “密码忘了” 的呢？这就事关自定义异常类中的构造函数和类方法方法啦！

**先说构造函数：**

不知你是否注意到，我们自定义抛出的异常，是使用 throw **new** UnknownException("密码忘了？")，注意这个 new，这其实就是告诉我们，我们在抛出异常的时候，就是在抛出一个新创建的类，只不过它是一种异常类。

而括号中的东西，想必你也知道它是啥了吧？其实就是构造函数的参数嘛！所以我们除了在自定义类里写有一个自己的空构造函数，还要写一个带有参数的构造函数。然而这个参数，却并不是传递给自定义类自己，而是通过使用 super()传递给其父类的构造函数。为啥捏？继续看就知道了。

我们先了解一下父类 Exception 的构造函数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426141933517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjM2MzIyNg==,size_16,color_FFFFFF,t_70)
就这样，通过父类的构造方法，我们将参数传入，但是又如何输出呢？

**再说说这个方法：**

这就用到了类方法 **getMessage()**，它能够返回我们传入的这个参数。这个方法属于 Throwable类，层层继承后便也可以由我们的自定义异常类所使用了。然而我们使用的例子只是针对仅传入一个 String 参数的返回方法，其他的参数表也会对应不同的返回方式。

除此之外，Throwable类 还提供有其他方法，比如：**getCaues()**，用于返回抛出异常原因，若不存在或未知，则返回 null。

所以这也说明，我们为什么要将参数传递给父类的构造函数而不是自己，这样子类就可以直接使用父类本就定义好的一些方法，对我们传入的参数进行处理，而不需要子类自己再多花功夫啦。

**“老爸的绝世秘籍如今传授于你，你好好看，好好学，好好用”**

***

好啦，以上便是我的思考和总结，本人编程小白一枚，初尝博客，一旦开始就要把写博客的习惯坚持下来！但是本人能力有限，知识浅薄，整理得肯定有很多不足之处，欢迎大神指教与批评！我始终相信虚心接受指导才能进步的！
![！多谢指教！](https://img-blog.csdnimg.cn/20200426150336954.jpg#pic_center)


注：本文部分内容启发参考自博客 [Java异常体系结构](https://blog.csdn.net/Junlixxu/article/details/6096266?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522158778124719725256752697%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=158778124719725256752697&biz_id=0&utm_source=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v25-1)
