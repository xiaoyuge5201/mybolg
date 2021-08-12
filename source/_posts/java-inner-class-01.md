---
title: Java内部类初始化
translate_title: java-internal-class-initialization
tags: java
categories: java
comments: false
date: 2021-06-01 00:00:00
---
### 1. 在同个java文件中，但不是内部类
```java
public class C {
}
//在同一个Java文件中只能存在一个public类，除内部类外
//只允许使用“public”、“abstract”和“final”。
class D{
    
}
```
```java
//实例化
public static void main(String[] args) {
    D d = new D();
}
```

### 2. 常规内部类
要实例化内部类对象，必须先有外部类对象，通过外部类对象.new 内部类();来实例化内部类对象，在其他文件或者其他包内都是这样，只是要能在其他包实例化的话，内部类Inner还得加上修饰符public。
```java
public class Outter {
    class Inner { }

    public static void main(String[] args) {
        Outter out = new Outter();
        Outter.Inner in = out.new Inner();
    }
}

//第二种情况：通过提供方法来获取实例对象
public class A {
    public class B{
        public void test(){
            System.out.println(111);
        }
    }

    public B getInstance(){
        return new B();
    }
    public static void main(String[] args) {
        A a = new A();
        B b = a.getInstance();
        b.test();
    }
}
```
### 3. 静态内部类
实例化静态内部类和实例化常规内部类有类似的地方，而不同之处在与静态内部类由于是静态的，所以不需要外部类对象就可以实例化，如上例Outter.Inner in = new Outter.Inner();
在其他Java文件也是这么实例化的
```java
class Outter {
    
    static class Inner {}
}

public class TestDemo {
    public static void main(String[] args) {
        Outter.Inner in = new Outter.Inner();
    }
}
```

### 4. 局部内部类
局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内，所以只能在方法或者该作用域内实例化,局部内部类不能有访问说明符,因为它不是外围类的一部分,但是可以访问当前代码块的常量,以及此外围类的所有成员
```java
public class A {
    class B {

    }

    public void pint() {
        class C {
        }
        new C();
    }

    public void pint(boolean b) {
        if (b) {
            class D {
            }
            new D();
        }
    }
}
```

### 5. 匿名内部类
匿名内部类可以继承一个类或实现一个接口，这里的ClassOrInterfaceName是匿名内部类所继承的类名或实现的接口名。但匿名内部类不能同时实现一个接口和继承一个类，也不能实现多个接口。如果实现了一个接口，该类是Object类的直接子类，匿名类继承一个类或实现一个接口，不需要extends和implements关键字
```java
ArrayList<String> list = new ArrayList<String>() {{
        add("A");
        add("B");
        add("C");
}};

new Thread(
        new Runnable() {
            public void run() { ... }
        }
).start();
```


