# hadoop-2.7.7-master
部署基于 Hadoop 2.7.7 版本的计算机集群。


1 设备要求

1.1 操作系统
•	本地 - Windows 10
•	虚拟机 - Ubuntu 16.04 

1.2 软件
•	Java 1.8.0_201
•	ssh server


2 本地文件系统的环境部署

2.1 创建组及用户
•	切换为 root 用户：
`$ su root`
•	增加名为 hadoop的组：
`$ addgroup hadoop`
•	增加名为 hadoop的用户：
`$ adduser --ingroup hadoop hadoop`

2.2 更改权限
•	在 /opt/ 目录下创建 hadoop 文件夹：
`$ mkdir /opt/hadoop`
•	将 hadoop 用户设为 hadoop 文件夹的拥有者，将文件夹权限设为 hadoop 用户可以读、写、执行：
```
$ chown -R hadoop /opt/hadoop
$ chmod -R 700 /opt/hadoop
```
•	切换至 hadoop 用户：
`$ su Hadoop`

2.3 安装 Java
因为 Hadoop 分布式文件系统是用 Java 语言编写的，所以要先安装 jdk。安装方法分为离线和在线两种方法，离线方法采用压缩包安装，在线方法使用 apt-get 命令安装。
（1）	离线条件下安装 Java
•	下载 jdk-8u201-linux-x64.tar.gz 压缩包，解压后将文件移至 /usr/lib/jvm/java-8-oracle 目录下：
```
$ tar –xzvf jdk-8u201-linux-x64.tar.gz
$ mv jdk-8u201-linux-x64/* /usr/lib/jvm/java-8-oracle
```

•	在 ~/.bashrc 文件中添加 Java环境变量并保存：
```
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=.:$JAVA_HOME/bin:$PATH
```

•	在当前 bash 环境中执行命令，之后查看 Java 版本，确定安装成功：
```
$ source ~/.bashrc
$ java version
```

（2）在线安装Java
•	用 apt-get 安装：
```
$ apt-get update 
$ add-apt-repository ppa:webupd8team/java 
$ apt-get update
$ apt install –y oracle-java8-installer
```

•	检查 Java 版本：
`$ java –version`

2.4 设置机器名称 
• 在 /etc/hostname 文件中更改机器名称：
`master`

• 在 /etc/hosts 文件中添加静态 IP 地址：
`xxx.xxx.xxx.xxx       master`

