---
title: Java 8学习
tags: java
categories: 知识整理
comments: true
abbrlink: 7605
date: 2020-04-23 14:04:02
translate_title: java8-learning
---

## 1. Optional
Optional 类主要解决的问题是臭名昭著的空指针异常（NullPointerException)。
本质上，这是一个包含有可选值的包装类，这意味着 Optional 类既可以含有对象也可以为空
### 1.1. optional构造方式

- Optional.of(T)

    该方式的入参不能为null，否则会有NPE，在确定入参不为空时使用该方式。

- Optional.ofNullable(T)

    该方式的入参可以为null，当入参不确定为非null时使用。

- Optional.empty()

    这种方式是返回一个空Optional，等效Optional.ofNullable(null)

### 1.2. 如何正确的使用Optional

- 尽量避免使用的地方

    1. 避免使用Optional.isPresent()来检查实例是否存在，因为这种方式和null != obj没有区别，这样用就没什么意义了。

    2. 避免使用Optional.get()方式来获取实例对象，因为使用前需要使用Optional.isPresent()来检查实例是否存在，否则会出现NPE问题。

    3. 避免使用Optional作为类或者实例的属性，而应该在返回值中用来包装返回实例对象。

    4. 避免使用Optional作为方法的参数，原因同3。

- 正确使用方式

    1. 实例对象存在则返回，否则提供默认值或者通过方法来设置返回值，即使用orElse/orElseGet方式：

        ```java
        //存在则返回
        User king = new User(1, "king");
        Optional<User> userOpt = Optional.of(king);
        User user =  userOpt.orElse(null);
        System.out.println(user.getName());
        
        //不存在提供默认值
        User user2 = null;
        Optional<User> userOpt2 = Optional.ofNullable(user2);
        User user3 = userOpt2.orElse(unknown);
        System.out.println(user3.getName());
        
        //通过方法提供值
        User user4 = userOpt2.orElseGet(() -> new User(0, "DEFAULT")); 
        System.out.println(user4.getName())
            
         //不建议下面这种使用
        if(userOpt.isPresent()) {
            System.out.println(userOpt.get().getName());
        } else {
            //。。。
        }
        ```

    2. 使用ifPresent()来进行对象操作，存在则操作，否则不操作。

        ```java
        //实例存在则操作，否则不操作
        userOpt.ifPresent(u -> System.out.println(u.getName()));
        userOpt2.ifPresent(u -> System.out.println(u.getName()));
        ```

    3. 使用map/flatMap来获取关联数据

        ```java
        //使用map方法获取关联数据
        System.out.println(userOpt.map(u -> u.getName()).orElse("Unknown"));
        System.out.println(userOpt2.map(u -> u.getName()).orElse("Default"));
        //使用flatMap方法获取关联数据
        List<String> interests = new ArrayList<String>();
        interests.add("a");interests.add("b");interests.add("c");
        user.setInterests(interests);
        List<String> interests2 = Optional.of(user)
            .flatMap(u -> Optional.ofNullable(u.getInterests()))
            .orElse(Collections.emptyList());
        System.out.println(interests2.isEmpty());
        ```

        

### 1.3.Optional判断第三方接口

使用java8的optional可以减少很多的NPE，再也不用当心别人的接口返回值问题了，也不用满屏的if（a != null）这种判断，下面是使用过程中遇到的问题以及如何使用Optional解决。

#### 1.3.1. 接口返回参数问题

1. 在微服务中使用feign调用其他接口，总担心别人返回的参数是否符合标准
2. 参数符合标准后，然后再进行数据判断，先判断是否code为200，然后判断数据存不存在，这样冗余的代码就很多

这是我们期望的返回格式

```json
{
	"code": "200",
	"msg": "调用成功!",
	"data": []
}
```

```java
//模拟接口调用方法
Map<String,Object> map = serviceImpl.queryList();
//即使map为空也能正常返回，配合map直接映射数据值
return Optional.ofNullable(map).map(r-> r.get("data")).orElseGet(ArrayList:: new)
    
 //JSONObject 判断是否返回成功，如果成功返回200， 不成功返回400   
JSONObject jsonObject = service.updateDate();
Optional.ofNullable(jsonObject).map(r->r.getInteger("code")).orElse(400)
```

#### 1.3.2. 避免判断风暴

对象层层嵌套，为了逻辑严谨必须要进行空判断

```java
//对于一个对象里面嵌套对象，那么需要层层去判断非空
School school = null;
if(school != null){
    Clazz clazz = school.getClazz();
    if(clazz != null){
        Student student = clazz.getStudent();
        if(student != null){
            String name = student.getName();
            if(name == null || "".equals(name)){
                name = "学生的姓名为空";
            }
        }
    }
}
//使用Optional后
 String name = Optional.ofNullable(school)
                .map(School::getClazz)
                .map(Clazz::getStudent)
                .map(Student::getName)
                .orElse("学生的姓名为空");
```

## 2. Stream

```java
//找出某一个字段等于某个值的那一条数据
JaponicaRiceCheck1 streamCheck = listItemRice.stream()
.filter(o -> o.getSYS_PARENTID().equals(check.getSYS_ID())).findAny().orElse(null);
```

