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