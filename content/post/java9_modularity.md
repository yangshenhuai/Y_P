---
title: "Java9模块化"
date: 2017-08-27T00:00:47+08:00
draft: false
tags: ["Java"]
author: "Peter Yang"
---

Java9 今年[九月份][1]就要正式发布了，这次发布最大的change就是模块化，这也可能是Java问世以来改变最大的一次版本发布.因为整个jdk都被模块化了 之前 "大而全" 的rt.jar(runtime)终于寿终正寝,取而代之的是很多个相对独立的模块.模块化的Java提供了`更强的封装`,使程序员使用`定义良好的接口`来相互调用 并且要`显式声明依赖`,这些都能保证开发中避免遇到不必要的错误.
# JDK9 modules
在Java9之前，在 `$JAVA_HOME/jre/lib` 中有一个很大的`rt.jar`(60M左右),里面包含了所有Java自己的类（比如`java.lang.List`,`java.lang.String`等）,当你的App（就算只是Hello world）启动的时候 JVM都会load rt.jar里面所有的class,即使你用不到里面的有些class（比如说 `awt` 或 `applet`）.

Java9没有rt.jar,取而代之的是很多modules，在`$JAVA_HOME/jmods`目录下可以看到有90多个module,从下图可以看到这些module之间的依赖关系，所有的module都默认depend on java.base ![][2]
# module-info.java
每个module都要有一个`module-info.java`去定义module,下面是java.sql的module-info.java文件


    module java.sql{  
    exports java.sql;  
    exports javax.sql;  
    exports javax.transaction.xa;  
    &nbsp;  
    requires java.logging  
    requires java.xml  
    }  



在module-info.java中用`module`关键字去定义module,后面跟module的名字,然后用 `exports` 关键字去声明当前module会暴露哪些包里面的类, 如果module里的一个包没有用`exports`来暴露,即使这个包里的类和方法都是public的，其它的module依旧引用不到 .用 `requires` 关键字去声明当前的module依赖哪些module

# 创建第一个简单的module
用Java9创建一个Hello world，先创建一个类似下面这样的目录结构 ![][3] module-info.java 里面的内容如下,`requires java.base`是可以省略的，因为Java默认会加上这句 ,Main.java 就是写一个main 方法打印Hello world！


    module hello {  
    requires java.base  
    }  



下面就可以用javac去编译java文件了,先用`javac -version`确认Java的version是9,然后在`java9-study`目录下执行 `javac --module-source-path src -d out $(find . -name '*.java')` module-source-path 用来指定你的module所在的目录.执行完这条命令后，Java文件应该会被编译成Class文件并保存在`out`目录下.

有了class文件就可以用来执行了, --module-path 类似 class-path 用来指定module所在的位置


    $ java --module-path out -m hello/com.hello.Main  
    Hello world!  

# 创建有依赖关系的module
上边创建的只是简单的单一的module,下面看下如何创建两个有依赖关系的module.创建类似下面的目录 ![][4] 在`hello` module 中的`Main` 类中调用 print_service module中的PrintService.

hello module的module-info.java


    module hello {  
    requires java.base;  
    requires print_service;  
    }  



print_service的module-info.java


    module print_service{  
    exports com.print;  
    }  



print_service 中的 PrintService.java


    package com.print;  
    &nbsp;

    public class PrintService {  
    public void print(String arg) {  
    System.out.println(arg);  
    }  
    }  



hello module 中的 Main.java


    package com.hello;  
    import com.print.PrintService;  
    &nbsp;  
    public class Main {  
    &nbsp;  
    public static void main(String args[]) {  
    new PrintService().print("Hello world");  
    }  
    }  



用javac命令编译，然后用java命令执行依旧可以正常运行. 可以看到在print_service中还有一个类`PrivateService`在包`com.private`下，因为我在module-info.java文件中并没有`exports`这个包，所以在module hello中是无法访问到这个类的,如果在`Main.java`加上对这个类的访问再进行编译,就会报错`package com.privates is not visible`


    package com.hello;  
    import com.print.PrintService;  
    import com.privates.PrivateService; //this will fail!!  
    public class Main {  
    public static void main(String args[]) {  
    new PrivateService().service();  
    new PrintService().print("Hello world!!!!!");  
    }  
    }  



同样的如果在module `hello` 的module-info.java文件中不声明`requires print_service` 就对PrintService 进行调用，一样会报`package com.print is not visible`. 只有在 moudle `hello` 的 module-info.java中加上 `requires print_service`,在hello里面的类可以访问所有print_service `exports出来的所有public type` 并且用`requires` Readablity**不是**传递的, 如果想要Readablity是传递的话，要用 `requires transitive`，如下图这样配置我们的application只要requires了`java.sql`就自动可以用`java.logging`里面暴露的类了 ![][5]

