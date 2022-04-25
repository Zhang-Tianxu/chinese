---
title: ssh免密登录
date: 2022-02-10 16:15:29
tags:
  - ssh
categories:
  - 开发工具
---

# ssh 免密码登陆
在配置分布式系统时经常要实现系统内机器之间的ssh免密登陆，在按照网上某些教程实现时发现并非完全正确（尤其是你不是管理员时），现在把我能成功实现的方法列出：
## 实现方法
1. 在用户机上生成密钥

   ```shell
   ssh-keygen -t rsa
   ```

2. 将生成的密钥的公钥复制到目标机的`.ssh/authorized_keys`内

   ```shell
   cat .ssh
