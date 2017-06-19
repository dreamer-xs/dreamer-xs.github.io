---
layout: post
title:  "docker中 CMD 与 ENTRYPOINT 的区别"
date:   2017-06-19 16:02:01 +0800
categories: docker
tag: docker
---

* content
{:toc}




- Dockerfile 用于自动化构建一个docker镜像。Dockerfile里有 CMD 与 ENTRYPOINT 两个功能咋看起来很相似的指令，开始的时候觉得两个互用没什么所谓，但其实并非如此：

CMD指令：
The main purpose of a CMD is to provide defaults for an executing container.
CMD在容器运行的时候提供一些命令及参数，用法如下：


	1. CMD ["executable","param1","param2"] (exec form, this is the preferred form) 
	2. CMD ["param1","param2"] (as default parameters to ENTRYPOINT) 
	3. CMD command param1 param2 (shell form) 


	* 第一种用法：运行一个可执行的文件并提供参数。
	* 第二种用法：为ENTRYPOINT指定参数。
	* 第三种用法(shell form)：是以”/bin/sh -c”的方法执行的命令。


如你指定:


	1. CMD [“/bin/echo”, “this is a echo test ”] 

build后运行(假设镜像名为ec):


	1. docker run ec 

就会输出: this is a echo test
是不是感觉很像开机启动项，你可以暂时这样理解。

注意点：
docker run命令如果指定了参数会把CMD里的参数覆盖： （这里说明一下，如：docker run -it ubuntu /bin/bash 命令的参数是指/bin/bash 而非 -it ,-it只是docker 的参数，而不是容器的参数，以下所说参数均如此。）
同样是上面的ec镜像启动：
docker run ec /bin/bash
就不会输出：this is a echo test，因为CMD命令被”/bin/bash”覆盖了。

ENTRYPOINT 　
字面意思是进入点，而它的功能也恰如其意。
An ENTRYPOINT allows you to configure a container that will run as an executable.它可以让你的容器功能表现得像一个可执行程序一样。
容器功能表现得像一个可执行程序一样，这是什么意思呢？
直接给个例子好说话：
例子一：
使用下面的ENTRYPOINT构造镜像：


	1. ENTRYPOINT ["/bin/echo"] 

那么docker build出来的镜像以后的容器功能就像一个/bin/echo程序：
比如我build出来的镜像名称叫imageecho，那么我可以这样用它：


	1. docker run -it imageecho “this is a test” 

这里就会输出”this is a test”这串字符，而这个imageecho镜像对应的容器表现出来的功能就像一个echo程序一样。 你添加的参数“this is a test”会添加到ENTRYPOINT后面，就成了这样　/bin/echo “this is a test” 。现在你应该明白进入点的意思了吧。

例子二：
ENTRYPOINT ["/bin/cat"]
构造出来的镜像你可以这样运行(假设名为st)：


	1. docker run -it st /etc/fstab 

这样相当： /bin/cat /etc/fstab 这个命令的作用。运行之后就输出/etc/fstab里的内容。

ENTRYPOINT有两种写法：
写法一：


	1. ENTRYPOINT ["executable", "param1", "param2"] (the preferred exec form) 

写法二：


	1. ENTRYPOINT command param1 param2 (shell form) 

你也可以在docker run 命令时使用–entrypoint指定（但是只能用写法一）。

下面是我把ENTRYPOINT设为[“/bin/sh -c”]时候运行的情况：


	1. linux-oj9e:/home/lfly/project/docker # docker run -it t2 /bin/bash 
	2. root@4c8549e7ce3e:/# ps 
	3. PID TTY TIME CMD 
	4. 1 ? 00:00:00 　sh 
	5. 9 ? 00:00:00 　bash 
	6. 19 ? 00:00:00 　ps 

可以看到PID为1的进程运行的是sh，而bash只是sh的一个子进程，/bin/bash只是作为 /bin/sh -c后面的参数。

CMD可以为ENTRYPOINT提供参数，ENTRYPOINT本身也可以包含参数，但是你可以把那些可能需要变动的参数写到CMD里而把那些不需要变动的参数写到ENTRYPOINT里面例如：


	1. FROM ubuntu:14.10  
	2. ENTRYPOINT ["top", "-b"]   
	3. CMD ["-c"]  

把可能需要变动的参数写到CMD里面。然后你可以在docker run里指定参数，这样CMD里的参数(这里是-c)就会被覆盖掉而ENTRYPOINT里的不被覆盖。

注意点１：
ENTRYPOINT有两种写法，第二种(shell form)会屏蔽掉docker run时后面加的命令和CMD里的参数。

注意点２：
网上有资料说ENTRYPOINT的默认值是[”/bin/sh -c”]，但是笔者在试验的时候得到的结果并不是这样的。
笔者使用ENTRYPOINT　[“/bin/sh -c”]　指令构造一个以/bin/sh -c为进入点的镜像，命名为sh，然后我可以这样运行：


	1. docker run -it sh “while(ture ) do echo loop; done” 

运行结果就是无限输出loop。但如果直接运行一个ubuntu:14.10镜像，情况不是这样的：


	1. docker run -it ubuntu:14.10 “while(ture ) do echo loop; done” 

得到这样的错误：


	1. linux-oj9e:/home/lfly # docker run -it ubuntu:14.10 “while(true) do echo this; done” 2014/11/16 18:07:53 Error response from daemon: Cannot start container 4bfe9c6faeec3ed465788a201a2f386cb1af35aba197dbc78b87c0d5dda1f88e: exec: “while(true) do echo this; done”: executable file not found in $PATH 

可以猜想默认情况下ENTRYPOINT并不是[“/bin/sh -c”]。
而且直接运行ubuntu:14.10列出程序也可以看到PID为1的程序并不是sh。所以更否定了网友的说法，ENTRYPOINT并不默认为[“/bin/sh -c”]　。
