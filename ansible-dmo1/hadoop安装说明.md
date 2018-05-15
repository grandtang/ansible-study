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
    5，修改hadoop配置文件（/data/hadoop/hadoop-env.sh）
```