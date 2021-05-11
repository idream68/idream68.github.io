---
title: 在linux和docker上使用selenium
date: 2021-05-10 21:21:26
categories:
  - selenium
tags:
  - selenium
  - linux selenium
  - docker selenium
---

1. vim /etc/yum.repos.d/google.repo
   \# 把下面的内容复制进去

   ```
   [google]
   name=Google-x86_64
   baseurl=http://dl.google.com/linux/rpm/stable/x86_64
   enabled=1
   gpgcheck=0
   gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
   ```

2. sudo yum makecache

3. sudo yum install google-chrome-stable -y

4. google-chrome --version

5. 下载chromedriver

   http://npm.taobao.org/mirrors/chromedriver/

   把二进制文件放入到miniconda 的bin目录下
   如果运行报错

   尝试添加chrome_options.add_argument('--no-sandbox')

docker 使用selenium，启动容器时需要添加：--privileged

docker run -itd --privileged spider-executor /bin/bash 