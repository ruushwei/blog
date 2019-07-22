---
title: mac下安装配置storm
description: mac下安装配置storm
date: 2017-08-02T12:54:24+02:00
tags: storm
categories: storm
---

装了一天多，才装好。。主要是jzmq装不好，把它依赖卸载重新安装就好了，记下来，免得以后还得再找。。笑脸



mac上安装storm ，就是安装四个东西，zookeeper, zeromq , jzmq , storm 

zookeeper

```
brew install zookeeper
```



或者自己去下载tar包解压，

修改配置文件，conf/zoo_sample.cfg

```
cp zoo_sample.cfg zoo.cfg             //  默认配置文件
```

<!--more-->

zeromq

```
brew install zeromq
cd zeromq-2.1.7
./configure
make
sudo make install 
```



注意加 sudo



jzmq

```
#install jzmq
git clone https://github.com/nathanmarz/jzmq.git
cd jzmq
./autogen.sh
./configure
make
sudo make install
```



注意加sudo

如果autoconf automake出问题，就全卸了重新安装一次。

brew uninstall 这两个

brew install 这两个



storm 

```
brew install storm
```

 

配置文件：

conf/storm.yaml     注意 ‘-’ 和 ‘:’ 后面都有一个空格

```
 storm.zookeeper.servers:
    - "localhost"
#storm.zookeeper.port:2181
 storm.local.dir: "/Users/lgrcyanny/Codelab/storm/apache-storm-1.1.0/storm-local"
#
nimbus.seeds: ["host1", "host2", "host3"]
#
 nimbus.host: "localhost"
 supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
```



启动



```
./zkServer.sh start 

./storm nimbus &

./storm supervisor &

./storm ui &
```



注意sudo



然后就可以看到如下所示：



![stormUi](https://ws2.sinaimg.cn/large/006tNc79ly1fi9zorhp5qj31kw0upqb3.jpg)







