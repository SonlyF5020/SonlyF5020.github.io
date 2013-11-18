---
layout: post
title: "install hadoop-2.2.0 in mac-os-10.8"
date: 2013-11-18 10:45
comments: true
categories: 
---
###在MAC-OS-10.8安装Hadoop-2.2.0

本周协同大师一起弄hadoop-2，期间碰到很多奇怪的问题，也看到很多愿意分享自己的安装经验的博客。正所谓一千个人眼里有一千个哈姆雷特，一千个安装Hadoop的人可能有一千个安装问题。首先对那些愿意将自己的安装经验及其细致的分享出来的有开源精神的前辈们致敬。
  
作为这一千个读者中的微不足道的一个，不敢保证您会按照我的这个安装指引一次就成功的安装完成Hadoop，只是希望给阅读者一个安装的正能量：  
1. Hadoop 是可以正常安装完成的，要对自己有信心。  
2. 你不是一个人在战斗，你的问题大家都碰到过。  

####开始  
如标题所言，本文会告诉你如何在mac-os-10.8这个系统上安装hadoop-2.2.0，如果你的系统跟这个不一样，或者你使用的hadoop版本跟这个不一样，请重新搜索别的指引博客。这种情况下，这个博客不会给你任何有用的信息，特别是在你作为一个和我一样的新手的时候，这是一个忠告。此外，本文作为一个起步版本，只会教会你搭建最基本的**单机版**hadoop，也就是说，namenode, secondary namenode, 唯一的一个datanode都是你正在玩的这一个电脑。有关多机的分布式集群搭建，在lz试验成功之后会奉上。 

####第一步  
下载Hadoop-2.2.0，地址是这个:<http://apache.fayea.com/apache-mirror/hadoop/common/hadoop-2.2.0/>。当然在apache上也可以有其他的连接找到这个2.2.0版本的下载地址。打开这个连接，你会看到大约5个链接，下载最大的那个hadoop-2.2.0.tar.gz，参考其他博客有提到这个hadoop-2.2.0.tar.gz是一个针对32位系统的编译过之后的版本，在64位系统上面可能会有些影响性能的东西发生。但是不要在意这些细节，更不推荐某些博客中提到的做法，他们是这样说的：  

下载hadoop-2.2.0-src.tar.gz，然后安装 **数个** 软件，最后编译这一部分源文件，生成一个适合于64位系统的类似hadoop-2.2.0.tar.gz解压之后的一堆东西，使用这一堆东西就可以了。

之所以不推荐他们的做法，是因为： 
 
1.按照他们的做法，你一定会觉得自己就像一个单挑roshan的一级warlock一样，前途渺茫，甚至会丧心病狂的“**打心眼里恨毒了hadoop**”。这个时候你应该意识到，是时候停止了。
  
2.他们所提供的做法，或许可以用，但是他们绝对没有足够详细的告诉你，那所谓的 **数个** 软件应该怎样的一个个的正确安装，怎么验证安装的正确性，以及安装的顺序，还有最重要的，安装的过程中会遇到什么样的问题，以及怎么处理  

3.最后也是最重要的一点，不这样搞也可以(是不是有点 **若不自宫，也可成功** 的感觉?)。  

所谓的不这样搞也可以成功，指的是不影响使用，但是你可能在命令行里看到这样一些东西：

``` 
WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform
``` 
目前，我们忽视掉这个WARN先，不要在意这些细节

####第二步
下载完了Hadoop-2.2.0 之后，下面是配置。一般而言第一步的配置是JAVA_HOME。有的博客里面会强调需要把这个etc/hadoop/hadoop-env.sh里面export JAVA_HOME给指定为一个绝对的java路径，但其实这个默认值：

```
export JAVA_HOME=${JAVA_HOME}
```
是可以工作的。当然你不放心的话，也可以给它弄成一个绝对的路径，比如这样：

```
export JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/1.6.0/Home
```
配置完了JAVA_HOME，下面是core-site.xml, mapred-core.xml , hdfs-site.xml 的三个配置。与hadoop-1不同的是，这些配置相关的文件想在不是放在conf目录下了，而是在etc/hadoop/目录下面。首先是core-site.xml，确保这个配置文件里面的`configuration`标签配置成这样：

```
<configuration>
	     <property>
                <name>hadoop.temp.dir</name>
                <value>/tmp</value>
        </property>
        <property>
                <name>fs.default.name</name>
                <value>hdfs://localhost:54310</value>
        </property>
</configuration>
```
这里你需要稍微记住这个叫做/tmp的目录，因为后面可能会有些问题跟这个缓存目录里面放的东西有关。这个目录里面会缓存一些切割之后的文件块，以及各个mapreduce的一些缓存的东西。如果你碰到任何问题准备重启hadoop了，不要犹豫这个/tmp/hadoop-hadoop/的目录里面是啥，删除了再试一试。fs.default.name这个配置的意思是指定namenode，显然这里是localhost，如果建立真正意义上分布式集群的话，这个localhost应该换成namenode的IP地址。

