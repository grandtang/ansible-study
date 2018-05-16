非HA的安装方式<br>
&nbsp;&nbsp;实例：
  ```
         server     安装软件                    运行的进程
         master      jdk,hadoop               ResourceManager,NameNode,DFSZKFailoverController
         master-01   jdk,hadoop               ResourceManager,NameNode,DFSZKFailoverController
         slave-01    jdk,hadoop,zookeeper     NodeManager,DataNode,QuorumPeerMain,JournalNode
         slave-02    jdk,hadoop,zookeeper     NodeManager,DataNode,QuorumPeerMain,JournalNode
         slave-03    jdk,hadoop,zookeeper     NodeManager,DataNode,QuorumPeerMain,JournalNode
  
  ```
 &nbsp;&nbsp;步骤一：安装jdk（root用户）
 ```
    1，下载 jdk 'jdk-8u151-linux-x64.tar.gz'
       解压到 /opt/java目录下
    2，配置环境变量
       创建 /etc/profile.d/jvm.sh 文件
       并且写入如下内容
       export JAVA_HOME=/opt/java/jdk1.8.0_151
       export PATH=$PATH:$JAVA_HOME/bin
    3，将 /opt/java 目录copy到各个服务器上
       将 /etc/profile.d/jvm.sh 文件copy到各个服务器上
    4，在各个服务器上执行 source /etc/profile
 ```
 
 &nbsp;&nbsp;步骤二：用户设置
 ```
    1，创建hadoop用户（root用户）
       useradd -m hadoop
       passwd hadoop
    2，创建bigdata组 并将bigdata组赋给hadoop用户
       groupadd bigdata
       usermod -a -G bigdata hadoop
    3，创建/data目录并且修改/data目录的组为bigdata（root用户）
       mkdir /data
       chgrp bigdata -R /data
```
 &nbsp;&nbsp;步骤三：安装hadoop（hadoop用户）
 ```
    1，解压hadoop包到/data/hadoop目录中（hadoop用户）
       mkdir /data/hadoop
       tar -zxvf hadoop-2.9.1.tar.gz -C /data/hadoop
    2，修改hadoop-env配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/hadoop-env.sh）
       修改变量 hadoop-env中的变量JAVA_HOME的值如下
       export JAVA_HOME=/opt/java/jdk1.8.0_
    3，修改core-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/core-site.xml）如下
       <configuration>
          <!-- 指定hdfs的nameservice为ns -->  
         <property>  
           <name>fs.defaultFS</name>  
           <value>hdfs://ns</value>
         </property>  
         <!-- 指定hadoop临时目录 -->  
         <property>  
           <name>hadoop.tmp.dir</name>  
           <value>{{hadoop_tmp_dir}}</value>  
         </property>  
         <!-- 指定zookeeper地址 -->  
         <property>  
           <name>ha.zookeeper.quorum</name>  
           <value>{{zk_01}}:2181,{{zk_02}}:2181,{{zk_03}}:2181</value>
         </property>  
       
       </configuration>
       
    4，修改hdfs-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/hdfs-site.xml）如下
        <configuration>
         <!--指定hdfs的nameservice为ns，需要和core-site.xml中的保持一致 -->  
            <property>  
                <name>dfs.nameservices</name>  
                <value>ns</value>  
            </property>  
            <!-- ns下面有两个NameNode，分别是nn1，nn2 -->  
            <property>  
                <name>dfs.ha.namenodes.ns</name>  
                <value>nn1,nn2</value>  
            </property>  
            <!-- nn1的RPC通信地址 -->  
            <property>  
                <name>dfs.namenode.rpc-address.ns.nn1</name>  
                <value>{{master}}:8020</value>
            </property>  
            <!-- nn1的http通信地址 -->  
            <property>  
                <name>dfs.namenode.http-address.ns.nn1</name>  
                <value>{{master}}:50070</value>  
            </property>  
            <!-- nn2的RPC通信地址 -->  
            <property>  
                <name>dfs.namenode.rpc-address.ns.nn2</name>  
                <value>{{master_01}}:8020</value>
            </property>  
            <!-- nn2的http通信地址 -->  
            <property>  
                <name>dfs.namenode.http-address.ns.nn2</name>  
                <value>{{master_01}}:50070</value>  
            </property>  
            <!-- 指定NameNode的元数据在JournalNode上的存放位置 -->  
            <property>  
                <name>dfs.namenode.shared.edits.dir</name>  
                <value>qjournal://{{slave_01}}:8485;{{slave_02}}:8485;{{slave_03}}:8485/ns</value>  
            </property>  
            <!-- 指定JournalNode在本地磁盘存放数据的位置 -->  
            <property>  
                <name>dfs.journalnode.edits.dir</name>  
                <value>{{dfs_jdata_dir}}</value>  
            </property>  
            <!-- 开启NameNode失败自动切换 -->  
            <property>  
                <name>dfs.ha.automatic-failover.enabled</name>  
                <value>true</value>  
            </property>  
            <!-- 配置失败自动切换实现方式 -->  
            <property>  
                <name>dfs.client.failover.proxy.provider.ns1</name>  
                <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>  
            </property>  
            <!-- 配置隔离机制方法，多个机制用换行分割，即每个机制暂用一行-->  
            <property>  
                <name>dfs.ha.fencing.methods</name>  
                <value>  
                    sshfence  
                    shell(/bin/true)  
                </value>  
            </property>  
            <!-- 使用sshfence隔离机制时需要ssh免登陆 -->  
            <property>  
                <name>dfs.ha.fencing.ssh.private-key-files</name>  
                <value>{{user_home}}/.ssh/id_rsa</value>  
            </property>  
            <!-- 配置sshfence隔离机制超时时间 -->  
            <property>  
                <name>dfs.ha.fencing.ssh.connect-timeout</name>  
                <value>30000</value>  
            </property>  
            
            <!-- 设置hdfs副本数量 -->
            <property>
                <name>dfs.replication</name>
                <value>{{dfs_replication}}</value>
            </property>
            <!-- 设置datanode存放的路径 -->
            <property>
                <name>dfs.datanode.data.dir</name>
                <value>{{dfs_datanode_data_dir}}</value>
            </property>
        
            <property>
                 <name>dfs.webhdfs.enabled</name>
                 <value>true</value>
            </property>
        </configuration>

    
    5，修改mapred-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/mapred-site.xml）如下
      <configuration>
          <!-- 通知框架MR使用YARN -->
          <property>
              <name>mapreduce.framework.name</name>
              <value>yarn</value>
          </property>
      
          <!-- job history server -->
          <!-- 
            2、启动history-server
              Hadoop启动jobhistoryserver来实现web查看作业的历史运行情况，由于在启动hdfs和Yarn进程之后，jobhistoryserver进程并没有启动，需要手动启动，
              启动的方法是通过(注意：必须是两个命令)：
              ./mr-jobhistory-daemon.sh start historyserver
              ./yarn-daemon.sh start timelineserver
           -->
           
          <property>  
                  <name>mapreduce.jobhistory.address</name>  
                  <value>{{master}}:10020</value>  
          </property>  
          <property>  
                  <name>mapreduce.jobhistory.webapp.address</name>  
                  <value>{{master}}:19888</value>  
          </property>  
          <property>  
                  <name>mapreduce.jobhistory.joblist.cache.size</name>  
                  <value>1000</value>  
                  <description>default 20000</description>  
          </property>  
          <property>  
                  <name>mapred.child.java.opts</name>  
                  <value>-Xmx512m</value>  
          </property>  
          <property>  
                  <name>mapreduce.jobhistory.cleaner.enable</name>  
                  <value>true</value>  
          </property>  
          <property>  
                  <name>mapreduce.jobhistory.cleaner.interval-ms</name>  
                  <value>86400000</value>  
                  <description>the job history cleaner checks for files to delete, in milliseconds. Default 86400000 (one day). Files are only deleted if they are older than</description>  
          </property>  
          <property>  
                  <name>mapreduce.jobhistory.max-age-ms</name>  
                  <value>432000000</value>  
                  <description>Job history files older than this many milliseconds will be deleted when the history cleaner runs. Defaults to 604800000 (1 week)</description>  
          </property>  
      </configuration>
    
    6，修改yarn-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/yarn-site.xml）如下
       <configuration>
       
            <!-- 开启resourcemanager高可用 -->  
           <property>  
              <name>yarn.resourcemanager.ha.enabled</name>  
              <value>true</value>  
           </property>  
           <!-- 指定resourcemanager的cluster id -->  
           <property>  
              <name>yarn.resourcemanager.cluster-id</name>  
              <value>yrc</value>  
           </property>  
           <!-- 指定resourcemanager的名字 -->  
           <property>  
              <name>yarn.resourcemanager.ha.rm-ids</name>  
              <value>rm1,rm2</value>  
           </property>  
           <!-- 分别指定RM的地址 -->  
           <property>  
              <name>yarn.resourcemanager.hostname.rm1</name>  
              <value>{{master}}</value>  
           </property>  
           <property>  
              <name>yarn.resourcemanager.hostname.rm2</name>  
              <value>{{master_01}}</value>  
           </property>  
           <!-- 指定zk集群地址 -->  
           <property>  
              <name>yarn.resourcemanager.zk-address</name>  
              <value>{{zk_01}}:2181,{{zk_01}}:2181,{{zk_01}}:2181</value>  
           </property>  
          
           <!-- reducer取数据的方式是mapreduce_shuffle -->
           <property>
               <name>yarn.nodemanager.aux-services</name>
               <value>mapreduce_shuffle</value>
           </property>
           <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
           </property>
       
           <!-- yarn 调度 -->
          <property>
            <name>yarn.resourcemanager.scheduler.address.rm1</name>
            <value>{{master}}:18030</value>
          </property>
          <!-- yarn webapp -->
          <property>
            <name>yarn.resourcemanager.webapp.address.rm1</name>
            <value>{{master}}:18088</value>
          </property>
          <!-- yarn 资源同步 -->
          <property>
            <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
            <value>{{master}}:18025</value>
          </property>
          <property>
            <name>yarn.resourcemanager.admin.address.rm1</name>
            <value>{{master}}:18141</value>
          </property>
       
       
          <property>
            <name>yarn.resourcemanager.scheduler.address.rm2</name>
            <value>{{master_01}}:18030</value>
          </property>
          <property>
            <name>yarn.resourcemanager.webapp.address.rm2</name>
            <value>{{master_01}}:18088</value>
          </property>
          <property>
            <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
            <value>{{master_01}}:18025</value>
          </property>
          <property>
            <name>yarn.resourcemanager.admin.address.rm2</name>
            <value>{{master_01}}:18141</value>
          </property>
       
       </configuration>

    7，配置环境变量
    
        创建 /etc/profile.d/hadoop.sh 文件
        并且写入如下内容
        export HADOOP_HOME=/data/hadoop/hadoop-2.9.1/
        export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
        
    8，copy /data/hadoop/hadoop-2.9.1/到各个服务器上
        copy /etc/profile.d/hadoop.sh 到各个服务器上
        
    9, 在slave机器上启动 zookeeper ，启动 JournalNode
        zkServer.sh start
        hadoop-daemon.sh start journalnode
    
    9，在 master 执行 hdfs  namenode -format 格式化namespace
        copy /data/hadoop/tmp 文件到master-01机器上
        执行 hdfs zkfc -formatZK
        执行 start-dfs.sh 启动hadoop
        执行 start-yarn.sh 启动yarn   
        
        在master-01 节点上 启动namenode 和 resourcemanager
        yarn-daemon.sh start resourcemanager
        hadoop-daemon.sh start namenode
        nn1 由standby转化成active
        
        hdfs haadmin -transitionToActive nn1 --forcemanual
        启动history-server（mapred-site.xml 配置文件中配置的是master节点,所以在master节点上启动）
        Hadoop启动jobhistoryserver来实现web查看作业的历史运行情况，由于在启动hdfs和Yarn进程之后，jobhistoryserver进程并没有启动，需要手动启动，
        启动的方法是通过(注意：必须是两个命令)：
        ./mr-jobhistory-daemon.sh start historyserver
        ./yarn-daemon.sh start timelineserver 
        
        
```
&nbsp;&nbsp;步骤四： 测试hadoop

```
查看集群状态

/data/hadoop-2.7.1/bin/hdfs dfsadmin -report

测试yarn
http://master:18088

测试hdfs
http://master:50070

如有疑问 
 参考 
  ansible-playbooks -i jdk ../playbooks/jdk.yml --tags=jdksetup
  ansible-playbooks -i hadoop_cluster_ha ../playbooks/hadoop_cluster_ha.yml --tags=install

```