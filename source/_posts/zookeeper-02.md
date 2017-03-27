---
title: Zookeeper的配置
date: 2017-03-26 12:24:35
category: [zookeeper]
tags: [zookeeper]
---

前面两篇文章介绍了Zookeeper是什么和可以干什么，那么接下来我们就实际的接触一下Zookeeper这个东西，看看具体如何使用，有个大体的感受，后面再描述某些地方的时候也能在大脑中有具体的印象。本文只关注分布式模式的zookeeper，因为这也是在生产环境的唯一部署方式，单机的zookeeper可以在测试和开发环境使用，但是单机环境的zookeeper就不再是zookeeper了。

安装配置很简单，官网也有介绍，这里就只对后面的文章有提到的点说明下。

配置-zoo.cfg

这是zookeeper的主要配置文件，因为Zookeeper是一个集群服务，集群的每个节点都需要这个配置文件。为了避免出差错，zoo.cfg这个配置文件里没有跟特定节点相关的配置，所以每个节点上的这个zoo.cfg都是一模一样的配置。这样就非常便于管理了，比如我们可以把这个文件提交到版本控制里管理起来。其实这给我们设计集群系统的时候也是个提示：集群系统一般有很多配置，应该尽量将通用的配置和特定每个服务的配置(比如服务标识)分离，这样通用的配置在不同服务之间copy就ok了。ok，下面来介绍一些配置点：

    clientPort=2181

client port，顾名思义，就是客户端连接zookeeper服务的端口。这是一个TCP port。

    dataDir=/data
    
    dataLogDir=/datalog

dataLogDir如果没提供的话使用的则是dataDir。zookeeper的持久化都存储在这两个目录里。dataLogDir里是放到的顺序日志(WAL)。而dataDir里放的是内存数据结构的snapshot，便于快速恢复。为了达到性能最大化，一般建议把dataDir和dataLogDir分到不同的磁盘上，这样就可以充分利用磁盘顺序写的特性。

下面是集群中服务的列表

    server.1=127.0.0.1:20881:30881
    server.2=127.0.0.1:20882:30882
    server.3=127.0.0.1:20883:30883

在上面的例子中，我把三个zookeeper服务放到同一台机器上。上面的配置中有两个TCP port。后面一个是用于Zookeeper选举用的，而前一个是Leader和Follower或Observer交换数据使用的。我们还注意到server.后面的数字。这个就是myid(关于myid是什么下一节会介绍)。

上面这几个是一些基本配置。

还有像 tickTime，这是个时间单位定量。比如tickTime=1000，这就表示在zookeeper里1 tick表示1000 ms，所有其他用到时间的地方都会用多少tick来表示。

比如 syncLimit = 2 就表示fowller与leader的心跳时间是2 tick。

maxClientCnxns -- 对于一个客户端的连接数限制，默认是60，这在大部分时候是足够了。但是在我们实际使用中发现，在测试环境经常超过这个数，经过调查发现有的团队将几十个应用全部部署到一台机器上，以方便测试，于是这个数字就超过了。

minSessionTimeout, maxSessionTimeout -- 一般，客户端连接zookeeper的时候，都会设置一个session timeout，如果超过这个时间client没有与zookeeper server有联系，则这个session会被设置为过期(如果这个session上有临时节点，则会被全部删除，这就是实现集群感知的基础，后面的文章会介绍这一点)。但是这个时间不是客户端可以无限制设置的，服务器可以设置这两个参数来限制客户端设置的范围。

autopurge.snapRetainCount，autopurge.purgeInterval -- 客户端在与zookeeper交互过程中会产生非常多的日志，而且zookeeper也会将内存中的数据作为snapshot保存下来，这些数据是不会被自动删除的，这样磁盘中这样的数据就会越来越多。不过可以通过这两个参数来设置，让zookeeper自动删除数据。autopurge.purgeInterval就是设置多少小时清理一次。而autopurge.snapRetainCount是设置保留多少个snapshot，之前的则删除。

不过如果你的集群是一个非常繁忙的集群，然后又碰上这个删除操作，可能会影响zookeeper集群的性能，所以一般会让这个过程在访问低谷的时候进行，但是遗憾的是zookeeper并没有设置在哪个时间点运行的设置，所以有的时候我们会禁用这个自动删除的功能，而在服务器上配置一个cron，然后在凌晨来干这件事。

以上就是zoo.cfg里的一些配置了。下面就来介绍myid。

配置-myid

在dataDir里会放置一个myid文件，里面就一个数字，用来唯一标识这个服务。这个id是很重要的，一定要保证整个集群中唯一。zookeeper会根据这个id来取出server.x上的配置。比如当前id为1，则对应着zoo.cfg里的server.1的配置。

2. 而且在后面我们介绍leader选举的时候，这个id的大小也是有意义的。

OK，上面就是配置的讲解了，现在我们可以启动zookeeper集群了。进入到bin目录，执行 ./zkServer.sh start即可。