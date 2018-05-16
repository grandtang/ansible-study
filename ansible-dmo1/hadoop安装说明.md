非HA的安装方式<br>
&nbsp;&nbsp;实例：
  ```
         server     安装软件        运行的进程
         master      jdk,hadoop     SecondaryNameNode,ResourceManager,NameNode
         slave-01    jdk,hadoop     NodeManager,DataNode
         slave-02    jdk,hadoop     NodeManager,DataNode
  
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
 &nbsp;&nbsp;步骤二：安装hadoop（hadoop用户）
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
    4，解压hadoop包到/data/hadoop目录中（hadoop用户）
       mkdir /data/hadoop
       tar -zxvf hadoop-2.9.1.tar.gz -C /data/hadoop
    5，修改hadoop-env配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/hadoop-env.sh）
       修改变量 hadoop-env中的变量JAVA_HOME的值如下
       export JAVA_HOME=/opt/java/jdk1.8.0_
    6，修改core-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/core-site.xml）如下
       <configuration>
           <!-- 指定HDFS老大（namenode）的通信地址 -->
           <property>
               <name>fs.defaultFS</name>
               <value>hdfs://{{master}}:9000</value>
           </property>
           <!-- 指定hadoop运行时产生文件的存储路径 -->
           <property>
               <name>hadoop.tmp.dir</name>
               <value>{{hadoop_tmp_dir}}</value>
           </property>
       
       </configuration>
       
    7，修改hdfs-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/hdfs-site.xml）如下
        <configuration>
        
            <!-- 设置namenode的http通讯地址 -->
            <property>
                <name>dfs.namenode.http-address</name>
                <value>{{master}}:50070</value>
            </property>
        
            <!-- 设置secondarynamenode的http通讯地址 -->
            <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>{{slave_01}}:50090</value>
            </property>
        
            <!-- 设置namenode存放的路径 -->
            <property>
                <name>dfs.namenode.name.dir</name>
                <value>{{dfs_namenode_name_dir}}</value>
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
    
    8，修改mapred-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/mapred-site.xml）如下
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
    
    9，修改yarn-site配置文件（/data/hadoop/hadoop-2.9.1/etc/hadoop/yarn-site.xml）如下
        <configuration>
            <!-- 设置 resourcemanager 在哪个节点-->
            <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>{{master}}</value>
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
        
           <property>
             <name>yarn.resourcemanager.scheduler.address</name>
             <value>{{master}}:18030</value>
           </property>
           <property>
             <name>yarn.resourcemanager.webapp.address</name>
             <value>{{master}}:18088</value>
           </property>
           <property>
             <name>yarn.resourcemanager.resource-tracker.address</name>
             <value>{{master}}:18025</value>
           </property>
           <property>
             <name>yarn.resourcemanager.admin.address</name>
             <value>{{master}}:18141</value>
           </property>
        </configuration>
    10，配置环境变量
        创建 /etc/profile.d/hadoop.sh 文件
        并且写入如下内容
        export HADOOP_HOME=/data/hadoop/hadoop-2.9.1/
        export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    11，copy /data/hadoop/hadoop-2.9.1/到各个服务器上
        copy /etc/profile.d/hadoop.sh 到各个服务器上
    12，在master
     
```