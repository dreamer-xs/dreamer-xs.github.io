---
layout: post
title:  "docker之voluoe使用心得"
date:   2017-06-19 16:02:01 +0800
categories: docker
tag: docker
---

* content
{:toc}




# voluoe使用心得
1.当dockerfile 中指定VOLUME /tmp 时，将容器中的/tmp目录持久化到主机上。

2.当docker run -v /tmp:/tmp 时， 在容器中使用主机上的某个目录， 明确地告诉Docker **使用指定的主机路径来代替Docker自己创建的根路径并挂载到容器内指定的路径** ，在Docker术语中，这通常被称为bind-mounts， 如果主机上的路径不存在，目录将自动在给定的路径中创建。主机上/tmp目录持久化到容器中/tmp下。与1中相反。

3.当docker run -v /tmp 时，与1相同，**将容器中的/tmp目录持久化到主机上**。

4.当1和2同时指定时，还是2。也就是说2的优先级大于1

**注意**:***`显式指定volume时，需要在容器启动后才能将数据落盘，启动之前容器的挂载目录会被主机所覆盖掉。`***

# 容器数据持久化方案
1.启动数据容器， 指定主机路径/tmp目录来代替data容器下的/tmp目录。
```
     docker run -v /tmp:/tmp  --name data busybox:latest
```
2.启动应用容器， 从一个容器访问另一个容器的volumes中的/tmp目录。
```
     docker run --volumes-from data  server:latest
```

即可将应用容器中的/tmp目录数据持久化本地的/tmp目录下。

```
 docker inspect  leanote

"Mounts": [
            {
                "Type": "bind",        --将主机上目录持久化到容器中下
                "Source": "/opt/svicloud/tools/leanote/conf",
                "Destination": "/usr/local/bin/leanote/conf",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",     --说明是将容器中的目录持久化到主机上
                "Name": "9796f9f0ebd9327adfb90c88bc4656353591e67b99032c22c03d0440bec53958",
                "Source": "/var/lib/docker/volumes/9796f9f0ebd9327adfb90c88bc4656353591e67b99032c22c03d0440bec53958/_data",
                "Destination": "/usr/local/bin/leanote/public",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        
``` 

************************