# Serivces
上面的例子其实break了[`SOLID`][6]原则中的`D(面向接口而不是实现编程)`.可以考虑用`工厂模式`去解决这个问题， 但是如果使用工厂模式,工厂依然在编译时需要知道你 的所有实现, 所以不能完全解决这个问题. 其实Java9提供了`Services`来解决这个问题, 下面可以通过例子看下到底怎么用`Services`隔离接口和实现.

首先将上边的例子扩展一下. `hello` 依旧是作为服务调用方,又叫`Consumer`,将原来的 print_service 改成 `print_service_api`,这将是`API-Only`的module,严格来说这个module里面应该 只包含interface,但实际上你可以放任何类在 `print_service_api` module中,然后加两个具体的服务实现的module:`print_service_A`和`print_service_B` ,这两个module也叫做 `Provider`.修改后整个项目的目录结构如下 ![][7]

先看下`print_service_api`的`module-info.java`. 非常简单，把接口所在的包exports出来，否则后面所有其他的模块都访问不到接口`PrintService`


    module print_service_api {  
    exports com.print.api;  
    }  



再看下 Provider `print_servce_A`的`module-info.java`.首先声明了需要调用模块`print_service_api`, 所以我们可以在这个模块中看到接口`com.print.api.PrintService`. 然后用`provides`关键字声明了一个`com.print.api.PrintService`的实现`com.print_a.PrintServiceA`.可以看到我们并没有exports任何的包,所以其他module无法直接看到这个module里面的任何类,这样就把实现类隐藏起来了.


    module print_service_A{  
    requires print_service_api;  
    provides com.print.api.PrintService with  
    com.print_a.PrintServiceA;  
    }  



另外一个Provider `print_service_B` 前面的 `print_servce_A` 相似,只不过它声明的服务实现是`com.print_b.PrintServiceB`.它的`module-info.java` 如下


    module print_service_B{  
    requires print_service_api;  
    provides com.print.api.PrintService with  
    com.print_b.PrintServiceB;  
    }  



最后看下Comsuer `hello`,首先它也要声明需要接口模块`print_service_api`.然后用`uses`声明了本模块需要使用到`print_service_api`的实现类,通过后面的代码会看到去找实现类是通过`ServiceLoader`机制实现的


    module hello {  
    requires java.base;  
    requires print_service_api;  
    uses com.print.api.PrintService;  
    }  



从前面四个模块的`module-info.java`可以看到，他们之间的关系如下图 ![][8]

下面来看下模块里的代码，首先接口模块里面的接口代码是最简单的一个接口定义


    package com.print.api;  
    &nbsp;  
    public interface PrintService{  
    public void print(String arg) ;  
    }  



然后是Provider模块里的接口实现类，也非常简单.直接写一个类去实现接口类就可以了.注意:用这种方式一定要保证实现类有无参默认构造函数 可以看到因为api模块exports了包`com.print.api`,并且在实现模块我们requires了api模块，所以我们能访问到接口模块里面的`PrintService`


    package com.print_a;  
    import com.print.api.*;  
    public class PrintServiceA implements PrintService {  
    public void print(String arg) {  
    System.out.println("This is Service A: " + arg);  
    }  
    }  



其实还有另外一种方式去写Provider就是用静态工厂方法.

刚才说到Consumer需要用`ServiceLoader`来获取服务实现,具体过程如下.`ServiceLoader.load(clazz)` 会返回一个ServiceLoader,ServiceLoader实现了Iteratable接口,可以迭代拿出所有的实现类 需要注意的是每次调用`ServiceLoader.load(clazz)`的时候，都会重新初始化全部的实现类并且返回,如果你的实现类初始化比较耗费资源，建议只调用一次`ServiceLoader.load(clazz)`并且保存结果，这样就不会每次都重新初始化实现类了.


    package com.hello;  
    import com.print.api.PrintService;  
    import java.util.ServiceLoader;  
    &nbsp;  
    public class Main {  
    public static void main(String args[]) {  
    Iterable<printservice> printServices = ServiceLoader.load(PrintService.class); //get all the service implemention!  
    for (PrintService printService : printServices) {  
    printService.print("Hello world");  
    }  
    }  
    }  



如果你没有提供任何服务实现,当你用`ServiceLoader.load(clazz)`的时候也不会报错，但是迭代器会是空的,一般可以在api里面提供一个default的服务实现,当你有多个服务实现可以通过在在服务类里加 一些方法来区分多个服务实现(如getName()等)

[1]: http://www.java9countdown.xyz/
[2]: /img/jdk-module.png
[3]: /img/hello_module_info.PNG
[4]: /img/hello_module_info_2.PNG
[5]: /img/implied_readabliity.PNG
[6]: https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)
[7]: /img/module_services.PNG
[8]: /img/Service_arch.PNG
