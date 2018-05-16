jdk

```
安装
ansible-playbook -i jdk ./playbooks/jdk.yml --tags=jdksetup

```


zookeeper
```
安装
ansible-playbook -i zookeeper_cluster ./playbooks/zookeeper_cluster.yml --tags=install
启动
ansible-playbook -i zookeeper_cluster ./playbooks/zookeeper_cluster.yml --tags=start
关闭
ansible-playbook -i zookeeper_cluster ./playbooks/zookeeper_cluster.yml --tags=stop
```

hadoop_ha
```
安装
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_cluster_ha.yml --tags=install
启动zookeeper
ansible-playbook -i zookeeper_cluster ./playbooks/zookeeper_cluster.yml --tags=start
启动journalnode
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_journal_node.yml
格式化hadoop
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_format.yml

启动hadoop
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_start.yml
启动master standby 节点namenode 和 resourcemanager
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_start_master_daemon.yml
启动historyServer
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_historyserver_start.yml

关闭hadoop
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_stop.yml
关闭master standby 节点namenode 和 resourcemanager
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_stop_master_daemon.yml
关闭historyServer
ansible-playbook -i hadoop_cluster_ha ./playbooks/hadoop_ha_historyserver_stop.yml
```