接着是hdfs-site.xml里面的配置，这里的配置很简单，只需要保证`configuration`标签配置成这样：

```
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
</configuration>
```
这个配置的意思是将dfs的备份数由默认的3改为1，因为这是一个单机版安装的例子，所以不想占用太多的磁盘空间的话，最好加上这个配置。

最后一个是mapred-site.xml中的配置，确保`configuration`标签配置成这个样子就行：

```
<configuration>
	<property>
		<name>mapred.job.tracker</name>
		<value>localhost:54311</value>
	</property>
</configuration>
```
如果你了解一点hadoop的mapreduce过程的话，你会知道这个是配置jobtracker的过程，你可以把jobtracker理解成hdfs中的namenode这种司令员的作用。如果你不了解的话，可以先无脑加上。

####第三步
至此，你已经做了很多配置上的事了，剩下的只是操作。但是最好的推荐是你在这里先等一等，让配置再飞一会。你需要做的应该是：确保你的配置无误  
首先你可以尝试启动hdfs，使用命令：

```
./bin/hdfs namenode -format
```
这是一个namenode 初始化的命令。初始化后，可以使用脚本start-dfs.sh来启动hdfs了，当然你可以不这个细致入微的一个个启动hadoop的各个模块，直接跑start-all.sh脚本就可以了。这样一来，你需要做的只是执行这个脚本：

```
./sbin/start-all.sh
```
在这个start-all.sh的脚本里面会逐步启动hadoop需要的各个组件。namenode,datanode,secondary namenode,yarn daemons,resource manager。正常的情况是，这五个东西成功启动之后，在dock中会看到有5个**茶杯+纸+笔**的图标。对的，是 **5** 个，如果你发现没有5个，比如只有4个，那么很可能后面是走不通的。这个时候，你有多重方法排除错误，第一种是重新运行

```
./bin/hdfs namenode -format
```
再尝试一次，第二种是**重启电脑**，简单粗暴，但是确实有效。等等，我觉得我的个人宣言应该改成**永远不要停止重启电脑**了。
如果start-all.sh跑的过程中，没有出现任何问题，那么很有可能你的hadoop是已经配置好了，是不是内心还有点小激动？ 

这个时候，你可以运行这个命令来查看集群状态了：

```
./bin/hdfs dfsadmin –report
```
需要注意的是，这个命令在hadoop-2.2.0里面是可以工作的，但是在更早一点的版本，比如2.0.5里面是没有的。在运行这个状态之后，好的情况是你看到的结果是这样的：

```
Datanodes available: 1 (1 total, 0 dead)

Live datanodes:
Name: 127.0.0.1:50010 (localhost)
Hostname: bogon
Decommission Status : Normal
Configured Capacity: 204000002048 (189.99 GB)
DFS Used: 4096 (4 KB)
Non DFS Used: 98520674304 (91.75 GB)
DFS Remaining: 105479323648 (98.24 GB)
DFS Used%: 0.00%
DFS Remaining%: 51.71%
```
坏的情况可能是这样的：

```
Configured Capacity: 0 (0 B)
Present Capacity: 0 (0 B)
DFS Remaining: 0 (0 B)
DFS Used: 0 (0 B)
DFS Used%: NaN%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
```
坏的情况还有很多种，就像图片尔斯泰的那句名言说的“**正常的Hadoop都是相似的，异常的Hadoop各有各的异常**”。事实上，lz在运行这个report命令的时候就马上看到了坏的情况，然后我默默的使用了我的宣言(**永远不要停止重启电脑**)，然后就出现了好的情况。

####第四步
是时候来一波进攻了! 输入这个命令跑一个简单的，不依赖文件的mapreduce任务：

```
./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar pi 20 20
```
这个任务是一个计算圆周率的任务，相对于wordcount命令的优势是，你不需要专门准备一个文件用于wordcount。你暂时不需要知道后面的两个20是什么意思。你只需要知道，如果这个命令执行完之后，你的命令行出现诸如3.1700000000000的东西，说明hadoop已经在正常执行mapreduce程序了(**这精度实在不敢恭维**)。同样你也可能看到各种各样的异常，不要灰心，如果之前的./bin/hdfs dfsadmin –report出现的是好的情况，而这个命令确产生了恶心的异常的话，推荐两种做法：
  
1.大刀向/tmp砍去，把hadoop-hadoop目录下的东西全部干掉，然后重新尝试。  
2.**永远不要停止重启电脑**

####附录A
常见问题处理办法：  
1.清空/tmp/hadoop-hadoop （成功率 10%）
2.ssh问题请单独考量，如不能解决请参阅附录B （成功率 10%）
3.下载新一个hadoop，从头再来一遍 （成功率 10%）
4.**永远不要停止重启电脑** （成功率 70%）

####附录B
如果附录A中的所有办法都没有办法解决你正在碰到的异常，lz想对你说三个字:"**请联系我!**":  
QQ: 502089948  
tel: 15520552070  
weibo: @草莓豆豆龙  
email: SonlyF5020@gmail.com 
skype: sonlyf5020

-- Warlock --


