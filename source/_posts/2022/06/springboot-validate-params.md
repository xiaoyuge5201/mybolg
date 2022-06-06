---
title: SpringBoot参数校验
comments: false
theme: xray
tags: springboot
categories: java
abbrlink: 64757
date: 2022-06-06 11:02:20
---
## 1. 前言
在控制器类的方法里自己写校验逻辑代码也可以，只是不够优美，业界有更好的处理方法，主要有以下几种。
 
## 2. PathVariable校验
```java
/**
 * 使用正则表达式限制group 只能是a-zA-Z0-9_
 */
@GetMapping("/path/{group:[a-zA-Z0-9_]+}/{userid}")
public String path(@PathVariable("group") String group, @PathVariable("userid") Integer userid) {
    return group + ":" + userid;
}
```

当请求URI不满足正则表达式时，客户端将收到404错误码，不方便的是，不能通过捕获异常的方式，向前端返回统一的、自定义格式的响应参数

## 3. 方法参数校验
```java
@GetMapping("/validate")
@ResponseBody
public String validate1(
        @Size(min = 1, max = 10, message = "姓名长度必须为1到10") @RequestParam("name") String name,
        @Min(value = 10, message = "年龄最小为10") @Max(value = 100, message = "年龄最大为100") @RequestParam("age") Integer age) {
    return name + ":" + age;
}
```
使用@Size 、@Min、@Max等校验注解进行参数校验

