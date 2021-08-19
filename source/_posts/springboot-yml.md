---
title: springboot读取yml配置的方式
tags: springboot
categories:
  - 微服务
  - springboot
comments: false
translate_title: how-springboot-reads-yml-configuration
date: 2021-08-18 21:41:24
---
springboot项目中默认的配置文件是application.properties；
### 1.yml文件规则
- 树状结构，结构清晰
- 不支持tab缩进
- 可以使用"_"或"-"消协字母代替大写字母；如userName 与user-name， user_name含义是一样的（宽松绑定原则 relaxed binding）; key: value格式书写，value前面有个空格

###2. 数据格式
- 普通的值（数字，字符串，布尔）如：
    ```yaml
    port: 123      
    name: abc      
    flag: true
    ```
    字符串默认不用加上单引号或者双引号；
  
    ""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思,name: "zhangsan \n lisi"：输出；zhangsan 换行 lisi
  
    ''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据,name: ‘zhangsan \n lisi’：输出；zhangsan \n lisi
- 对象、Map(属性和值)如：
    ```yaml
    #k: v：在下一行来写对象的属性和值的关系；注意缩进(不支持tab,使用空格)
    server:
      port: 8123
      tomcat:
        uri-encoding: utf-8
      servlet:
        context-path: /app
    ```
  
- 数组（list， set）
    ```yaml
    #用- 值表示数组中的一个元素
    hands:
        - left
        - right
    ```
  
### 3. 读取方式
1. @Value注解
    ```yaml
    server:
      port: 8081
    ```
    ```text
    @Value("${server.port}")
    public String port;
    ```
    此处的port所在的类需要是一个组件,如果是实体类需要加上@Component
   

2. @ConfigurationProperties
   
   需要一个JavaBean 来专门映射配置的话,我们一般会使用@ConfigurationProperties来读取.
   
   使用的使用需要@EnableConfigurationProperties注解让类被springboot扫描到；
    ```yaml
    spring:
      datasource:
        druid:
          url: jdbc:mysql://localhost:3307/app?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT%2B8&useLegacyDatetimeCode=false
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password: root
    ```
    ```java
    //prefix 指定前缀
    @ConfigurationProperties(prefix = "spring.datasource")
    public class MyDataSourceProperties {
    
        private String type;
        private String driverClassName;
        private String url;
        private String username;
        private String password;
        //省略getter setter方法
    }
    ```
   - 前缀定义了哪些外部属性将绑定到类的字段上
   - 根据 Spring Boot 宽松的绑定规则，类的属性名称必须与外部属性的名称匹配
   - 我们可以简单地用一个值初始化一个字段来定义一个默认值
   - 类本身可以是包私有的
   - 类的字段必须有公共 setter 方法
    

   
3. Environment
   
   Spring Environment bean
    ```yaml
    @RestController
    @RequestMapping("/test")
    public class TestC {
    
        @Autowired
        private Environment env;
    
        @RequestMapping(value = "index", method = RequestMethod.GET)
        public String index() {
            return "environment : "+ env.getProperty("spring.datasource.druid.url");
        }
    }
    ```