---
title: 在MAC系统下用DOCKER搭建SWOOLE开发环境
date: 2016-05-12 00:00:0
tags:
---

作为一款重新定义PHP的开源框架，Swoole让PHP可以应用于更多的场景。

对于一位PHP程序员来说，通过Swoole可以了解以往不曾接触过的编程方法。

众所周知，搭建开发环境其实是很麻烦的一件事，经常会遇到各种各样的问题。所以我用Docker搭建了一套Swoole环境，按照这篇教程，就可以非常简单得开始Swoole之旅。

首先是安装Docker，官网有详细的安装步骤：https://docs.docker.com/mac/step_one/

Mac用户参考上述网址即可。对于其他系统的用户来说，官网都有相应的方法。这里不再细说。

以下的步骤也都是基于Mac系统来进行。

安装完成后，在Launchpad里会看到Docker Quickstart Terminal这个名字的应用，点击后，会打开系统默认的终端，然后等一会儿，就可以看到鲸鱼的画像出现了。

要注意终端里的一条提示信息：

    docker is configured to use the default machine with IP 192.168.99.100
    
记住这个IP地址，我们访问容器时会用到。

为了验证Docker是否运行起来，在打开的终端里输入下面的命令：

    docker images
    
如果Docker没有正确运行，会看到下面的错误提示：

    Cannot connect to the Docker daemon. Is the docker daemon running on this host?
    
确保Docker正确运行后，输入下面的命令，获取swoole镜像：

    docker pull koolob/swoole-docker
    
这个镜像是基于php:5.6-cli的镜像构建，包括swoole的最新版本1.8.4，默认使用了下面的编译参数进行编译：

    --enable-async-redis 
    --enable-async-httpclient 
    --enable-openssl 
    --enable-jemalloc
    
swoole的编译参数说明：http://wiki.swoole.com/wiki/page/437.html

这个镜像的Dockerfile是开源的，有需要搭建自定义环境的朋友可以参考我的Dockerfile文件进行修改：Github

下载镜像后，就可以运行一个容器，进入Swoole环境了。

输入命令：

    docker run -t -i koolob/swoole-docker /bin/bash
    
之后会进入到容器内。

再次输入：

    php -r 'echo swoole_version()."\n";'
    
可以看到输出了1.8.4，也就是当前swoole的版本号。

这样我们就有了一个通过Docker构建的Swoole环境。

接下来，我们可以把代码和环境构建成一个镜像来运行。依然有个示例可以参考：

    git clone https://github.com/koolob/swoole-docker-example.git
    
下载下来后，切换到目录中，直接执行

    ./build.sh
    
然后浏览器访问地址：http://192.168.99.100:8080/ 就可以看到结果了，同时在终端中也能看到程序输出的内容。在终端里使用Ctrl+C就可以退出容器。

192.168.99.100这个IP就是上文提到的终端提示的IP

我们运行的是一个非常简单的HTTP服务，代码位于src文件夹下：

    $serv = new swoole_http_server("0.0.0.0", 8080);
    
    $serv->on('Request', function($request, $response) {
        var_dump($request->get);
        var_dump($request->post);
        var_dump($request->cookie);
        var_dump($request->files);
        var_dump($request->header);
        var_dump($request->server);
    
        $response->cookie("User", "Swoole");
        $response->header("X-Server", "Swoole");
        $response->end("<h1>Hello Swoole!</h1>");
    });
    
    $serv->start();
    
build.sh脚本做的事就是构建镜像，并启动一个容器。

新构建的镜像也很简单，基于koolob/swoole-docker，开放8080端口，将src目录中的内容复制到镜像内，并在容器运行时，执行php代码。

之后要做的事就是修改src中的代码，执行build.sh脚本，然后测试。

至此，我们的Swoole开发环境搭建完毕。