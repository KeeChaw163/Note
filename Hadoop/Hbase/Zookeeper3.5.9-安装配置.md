# Zookeeper安装配置

## 1. 环境配置

将安装包上传至CentOS中，解压到hadoop目录下并重命名

    tar -zxvf apache-zookeeper-3.5.9-bin.tar.gz -C /usr/hadoop
    mv apache-zookeeper-3.5.9-bin zookeeper-3.5.9

进入conf目录，修改Zookeeper配置文件

```
cp zoo_sample.cfg zoo.cfg 
```

```
tickTime=2000
clientPort=2181
initLimit=5
syncLimit=2
dataDir=/usr/hadoop/zookeeper-3.5.9/data

# 节点数为单数（投票机制）
server.0=master:2888:3888
server.1=slave1:2888:3888
server.2=slave2:2888:3888
```

发送 zoo.cfg 文件至slave1和slave2

```
scp -r /usr/hadoop/zookeeper-3.5.9 slave1:/usr/hadoop/zookeeper-3.5.9
scp -r /usr/hadoop/zookeeper-3.5.9 slave2:/usr/hadoop/zookeeper-3.5.9
```

分别在master、slave1、slave2节点上的 /zookeeper-3.5.7/data 路径下创建myid文件，按照各自server后面的数值进行填写，以master为例：

```
vi myid
0
```

配置环境变量

	vi /etc/profile

在末尾添加以下代码，保存退出

	export ZOOKEEPER_HOME=/usr/hadoop/zookeeper-3.5.9
	export PATH=$ZOOKEEPER_HOME/bin:$PATH

发送 /etc/profile 配置文件至其他ZooKeeper节点

```
scp -r /etc/profile slave1:/etc/profile
scp -r /etc/profile slave2:/etc/profile
```

分别在master、slave1、slave2上生效配置

	source /etc/profile

### 2. 启动Zookeeper

分别在master、slave1、slave2上启动Zookeeper

	zkServer.sh start

分别在节点上查看Zookeeper状态

```
[root@master ~]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/hadoop/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
```

```
[root@slave1 ~]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/hadoop/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

```
[root@slave2 ~]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/hadoop/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

