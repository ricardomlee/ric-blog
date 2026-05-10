---
description: ''

pubDatetime: 2020-03-23 21:50:41+08:00
tags:
- 笔记
- unraid
title: Linux迷惑行为大赏——创建硬链接后inode居然不同？
---

最近联盟站免，又下了一堆种，挂种的时候发现有的种子红了，原来是tmm刮削电影nfo的时候覆盖了种子里原来的nfo文件，导致种子校验失败。电影越下越多，文件夹也越来越复杂，有没有什么办法既可以正常挂种又能拯救杂乱的电影文件还能完美刮削电影信息呢？贴吧网友提供了一条思路——硬链接。

关于硬链接的内容，这里直接引用鸟哥的Linux私房菜里的相关内容

>Hard Link (实体链接, 硬式连结或实际连结)
在前一小节当中，我们知道几件重要的信息，包括：
每个文件都会占用一个 inode ，文件内容由 inode 的记录来指向；
想要读取该文件，必须要经过目录记录的文件名来指向到正确的 inode 号码才能读取。
也就是说，其实文件名只与目录有关，但是文件内容则与 inode 有关。那么想一想， 有没有可能有多个档名对应到同一个 inode 号码呢？有的！那就是 hard link 的由来。 所以简单的说：hard link 只是在某个目录下新增一笔档名链接到某 inode 号码的关连记录而已。
举个例子来说，假设我系统有个 /root/crontab 他是 /etc/crontab 的实体链接，也就是说这两个档名连结到同一个 inode ， 自然这两个文件名的所有相关信息都会一模一样(除了文件名之外)。实际的情况可以如下所示：
```shell
[root@www ~]# ln /etc/crontab .   <==创建实体链接的命令
[root@www ~]# ll -i /etc/crontab /root/crontab
1912701 -rw-r--r-- 2 root root 255 Jan  6  2007 /etc/crontab
1912701 -rw-r--r-- 2 root root 255 Jan  6  2007 /root/crontab
```
>你可以发现两个档名都连结到 1912701 这个 inode 号码，所以您瞧瞧，是否文件的权限/属性完全一样呢？ 因为这两个『档名』其实是一模一样的『文件』啦！而且你也会发现第二个字段由原本的 1 变成 2 了！ 那个字段称为『连结』，这个字段的意义为：『有多少个档名链接到这个 inode 号码』的意思。 如果将读取到正确数据的方式画成示意图，就类似如下画面：
<!-- Image missing: hard_link1.gif -->
上图的意思是，你可以透过 1 或 2 的目录之 inode 指定的 block 找到两个不同的档名，而不管使用哪个档名均可以指到 real 那个 inode 去读取到最终数据！那这样有什么好处呢？最大的好处就是『安全』！如同上图中， 如果你将任何一个『档名』删除，其实 inode 与 block 都还是存在的！ 此时你可以透过另一个『档名』来读取到正确的文件数据喔！此外，不论你使用哪个『档名』来编辑， 最终的结果都会写入到相同的 inode 与 block 中，因此均能进行数据的修改哩！
一般来说，使用 hard link 配置链接文件时，磁盘的空间与 inode 的数目都不会改变！ 我们还是由图来看，由图中可以知道， hard link 只是在某个目录下的 block 多写入一个关连数据而已，既不会添加 inode 也不会耗用 block 数量哩！
>>Tips:
hard link 的制作中，其实还是可能会改变系统的 block 的，那就是当你新增这笔数据却刚好将目录的 block 填满时，就可能会新加一个 block 来记录文件名关连性，而导致磁盘空间的变化！不过，一般 hard link 所用掉的关连数据量很小，所以通常不会改变 inode 与磁盘空间的大小喔！

>由图其实我们也能够知道，事实上 hard link 应该仅能在单一文件系统中进行的，应该是不能够跨文件系统才对！ 因为图就是在同一个 filesystem 上嘛！所以 hard link 是有限制的：
不能跨 Filesystem；
不能 link 目录。
不能跨 Filesystem 还好理解，那不能 hard link 到目录又是怎么回事呢？这是因为如果使用 hard link 链接到目录时， 链接的数据需要连同被链接目录底下的所有数据都创建链接，举例来说，如果你要将 /etc 使用实体链接创建一个 /etc_hd 的目录时，那么在 /etc_hd 底下的所有档名同时都与 /etc 底下的檔名要创建 hard link 的，而不是仅连结到 /etc_hd 与 /etc 而已。 并且，未来如果需要在 /etc_hd 底下创建新文件时，连带的， /etc 底下的数据又得要创建一次 hard link ，因此造成环境相当大的复杂度。 所以啰，目前 hard link 对于目录暂时还是不支持的啊！

简言之，如果把文件名看作是读写文件数据的“入口”，使用硬链接相当于给文件添加了不同的“入口”，在我的nas中实现的效果就是同一个电影文件被分别存放在了两个地方，既可以正常保种也可以整理在媒体库中。

然而实际操作的时候迷惑了
打开unraid的terminal，进入Downloads文件夹
`cd /mnt/user/Downloads`
尝试为6.txt创建硬链接6h
`ln 6.txt 6h`
查看一下inode吧
<!-- Image missing: GkCJtIoUBLj1YNu.png -->
？？？
inode不一样？发生了什么？

经过一番Google，在unraid官方论坛找到了作者的解释
<!-- Image missing: iIT2DpmkNHQ5SqO.png -->
<!-- Image missing: t9A3XC2bqvBhxmc.png -->

unraid里的share使用了shfs FUSE文件系统，作者在添加硬链接支持的时候产生了一系列bug。。。

在 /mnt/disk1 下面插看inode是一致的，电影文件可以创建硬链接
<!-- Image missing: gnMhLlzuPOtw6a3.png -->
<!-- Image missing: io24ns1kpdV5KMq.png -->

如果找不到批量添加硬链接的方法只能等有空再手动 ln 了。。