---
layout: post
title: Ubuntu修改用户名
date: 2021-06-07
category: tips
---

1. 使用root账户登录
2. vim /etc/passwd，找到旧用户名那行，仅将开头旧用户名改为新用户名
3. vim /etc/shadow，找到旧用户名那行，将旧用户名改为新用户名
4. vim /etc/group，将所有旧用户名改为新用户名
5. reboot
6. vim /etc/passed，找到原来旧用户名行，将后面的旧用户名也改为新用户名
7. cd /home，将旧用户名目录改为新用户名（mv 旧用户名 新用户名）
8. （可选）其他账户使用命令su -进入root账户，visudo修改新用户名的sudo权限
