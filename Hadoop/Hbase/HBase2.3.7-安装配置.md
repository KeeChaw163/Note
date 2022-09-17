# HBase安装

### 1.环境变量配置

将安装包上传至CentOS中，解压安装包到hadoop目录下

    tar -zxvf hbase-2.3.7-bin.tar.gz -C /usr/hadoop

配置环境变量

	vi /etc/profile

在末尾添加以下代码，保存退出

	export HBASE_HOME=/usr/hadoop/hbase-2.3.7
	export PATH=$HBASE_HOME/bin:$PATH

生效配置

	source /etc/profile

验证

```
hbase version
```

### 2.配置HBase

进入hbase的conf 文件夹中，编辑hbase-env.sh文件，修改Java路径

	export JAVA_HOME=/usr/java/jdk1.8.0_241

不使用hbase自带的zookeeper

	export HBASE_MANAGES_ZK=false

编辑hbase-site.xml文件

	<configuration>
	
	    <!-- 将HBase数据保存在HDFS目录中 -->
	    <property>
	        <name>hbase.rootdir</name>
	        <value>hdfs://master:8020/hbase</value>
	    </property>
	
	    <!-- 配置zookeeper地址，节点全部启用zookeeper，个数必须是奇数 -->
	    <property>
	        <name>hbase.zookeeper.quorum</name>
	        <value>master,slave1,slave2</value>
	    </property>
	
	    <!-- zookeeper数据目录  -->
	    <property>
	        <name>hbase.zookeeper.property.dataDir</name>
	        <value>/usr/hadoop/zookeeper-3.5.9/data</value>
	    </property>
	
	    <property>
	        <name>hbase.zookeeper.property.clientPort</name>
	        <value>2181</value>
	    </property>
	    
	    <!-- HBase是否是分布式环境 -->
	    <property>
	        <name>hbase.cluster.distributed</name>
	        <value>true</value>
	    </property>
	    
	  	<property>
	    	<name>hbase.tmp.dir</name>
	    	<value>./tmp</value>
	 	</property>
	      
	 </configuration>

修改文件regionservers，添加从节点的主机名

	slave1
	slave2

### 3.同步HBase配置文件

同步master结点的HBase配置文件，至slave1、slave2

	scp -r /usr/hadoop/hbase-2.3.7 slave1:/usr/hadoop 
	scp -r /usr/hadoop/hbase-2.3.7 slave2:/usr/hadoop 

分别配置slave1、slave2的环境变量

	scp /etc/profile slave1:/etc/
	scp /etc/profile slave2:/etc/

分别在slave1、slave2上生效配置

	source /etc/profile

### 4.启动HBase

分别启动zookeeper、hdfs、yarn

zkServer.sh start

start-dfs.sh

start-yarn.sh



运行HBase启动命令

	start-hbase.sh 

主节点

```
[root@master ~]# jps
5312 SecondaryNameNode
6225 HMaster
5113 NameNode
7210 Jps
1243 QuorumPeerMain
5468 ResourceManager
```

子节点

```
[root@slave1 ~]# jps
3666 Jps
1459 QuorumPeerMain
2836 DataNode
2948 NodeManager
2582 HRegionServer
```

进入HBase web管理页面

	http://master:16010/

### 5.进入HBase Shell

```
hbase shell
```

### BUGS TIPS：

1.表初始化失败

```
2021-11-19 22:32:47,701 ERROR [main] regionserver.HRegionServer: Failed construction RegionServer
java.io.IOException: Failed update hbase:meta table descriptor
```

使用hbase的用户创建目录，查看是否有权限

```
hadoop fs -mkdir -p /hbase
```

如果提示hadoop在安全模式中

```
hdfs dfsadmin -safemode leave
```

如果提示无操作权限，在hadoop的hdfs-site中添加

    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>

2.如出现以下情况，可能导致web管理页面无法打开


	pids/hbase-root-master.pid: 没有那个文件或目录

确定hdfs与hbase配置文件是否相同

> hbase-site.xml下的hbase.rootdir下面的value值，必须要和hadoop配置文件core-site.xml下的fs.defaultFS下的value值、ip和端口相同！

修改hbase的pid文件保存路径

打开conf目录下的hbase-env.sh，找到以下代码

	# export HBASE_PID_DIR=/var/hadoop/pids

修改为自己的路径

	export HBASE_PID_DIR=/usr/hadoop/hbase.xx/pids

3.HMaster无法启动时，hbase-site.xml中添加以下内容：

```
<property>
	<name>hbase.unsafe.stream.capability.enforce</name>
	<value>false</value>
    <description>
        Controls whether HBase will check for stream capabilities (hflush/hsync).
        Disable this if you intend to run on LocalFileSystem.
        WARNING: Doing so may expose you to additional risk of data loss!
    </description>
</property>
```

出处：

https://stackoverflow.com/questions/48709569/hbase-error-illegalstateexception-when-starting-master-hsync

官方解释：

https://github.com/apache/hbase/blob/master/hbase-procedure/src/test/resources/hbase-site.xml

4.显示端口已被占用时，先查看已占用，kill掉端口，重新启动

```
netstat -nultp
kill -9 端口号
```
