---
title: hexo_报错问题解决
date: 2018-07-12 18:39:43
category: [技术总结]
tags: [hexo permission denied (publickey) 问题解决]
---

报错如下：
```
ATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (/Users/zhaowenlei/dev/hexo-blog/node_modules/hexo-deployer-git/node_modules/hexo-util/lib/spawn.js:37:17)
    at emitTwo (events.js:87:13)
    at ChildProcess.emit (events.js:172:7)
    at maybeClose (internal/child_process.js:827:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:211:5)
FATAL Permission denied (publickey).
fatal: Could not read from remote repository.

```

重新替换github上的秘钥也无济于事

rm -rf .deploy_git 也不行，但是还是执行了

最后吧_config.yml 中的repo
```
git@github.com:zhaowenlei/zhaowenlei.github.io.git
```
改为
```
https://github.com/zhaowenlei/zhaowenlei.github.io.git
```

sudo hexo deploy 居然成功了

只不过要输入github 账号和密码！！