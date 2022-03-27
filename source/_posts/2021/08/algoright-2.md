---
title: 转换成小写字母
tags: algorithm
categories: 算法
comments: true
abbrlink: 4638
date: 2021-08-22 13:15:12
translate_title: algorithm-02
---
实现函数 ToLowerCase()，该函数接收一个字符串参数 str，并将该字符串中的大写字母转换成小写字母，之后返回新的字符串。
```text
示例 1：
输入: "Hello"
输出: "hello"

示例2：
输入: "LOVELY"
输出: "lovely"
```
### 方法一： ASCCII码
**解题思路：通过ascii码表操作字符串即可,a和A相差32；**
- a-z: 97 - 122
- A-Z: 65 - 90
- 0-9: 48 - 57
```java
public static String toLowerCase(String str) {
    if (str == null || str.length() == 0) {
        return str;
    }
    StringBuilder sb = new StringBuilder();
    for (char ch : str.toCharArray()) {
        // a-z：97-122  A-Z：65-90  0-9：48-57
        if (ch >= 'A' && ch <= 'Z') {
            sb.append((char)(ch + 32));
        } else {
            sb.append(ch);
        }
    }
    return sb.toString();
}
```
或者：
```java
public static String toLowerCase(String str) {
    if (str == null || str.length() == 0) {
        return str;
    }
    char[] ch = str.toCharArray();
    for (int i = 0; i < str.length(); i++) {
        if (ch[i] >= 'A' && ch[i] <= 'Z') {
            ch[i] += 32;
        }
    }
    return String.valueOf(ch);
}
```

### 方法二： 位运算
**解题思路：**
- 大写变小写、小写变大写：字符 ^= 32;
- 大写变小写、小写变小写：字符 |= 32;
- 大写变大写、小写变大写：字符 &= 33;

ASCII码表中大写的A是65，小写的a是97，它们的差是32
65 | 32 转为二进制（按8位来算）可以得到 0100 0001 | 0010 0000 = 0110 0001 = 97 = a
  
```java
public String toLowerCase(String str) {
    if (str == null || str.length() == 0) {
        return str;
    }
    char[] ch = str.toCharArray();
    for (int i = 0; i < str.length(); i++) {
        ch[i] |= 32;
    }
    return String.valueOf(ch);
}
```

