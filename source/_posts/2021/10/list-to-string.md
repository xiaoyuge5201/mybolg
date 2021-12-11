---
title: Java中将List列表转换为字符串
comments: false
tags: 集合
abbrlink: 28733
date: 2021-10-10 17:48:19
categories: Java
translate_title: list_to_string
---
### 1. toString() 方法
List.toString()是最简单的，但它在开头和结尾添加方括号，每个字符串用逗号分隔符分隔。
缺点是我们不能用另一个分隔符替换逗号，也不能去掉方括号
```java
public class ListToStringUsingToStringExample {

    public static void main(String args) {
        
    // creating a list with strings.
    List<String> list = Arrays.asList("One",
                      "Two",
                      "Three",
                      "Four",
                      "Five");
    
    // converting List<String> to String using toString() method
    String stringFromList = list.toString();
    
    // priting the string
    System.out.println("String : "+stringFromList);        
    }
}
// 输出：String : [One, Two, Three, Four, Five]
```

### 2. Java 8 String.join() 
java 8 String添加了一个特殊的方法String.join()以将集合转换为具有给定分隔符的字符串
```java
public class ListToStringUsingString_JoinExample {

    public static void main(String args) {
        // creating a list with strings.
        List<String> list = Arrays.asList("One",
                          "Two",
                          "Three",
                          "Four",
                          "Five");
        
        // converting List<String> to String using toString() method
        String stringFromList = String.join("~", list);
        
        // priting the string
        System.out.println("String with tilde delimiter: "+stringFromList);
        
        // delimiting with pipe | symbol.
        String stringPipe = String.join("|", list);
        
        // printing
        System.out.println("String with pipe delimiter : "+stringPipe);
    
    }
}
//输出：
//  String with tilde delimiter: One~Two~Three~Four~Five
//  String with pipe delimiter : One|Two|Three|Four|Five
```

### 3. Collectors.joining() 
Collectors.join()方法来自 java 8 stream api。Collctors.joining()方法将分隔符、前缀和后缀作为参数。此方法将列表转换为具有给定分隔符、前缀和后缀的字符串。

查看以下有关使用不同分隔符的 join() 方法的示例。但是，String.join() 方法不提供前缀和后缀选项。
```java
public class ListToStringUsingString_JoinExample {

    public static void main(String args) {
        // creating a list with strings.
        List<String> list = Arrays.asList("One",
                          "Two",
                          "Three",
                          "Four",
                          "Five");
    
        // using java 8 Collectors.joining with delimiter, prefix and suffix
        String joiningString = list.stream().collect(Collectors.joining("-", "{", "}"));
        // printing
        System.out.println("Collectors.joining string : "+joiningString);
        String joiningString3 = list.stream().collect(Collectors.joining("@", "", ""));
        // printing
        System.out.println("Collectors.joining string with @ separator : "+joiningString3);
    
    
    }
}
//输出：
//Collectors.joining string : {One-Two-Three-Four-Five}
//Collectors.joining string with @ separator : One@Two@Three@Four@Five
```
### 4. Apache Commons StringUtils.join() 
使用来自 apache commons 包的外部库。该库有一个方法StringUtils.join() ，它采用类似于 String.join() 方法的列表和分隔符
```java
public class ListToStringUsingStringUtils_JoinExample {

    public static void main(String args) {
        // creating a list with strings.
        List<String> list = Arrays.asList("One",
                          "Two",
                          "Three",
                          "Four",
                          "Five");
    
        // using java 8 Collectors.joining with delimiter, prefix and suffix
        String joiningString = StringUtils.join(list, "^");
        
        // printing
        System.out.println("StringUtils.join string with ^ delimiter : "+joiningString);
        
        String joiningString3 = StringUtils.join(list, "$");
        
        // printing
        System.out.println("StringUtils.join string with @ separator : "+joiningString3);
    }
}
//输出：
//  StringUtils.join string with ^ delimiter : One^Two^Three^Four^Five
//  StringUtils.join string with @ separator : One$Two$Three$Four$Five
```