2.5 设置 ssh 免密登陆
```
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

3 单机模式的配置和运行

3.1 下载 hadoop 2.7.7 版
•	用 wget 命令下载 Hadoop 的镜像文件并解压缩：
```
$ wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz
$ tar –xzvf Hadoop-2.7.7.tar.gz
```

•	将解压后的文件夹移至 /opt/hadoop/ 目录下：
`$ cp hadoop-2.7.7 /opt/Hadoop/ `

•	在 HADOOP_HOME/etc/hadoop/ 目录下的 hadoop-env.sh 文件中修改 java 参数和 hadoop 参数，并保存文件：
```
export JAVA_HOME=/usr/lib/jvm/java-8-oracle/jre
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_HOME=/opt/hadoop/hadoop-2.7.7
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export CLASSPATH=$CLASSPATH:/usr/local/hadoop/lib/*:.
export HADOOP_OPTS="$HADOOP_OPTS -Djava.security.egd=file:/dev/../dev/urandom"
```

3.2 运行非分布单机模式
```
$ mkdir input
$ cp etc/hadoop/*.xml input
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```


4 伪分布式的配置和运行

4.1 属性配置
•	修改 $HADOOP_HOME/etc/hadoop 目录下的 core-site.xml 文件：
```
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:$HADOOP_HOME/tmp</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
</property>
<property>
    <name>io.file.buffer.size</name>
    	   <value>131072</value>
    </property>
</configuration>
```
•	修改 $HADOOP_HOME/etc/hadoop 目录下的 hdfs-site.xml 文件：
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:$HADOOP_HOME/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:$HADOOP_HOME/tmp/dfs/data</value>
</property>
<property>
 	   <name>dfs.block.size</name>
   	   <value>268435456</value>
</property>
    <property>
        <name>dfs.namenode.handler.count</name>
        <value>100</value>
    </property>
</configuration>
```

4.2 运行伪分布式模式
（1）格式化文件系统：
`$ bin/hdfs namenode -format`
（2）运行名称节点进程和数据节点进程：
`$ sbin/start-dfs.sh`
Hadoop 进程日志的输出结果写入 $HADOOP_LOG_DIR 路径（默认为  $HADOOP_HOME/logs）。
（3）可以在网页浏览 NameNode：http://localhost:50070/
（4）为执行 MapReduce 作业，在分布式文件系统中创建路径：
```
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/hadoop
```
（5）将输入文件复制到分布式文件系统：
`$ bin/hdfs dfs -put etc/hadoop input`
（6）运行一些现有的例子：
`$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar grep input output 'dfs[a-z.]+'`
（7）检查输出文件：
可以将输出文件从分布式文件系统拷贝到本地文件系统再检查：
```
$ bin/hdfs dfs -get output output
$ cat output/*
```
或者直接在分布式文件系统中查看：
`$ bin/hdfs dfs -cat output/*`
（8）完成之后，结束进程：
`$ sbin/stop-dfs.sh`


5 YARN模式的配置和运行

5.1 属性配置
•	将 $HADOOP_HOME/etc/hadoop/ 下的 mapred-site.xml.template 复制粘贴到 mapred-site.xml：
```
$ cd etc/hadoop
$ cp mapred-site.xml.template mapred-site.xml
```

•	修改 $HADOOP_HOME/etc/hadoop 目录下的 mapred-site.xml 文件：
```
<configuration>
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
</property>
<property>
   	   <name>mapreduce.map.memory.mb</name>
   	   <value>1536</value>
</property>
<property>
         <name>mapreduce.map.java.opts</name>
   	    <value>-Xmx1024M</value>
</property>
<property>
 	   <name>mapreduce.reduce.memory.mb</name>
   	   <value>3072</value>
</property>
<property>
    	   <name>mapreduce.reduce.java.opts</name>
         <value>-Xmx2560M</value>
</property>
<property>
    	   <name>mapreduce.task.io.sort.mb</name>
         <value>512</value>
</property>
<property>
<name>mapreduce.task.io.sort.factor</name>
         <value>100</value>
</property>
<property>
         <name>mapreduce.jobhistory.address</name>
         <value>master:10020</value>
</property>
<property>
         <name>mapreduce.jobhistory.webapp.address</name>
         <value>master:19888</value>
</property>
<property>
         <name>yarn.app.mapreduce.am.staging-dir</name>
         <value>/user/app</value>
</property>
<property>
         <name>mapred.child.java.opts</name>
         <value>-Djava.security.egd=file:/dev/../dev/urandom</value>
</property>
</configuration>
```

•	修改 $HADOOP_HOME/etc/hadoop 目录下的 yarn-site.xml 文件：
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.resourcemanager.hostname</name>
        <value>master</value>
</property>
<property>
        <name>yarn.resourcemanager.bind-host</name>
        <value>0.0.0.0</value>
</property>
<property>
        <name>yarn.nodemanager.bind-host</name>
        <value>0.0.0.0</value>
</property>
<property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
        <name>yarn.log-aggregation-enable</name>
    	   <value>true</value>
</property>
<property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>file:/opt/hadoop/hadoop-2.7.7/tmp/yarn/local</value>
</property>
<property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>file:/opt/hadoop/hadoop-2.7.7/tmp/yarn/log</value>
</property>
<property>
        <name>yarn.nodemanager.remote-app-log-dir</name>
    	   <value>hdfs://master:8020/var/log/hadoop-yarn/apps</value>
</property>
</configuration>
```

5.2 运行 YARN
（1）运行 ResourceManager 进程和 NodeManager 进程：
`$ sbin/start-yarn.sh`
（2）在网页浏览 ResourceManager，http://localhost:8088/
（3）运行一个 MapReduce 作业：
`$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar grep input output 'dfs[a-z.]+'`
（4）完成之后，结束进程：
`$ sbin/stop-yarn.sh`


6 完全分布式的配置和运行

6.1 集群的环境部署
• 在每台子机上重复上述第 3 节中的五个步骤

• 更改子机的名称，在 /etc/hostname文件中分别将其更名为 worker1，worker2，如有更多子机则依此类推。

• 将子机的 IP 地址和名称添加到主机的 /etc/hosts 文件中
```
xxx.xxx.xxx.xxx       worker1
xxx.xxx.xxx.xxx       worker2
```

• 将子机的名称写入主机的 $HADOOP_HOME/etc/hadoop/slaves 文件中
```
worker1
worker2
```

• 将主机的hadoop 文件发送到各个子机的本地文件系统
```
$ scp –r hadoop-2.7.7 worker1:/opt/hadoop
$ scp –r hadoop-2.7.7 worker2:/opt/hadoop
```

6.2 运行完全分布式
（1）在主机上格式化文件系统：
`$ bin/hdfs namenode -format`
（2）运行名称节点进程：
`$ sbin/start-dfs.sh`
（3）用 jps 命令在各台机器上查看状态：
```
master：NameNode, SecondaryNamenode
worker1：DataNode
worker2：DataNode
```
（4）完成之后，结束进程：
`$ sbin/stop-dfs.sh`

