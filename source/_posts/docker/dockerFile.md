---
title: Dockerfile 说明
date: 2021-05-08 14:34:11
categories: docker
tags:
  - docker
  - Dockerfile
---

- FROM #基础镜像，一切从这里开始
- MAINTAINER #镜像作者姓名、邮箱
- RUN #镜像构建时需要运行的命令
- ADD #步骤，tomcat镜像，这个tomcat压缩包，添加内容
- WORKDIR #镜像工作的目录
- VOLUME #挂载卷的目录
- EXPOST #暴露端口位置
- CMD #指定这个容器启动时要运行的命令，只有最后一个命令会生效，可被替代
- ENTRYPOINT #指定这个容器启动时要运行的命令，可以追加命令
- ONBUILD #当构建一个被继承 Dockerfile 这个时候就会运行 ONBUILD 的指令，触发指令
- COPY #类似ADD，将我们文件拷贝到镜像中
- ENV #构建时设置环境变量

---

ADD和COPY区别：

- ADD会解压文件，COPY不会
- 正常情况下建议使用COPY