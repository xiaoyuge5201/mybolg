---
title: Hexo博客加密
comments: true
abbrlink: 58929
date: 2022-05-05 09:28:20
tags: hexo
categories: hexo
theme: xray
password: 12333
---
### 1.概述
Hexo编写Markdown文章后生成的静态页面默认是公开不加密的，所有人都可以访问，如果希望某些文章需要访问者只有输入正确的密码后才能继续访问，则需要使用插件hexo-blog-encrypt
 

### 2.加密后的文章特性
- 一旦你输入了正确的密码, 它将会被存储在本地浏览器的localStorage中。再次访问，不需输入密码
- 支持按标签加密
- 所有的核心功能都是由原生的API所提供的。 在 Node.js中, 我们使用 Crypto。在浏览器中, 我们使用 Web Crypto API
- 所有的核心功能都是由原生的API所提供的。 在 Node.js中, 我们使用 Crypto。在浏览器中, 我们使用 Web Crypto API
- 广泛地使用 Promise 来进行异步操作, 以此确保线程不被堵塞
- 过时的浏览器将不能正常显示

### 3.安装encrypt插件
在博客目录下执行下面的指令安装encrypt
```shell
npm install --save hexo-blog-encrypt
```
安装完成后，再package.json文件的dependecies依赖中看到encrypt插件
```lombok.config
"dependencies": {
    "hexo-blog-encrypt": "^3.0.16",
}
```
### 4.快速入门
#### 4.1 加密文章设置(password属性)
将"password"字段添加到文章信息头部：
```text
---
title: 标题
categories:
- 站点
- WordPress
  tags:
- WordPress
  abbrlink: asdfajklj3
  date: 2021-05-07 12:23:11
  password: 1234
---
```
#### 4.2 执行hexo g&&hexo s后，便可在本地查看加密后的文章预览
 

### 5.高级设置
####5.1 博客根目录 _config.yml
```yaml
# Security
encrypt: # hexo-blog-encrypt
abstract: 文章已被加密, 请输入密码查看.
message: 您好, 这里需要密码.
tags:
- {name: tagName, password: 密码A}
- {name: tagName, password: 密码B}
  template: <div id="hexo-blog-encrypt" data-wpm="{{hbeWrongPassMessage}}" data-whm="{{hbeWrongHashMessage}}"><div class="hbe-input-container"><input type="password" id="hbePass" placeholder="{{hbeMessage}}" /><label>{{hbeMessage}}</label><div class="bottom-line"></div></div><script id="hbeData" type="hbeData" data-hmacdigest="{{hbeHmacDigest}}">{{hbeEncryptedData}}</script></div>
  wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
  wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
```

#### 5.2 修改文章信息头如下
```text
password: 1234
abstract: 文章已被加密，需要输入密码查看。
message: 您好，这里需要密码。
wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
```
#### 5.3 配置优先级

文章信息头 > _config.yml (站点根目录下的) > 默认配置

#### 5.4 修改hexo新建博客的模板
打开博客目录下的 scaffolds/post.md文件，编辑
```text
---
title: {{ title }}
date: {{ date }}
tags:
categories: 
comments: true
password: 123456
abstract: 文章已被加密，需要输入密码查看，如需查看请联系博主！
message: 您好，这里需要密码。
wrong_pass_message: 抱歉，这个密码看着不太对，请再试试。
wrong_hash_message: 抱歉，这个文章不能被纠正，不过您还是能看看解密后的内容。
---
```

### 6.设置Hexo博客加密主题
目前已有的加密主题如下，可以选择一个
- default
- blink
- shrink
- flip 
- up
- surge
- wave
- xray

编辑根路径下的_config.yml，如下
```yaml
# Security
encrypt: # hexo-blog-encrypt
  abstract: Here's something encrypted, password is required to continue reading.
  message: Hey, password is required here.
  tags:
    - {name: encryptAsDiary, password: passwordA}
    - {name: encryptAsTips, password: passwordB}
  theme: xray   #增加主题属性
  wrong_pass_message: Oh, this is an invalid password. Check and try again, please.
  wrong_hash_message: Oh, these decrypted content cannot be verified, but you can still have a look.
```

在博客中增加
```yaml
---
title: Theme test
date: 2019-12-21 11:54:07
tags:
    - A Tag should be encrypted
theme: xray   #主题名称
password: "hello"
---
```
至此，完毕！可以参考：https://github.com/D0n9X1n/hexo-blog-encrypt#encrypt-theme