如果前端传递的参数不满足规则，则跑出异常，上面代码中@size、@Min、@Max注解来源于validation-api包中。
更多注解参 [参数校验注解](#index1) 小节。

## 4.表单对象/VO对象校验
当参数是VO时，可以在VO类的属性上添加校验注解。
```java
public class User {
    @Size(min = 1,max = 10,message = "姓名长度必须为1到10")
    private String name;

    @NotEmpty
    private String firstName;

    @Min(value = 10,message = "年龄最小为10")@Max(value = 100,message = "年龄最大为100")
    private Integer age;

    @Future
    @JSONField(format="yyyy-MM-dd HH:mm:ss")
    private Date birth;
    //。。。
}
```
其中，@Future注解要求必须是相对当前时间来讲“未来的”某个时间。@Past表示过去的某个时间
```java
@PostMapping("/validate2")
@ResponseBody
public User validate2(@Valid @RequestBody User user){
    return user;
}
```
## 5.自定义校验规则

### 5.1 自定义注解校验
需要自定义一个注解类和一个校验类。
```java
import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER,ElementType.FIELD})
@Constraint(validatedBy = FlagValidatorClass.class)
public @interface FlagValidator {
    // flag的有效值，多个使用,隔开
    String values();

    // flag无效时的提示内容
    String message() default "flag必须是预定义的那几个值，不能随便写";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```
```java
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

public class FlagValidatorClass implements ConstraintValidator<FlagValidator,Object> {
    /**
     * FlagValidator注解规定的那些有效值
     */
    private String values;

    @Override
    public void initialize(FlagValidator flagValidator) {
        this.values = flagValidator.values();
    }

    /**
     * 用户输入的值，必须是FlagValidator注解规定的那些值其中之一。
     * 否则，校验不通过。
     * @param value 用户输入的值，如从前端传入的某个值
     */
    @Override
    public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext) {
        // 切割获取值
        String[] value_array = values.split(",");
        Boolean isFlag = false;

        for (int i = 0; i < value_array.length; i++){
            // 存在一致就跳出循环
            if (value_array[i] .equals(value)){
                isFlag = true; break;
            }
        }

        return isFlag;
    }
}
```
使用我们自定义的注解：
```java
public class User {
    // 前端传入的flag值必须是1或2或3，否则校验失败
    @FlagValidator(values = "1,2,3")
    private String flag ;
    //。。。
}
```
## 5.2 分组校验
```java
import org.hibernate.validator.constraints.Length;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;

public class Resume {
    public interface Default {
    }

    public interface Update {
    }

    @NotNull(message = "id不能为空", groups = Update.class)
    private Long id;

    @NotNull(message = "名字不能为空", groups = Default.class)
    @Length(min = 4, max = 10, message = "name 长度必须在 {min} - {max} 之间", groups = Default.class)
    private String name;

    @NotNull(message = "年龄不能为空", groups = Default.class)
    @Min(value = 18, message = "年龄不能小于18岁", groups = Default.class)
    private Integer age;
    //。。。
}

```
```java
/**
 * 使用Defaul分组进行验证
 * @param resume
 * @return
 */
@PostMapping("/validate5")
public String addUser(@Validated(value = Resume.Default.class) @RequestBody Resume resume) {
    return "validate5";
}

/**
 * 使用Default、Update分组进行验证
 * @param resume
 * @return
 */
@PutMapping("/validate6")
public String updateUser(@Validated(value = {Resume.Update.class, Resume.Default.class}) @RequestBody Resume resume) {
    return "validate6";
}
```
建立了两个分组，名称分别为Default、Update。POST方法提交时使用Defaut分组的校验规则，PUT方法提交时同时使用两个分组规则。

## 6. 异常拦截器
通过设置全局异常处理器，统一向前端返回校验失败信息
```java
import com.scj.springbootdemo.WebResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.CollectionUtils;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;
import java.util.List;
import java.util.Set;

/**
 * 全局异常处理器
 */
@ControllerAdvice
public class GlobalExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    /**
     * 用来处理bean validation异常
     * @param ex 异常信息
     * @return 结果
     */
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseBody
    public  WebResult resolveConstraintViolationException(ConstraintViolationException ex){
        WebResult errorWebResult = new WebResult(WebResult.FAILED);
        Set<ConstraintViolation<?>> constraintViolations = ex.getConstraintViolations();
        if(!CollectionUtils.isEmpty(constraintViolations)){
            StringBuilder msgBuilder = new StringBuilder();
            for(ConstraintViolation constraintViolation :constraintViolations){
                msgBuilder.append(constraintViolation.getMessage()).append(",");
            }
            String errorMessage = msgBuilder.toString();
            if(errorMessage.length()>1){
                errorMessage = errorMessage.substring(0,errorMessage.length()-1);
            }
            errorWebResult.setInfo(errorMessage);
            return errorWebResult;
        }
        errorWebResult.setInfo(ex.getMessage());
        return errorWebResult;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public WebResult resolveMethodArgumentNotValidException(MethodArgumentNotValidException ex){
        WebResult errorWebResult = new WebResult(WebResult.FAILED);
        List<ObjectError>  objectErrors = ex.getBindingResult().getAllErrors();
        if(!CollectionUtils.isEmpty(objectErrors)) {
            StringBuilder msgBuilder = new StringBuilder();
            for (ObjectError objectError : objectErrors) {
                msgBuilder.append(objectError.getDefaultMessage()).append(",");
            }
            String errorMessage = msgBuilder.toString();
            if (errorMessage.length() > 1) {
                errorMessage = errorMessage.substring(0, errorMessage.length() - 1);
            }
            errorWebResult.setInfo(errorMessage);
            return errorWebResult;
        }
        errorWebResult.setInfo(ex.getMessage());
        return errorWebResult;
    }
}
```
## 7.<span id="index1">参数校验注解</span>

Java中参数校验的注解来自三个方面，分别是
- javax.validation:validation-api，对应包javax.validation.constraints
- org.hibernate:hibernate-validator，对应包org.hibernate.validator.constraints
- org.springframework:spring-context，对应包org.springframework.validation

JSR 303 是Bean验证的规范 ，Hibernate Validator 是该规范的参考实现，它除了实现规范要求的注解外，还额外实现了一些注解。
### 7.1 validation-api中的注解：

|  配置项  |         说明        |     适用类型  |
| :----- | :---------------- | :---------- |
| @AssertFalse  |  限制必须是false  |  boolean Boolean：not null时才校验 |
| @AssertTrue	 |  限制必须是true |    boolean Boolean：not null时才校验   |
| @Max(value)	 |  限制必须为一个小于等于value指定值的整数，value是long型 |  byte/short/int/long/float/double及其对应的包装类；包装类对象not null时才校验  |
| @Min(value)	 |  限制必须为一个大于等于value指定值的整数，value是long型 |  byte/short/int/long/float/double及其对应的包装类；包装类对象not null时才校验  |
| @DecimalMax(value)	 |  限制必须小于等于value指定的值，value是long型 |  byte/short/int/long/float/double及其对应的包装类；包装类对象not null时才校验  |
| @DecimalMin(value)	 |  限制必须大于等于value指定的值，value是字符串类型	 |  byte/short/int/long/float/double及其对应的包装类；包装类对象not null时才校验  |
| @Digits(integer, fraction)	 | 限制必须为一个小数（其实整数也可以），且整数部分的位数不能超过integer，小数部分的位数不能超过fraction。integer和fraction可以是0。	 |  byte/short/int/long/float/double及其对应的包装类；包装类对象not null时才校验  |
| @Null |  限制只能为null |   任意对象类型（比如基本数据类型对应的包装类、String、枚举类、自定义类等）；不能是8种基本数据类型  |
| @NotNull |  限制必须不为null |    任意类型（包括8种基本数据类型及其包装类、String、枚举类、自定义类等）；但是对于基本数据类型，没有意义  |
| @Size(min, max) |  限制Collection类型或String的长度必须在min到max之间，包含min和max |  1.Collection类型（List/Set）<br> 2.String  |
| @Pattern(regexp) |  限制必须符合regexp指定的正则表达式	 |  String  |
| @Future |  限制必须是一个将来的日期 |  Date/Calendar |
| @Past	 |  限制必须是一个过去的日期 |  Date/Calendar |
| @Valid |  校验任何非原子类型，标记一个对象，表示校验对象中被注解标记的对象（不支持分组功能）	 |  需要校验成员变量的对象，比如@ModelAttribute标记的接口入参  |

### 7.2 hibernate-validator中的注解：
下面列举的注解是hibernate-validator-5.3.6版本的。

|  配置项  |         说明        |     适用类型  |
| :----- | :---------------- | :---------- |
| @Length(min,max)  |  限制String类型长度必须在min和max之间，包含min和max	  |   String, not null时才校验 |
| @NotBlank	 |  验证注解的元素值不是空白（即不是null，且包含非空白字符）	 |    String   |
| @NotEmpty	 |  验证注解的元素值不为null且不为空（即字符串非null且长度不为0、集合类型大小不为0）|  1.Collection类型（List/Set）<br> 2.String  |
| @Range(min,max) |  验证注解的元素值在最小值和最大值之间 |  1. String(数字类型的字符串)，非null时才校验 <br>  2. byte/short/int/long/float/double及其包装类，包装类非null时才校验  |
| @Email(regexp) |  验证注解的字符串符合邮箱的正则表达式，可以使用regexp自定义正则表达式 | String |
| @CreditCardNumber |  验证银行借记卡、信用卡的卡号	 |  String（不能包含空格等特殊字符）|

### 7.3 spring-context中的注解：
|  配置项  |         说明        |     适用类型  |
| :----- | :---------------- | :---------- |
| @Validated	  |  校验非原子类型对象，或启用类中原子类型参数的校验（支持分组校验；只校验包含指定分组的注解参数）  |  1. Controller类  <br> 2. @ModelAttribue标记的查询条件对象类 <br>3. @RequestBody标记的请求体对象类 |
 

### 7.4 注解的启用
1. 方法中对象参数中成员变量校验注解的生效条件
    - @ModelAttribute标记的查询条件类参数，需要同时用@Valid或@Validated标记，类中的注解校验才会生效
    - @RequestBody标记的请求体对象参数，需要同时用@Valid或@Validated标记，类中的注解校验才会生效
    - @Valid或@Validated标记在方法或方法所属类上无效
2. 方法中原子类型参数校验注解的生效条件
    - @Validated标记在方法所属类上
3. 按照分组启用
    - 在注解中使用groups添加启用注解的分组
    - 在@Validated中指定启用的分组


博客参考地址：https://mp.weixin.qq.com/s/0VX6lLS133CA4NFCVOHhhw

