---
title: "Lombok"
date: 2017-08-16T00:00:47+08:00
draft: false
tags: ["Java"]
author: "Peter Yang"
---

After Try [Kotlin][1],I am very impressed by this language.It's very consice and expressive,  and it's totally interoperable with Java.It provide a lot syntax sugar can help us write less do more But If you like to stay in Java,[Lombok][2] can provide some syntax sugar make life better
# Install
The lombok provide some keyword or annotation to make java easier to write,but it will generate the same class file as we write regular Java. So we not need lombok when run the application. For maven , make sure the scope is `provided` and gradle `compileOnly`(since v2.12)


    # For Maven  
    <dependency>  
    <groupid>org.projectlombok</groupid>  
    <artifactid>lombok</artifactid>  
    <version>1.16.18</version>  
    <scope>provided</scope>  
    </dependency>  
    //For Gradle  
    dependencies {  
    compileOnly 'org.projectlombok:lombok:1.16.18'  
    }  



And also need to install lombok plugins in your IDE so the IDE doesn't complain the syntax error
# Features
## val
same as kotlin, but it can only be used to declare the final local variables, and it also infer the type from the initialize expression
## @Cleanup
its very useful for closeable resource,its like try-with in Java7 at the end of the scope ,it will generate some code to safely close the resource
## @Setter
you can use @Getter/@Setter in class attribute, it will generate default get and set method and you can also config the access level etc.
## @ToString
generate the toString using all the field of the class. you can configure the `includeFieldNames`
## @Data
exactly like `data class` in Kotlin
## @Value
this is immutable variant of `@Data`, all the field are `private` and `final`
## @Builder
according [Builder pattern][3] produce builder APIs for your class,you can place on a class or on a constructor.

And there're other useful annotation or keyword,you MUST have a try

[1]: https://kotlinlang.org/
[2]: https://projectlombok.org/
[3]: https://en.wikipedia.org/wiki/Builder_pattern
