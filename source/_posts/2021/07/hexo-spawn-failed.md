---
title: Hexo部署出现错误Error Spawn failed解决方式
date: 2021-08-15 18:28:42
tags: hexo
comments: false
translate_title: spawn-failed
---
部署过程中可能会出现错误:
```text
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: unable to access 'https://github.com/xiaoyuge5201/xiaoyuge5201.github.io.git/': The requested URL returned error: 403
FATAL {
  err: Error: Spawn failed
      at ChildProcess.<anonymous> (/Users/xiaoyuge/workspace/mybolg/node_modules/hexo-util/lib/spawn.js:51:21)
      at ChildProcess.emit (events.js:315:20)
      at Process.ChildProcess._handle.onexit (internal/child_process.js:277:12) {
    code: 128
  }
} Something's wrong. Maybe you can find the solution here: %s https://hexo.io/docs/troubleshooting.html
xiaoyuge@xiaoyugedeMacBook-Pro mybolg % hexo clean
```
####解决方式一：
```shell
##进入站点根目录
cd cd /Users/xiaoyuge/workspace/mybolg

##删除git提交内容文件夹
rm -rf .deploy_git/

##执行
git config --global core.autocrlf false

##最后
hexo clean && hexo g && hexo d
```
####解决方式二：
有可能是你的git repo配置地址不正确,可以将http方式变更为ssh方式（我的就是这个问题）
github在2021-08-13正式启用personal access token后，原来的用户名+密码方式部署会报错，需要采用最新的token登录方式进行部署 。
具体方式参考：https://cloud.tencent.com/developer/article/1861466
```text
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
```
查看_config.yml文件，
```yaml
deploy:
  type: git
  #repo:https://github.com/xiaoyuge5201/xiaoyuge5201.github.io.git  这是原来的路径，现在改成了下面这种
  repo: git@github.com:xiaoyuge5201/xiaoyuge5201.github.io.git
  branch: master
```