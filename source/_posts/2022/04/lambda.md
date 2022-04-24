---
title: java8的lambda表达式语法
comments: true
tags: java
categories: java
translate_title: lambda
abbrlink: 51354
date: 2022-04-13 20:32:44
---
记录一下用到的一些java8的lambda表达式语法
### 1 list集合根据某个字段分组后求多个字段的和
```java
 List<SafeSystemVO> list = new ArrayList<>(16);
//....省略添加元素的代码
//分组字段 driver_id
list.stream().collect(Collectors.groupingBy(SafeSystemVO::getDriver_id)).values().stream().map(d -> {
    SafeSystemVO vo = d.get(0);
    //求和1
    vo.setAg_total(d.stream().map(s -> BigDecimal.valueOf(s.getAg_total())).reduce(BigDecimal.ZERO, BigDecimal::add).doubleValue());    
    // 求和2
    vo.setScore(d.stream().map(s -> BigDecimal.valueOf(s.getScore())).reduce(BigDecimal.ZERO, BigDecimal::add).doubleValue());  
    vo.setLkj_score(d.stream().map(s -> BigDecimal.valueOf(s.getLkj_score())).reduce(BigDecimal.ZERO, BigDecimal::add).doubleValue());  
    vo.setTotalScore(d.stream().map(s -> BigDecimal.valueOf(s.getTotalScore())).reduce(BigDecimal.ZERO, BigDecimal::add).doubleValue());    
    return vo;
}).collect(Collectors.toList());
```

### 2. list 根据某个字段分组后求单个字段的平均值，并按照分组字段排序
```java
List<SafeSystemVO> list = new ArrayList<>(16);
//....省略添加元素的代码
Map<String, Double> monthAvg = list1.stream().collect(Collectors.groupingBy(SafeSystemVO::getMonth, Collectors.averagingDouble(SafeSystemVO::getTotalScore)));
//  根据Map 对象的key排序
// 我的分组字段是日期，就用了下面的
monthAvg.entrySet().stream().sorted((o1, o2) -> {
    try {
        Date d1 = DateUtils.convertStringToDate(o1.getKey(), DateUtils.FM2);
        Date d2 = DateUtils.convertStringToDate(o2.getKey(), DateUtils.FM2);
        assert d1 != null;
        return d1.compareTo(d2);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return 0;
}).collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, (oldVal, newVal) -> oldVal, LinkedHashMap::new));

//根据Map 对象的value排序
//monthResult = workShopAvg.entrySet().stream().sorted((p1, p2) -> p2.getValue().compareTo(p1.getValue())).collect(Collectors.toList());

//方法一：
//list1.sort((o1, o2) -> o1.getAge().compareTo(o2.getTotalScore())); //正序
//list1.sort((o1, o2) -> o2.getAge().compareTo(o1.getTotalScore())); //倒序
//方法二
//list1.sort(Comparator.comparing(Person::getTotalScore)); // 正序
//list1.sort(Comparator.comparing(Person::getTotalScore).reversed()); // 倒序
```

### 3. list 根据字段分组求和后取 前/后10名
```java
//list对象接上面的

//根据driver_id分组，求平均值
Map<String, Double> driverScores = list3.stream()
        .collect(Collectors.groupingBy(SafeSystemVO::getDriver_id, Collectors.averagingDouble(SafeSystemVO::getTotalScore)));

//排序后获取后10 名， 前10名的话修改sorted逻辑为：sorted((p1, p2) -> p2.getValue().compareTo(p1.getValue()))
List<Map.Entry<String, Double>> driverScoresTop10 = driverScores.entrySet().stream().sorted((p1, p2) -> p1.getValue().compareTo(p2.getValue())).limit(10).collect(Collectors.toList());
```


### 4. 其他
```java
public static void main(String[] args) {
        //Id, name , age
        Person p1 = new Person(1,"麻子", 31);
        Person p2 = new Person(2,"李四", 20);
        Person p3 = new Person(3,"王五", 26);
        Person p4 = new Person(3,"王五", 26);

        List<Person> personList = new ArrayList<Person>();
        personList.add(p1);
        personList.add(p2);
        personList.add(p3);
        personList.add(p4);

        //java8遍历
        personList.forEach(p -> System.out.println(p.getAge()));
        //按照person的 age进行排序
        //方法一
        personList.sort((o1, o2) -> o1.getAge().compareTo(o2.getAge())); //正序
        personList.sort((o1, o2) -> o2.getAge().compareTo(o1.getAge())); //倒序
        //方法二
        personList.sort(Comparator.comparing(Person::getAge)); // 正序
        personList.sort(Comparator.comparing(Person::getAge).reversed()); // 倒序
        //多个字段排序
        personList.sort(Comparator.comparing(User::getId).thenComparing(Person::getAge));
        //注：若选择排序字段为null值，正序可personList.sort(Comparator.comparing(Person::getAge,Comparator.nullsFirst(Comparator.naturalOrder())))

        System.out.println("========================================");

        //获取年龄最大的Person
        Person maxAgePerson = personList.stream().max(Comparator.comparing(Person::getAge)).get();
        System.out.println(maxAgePerson.getAge());

        System.out.println("========================================");

        //获取年龄最小的Person
        Person minAgePerson = personList.stream().min(Comparator.comparing(Person::getAge)).get();
        System.out.println(minAgePerson.getAge());

        //过滤出年龄是20的person，想过滤出什么条件的均可以
        List<Person> personList1 = personList.stream().filter(person -> person.getAge() == 20).collect(Collectors.toList());

        //过滤-- 统计出年龄等于20的个数
        long count = personList.stream().filter(person -> person.getAge() == 20).count();

        //过滤出年龄大约20的人
        List<Person> personList2 = personList.stream().filter(t -> t.getAge().equals(20)).collect(Collectors.toList());
        //得到年龄的平均值
        double asDouble = personList.stream().mapToInt(person -> person.getAge()).average().getAsDouble();

        //得到年龄的求和--基本类型
        int sum = personList.stream().mapToInt(person -> person.getAge()).sum();
        
        //得到年龄的求和--包装类型,其中，若bigDecimal对象为null，可filter()过滤掉空指针.
        BigDecimal totalAge = personList.stream().map(User::getAge).reduce(BigDecimal.ZERO, BigDecimal::add);

        （其中，若bigDecimal对象为null，可filter()过滤掉空指针.）
        
        //去重
        List<Person> personList3 = personList.stream().distinct().collect(Collectors.toList());

        //list转map.
        //（其中，若集合对象key有重，可根据(k1,k2)->k1设置<保留k1，舍弃k2>.）
        Map<Long, Person> personMap = personList.stream().collect(Collectors.toMap(User::getId, t -> t,(k1,k2)->k1));
        
    }
}
```