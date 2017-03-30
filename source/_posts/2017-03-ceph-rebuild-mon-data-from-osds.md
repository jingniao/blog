---
title: 从osd恢复mon数据
mathjax: false
date: 2017-03-30 21:15:34
tags: ['shell','ceph']
categories: ['linux']
---

# 背景介绍
之前手上的一个Ceph 0.94.9集群因为突然断电，三个MON都因为leveldb的问题而无法启动。



# 已有的解决方案
1. [从已死的集群中恢复rbd镜像-ceph官方](http://ceph.com/planet/ceph-recover-a-rbd-image-from-a-dead-cluster/)
2. [Ceph的Mon数据重新构建工具-张鹏](http://www.zphj1987.com/2016/09/20/Ceph%E7%9A%84Mon%E6%95%B0%E6%8D%AE%E9%87%8D%E6%96%B0%E6%9E%84%E5%BB%BA%E5%B7%A5%E5%85%B7/) 或者   [官方的文档](http://docs.ceph.com/docs/hammer/rados/troubleshooting/troubleshooting-mon/#recovery-using-osds) 这两个是同一个方法

我这里选择第二种

<!-- more --> 

# 准备工作
`ceph-objectstore-tool`是从osd收集mon相关数据，这个工具从Hammer(0.94系列)应该就在Ceph中默认安装，并且可以使用，但是收集mon数据的参数`--op update-mon-db`却是在`0.94.10`版本中才添加进去的  
`ceph-monstore-tool`工具也是在`0.94.10`版本中才开始添加到`Hammer`中的，并且默认没有安装，需要手动安装`ceph-test`包这个工具才有，并且在`0.94.10`版本中这个工具是无法使用的，因为[这个bug](https://github.com/ceph/ceph/pull/13605/files)，到目前为止(`2017-03-30`),这个PR还没有合并到Hammer分支，我已经编译好了的`ceph-monstore-tool`工具，在[http://pan.baidu.com/s/1c1TvcL6](http://pan.baidu.com/s/1c1TvcL6) 密码：`jjic`

步骤跟张鹏教程基本上一致，只是有些参数可能不太一致

# 脚本
这个脚本是对官方的那个类似的脚本的补充，官方的那个脚本好像有点问题  
添加注释会让文章中的代码很难看，还有gist在这里：[gist代码](https://gist.github.com/jingniao/839c6fe3b635a90b0fcfafa19c1b1557)
```shell
MS=/tmp/mon-store
KEYRING=/etc/ceph/ceph.client.admin.keyring
FSID=12730f48-443f-4483-9231-545496c5973a
MONID=new-test-3
MONIP=192.168.213.133
HOSTS="new-test-1 new-test-2"
mkdir $MS
for host in $HOSTS; do
  rsync -avz $MS/ root@$host:$MS/
  rm -rf $MS
  ssh -t root@$host "for osd in /var/lib/ceph/osd/ceph-*; do    ceph-objectstore-tool --data-path \$osd --journal-path \$osd/journal --op update-mon-db --mon-store-path $MS; done"
  rsync -avz root@$host:$MS/ $MS/
done

ceph-authtool $KEYRING --import-keyring /var/lib/ceph/mon/ceph*/keyring
ceph-authtool $KEYRING -n client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *'
./ceph-monstore-tool $MS rebuild -- --keyring $KEYRING
# mkdir /var/lib/ceph/mon/ceph-$MONID 
cp -ra $MS/store.db /var/lib/ceph/mon/ceph-$MONID/
# touch /var/lib/ceph/mon/ceph-$MONID/done
# touch /var/lib/ceph/mon/ceph-$MONID/sysvinit
monmaptool --create --fsid $FSID --add $MONID $MONIP:6789  monmap
ceph-mon -i $MONID --inject-monmap monmap
service ceph start mon  
```
基本上就这些了