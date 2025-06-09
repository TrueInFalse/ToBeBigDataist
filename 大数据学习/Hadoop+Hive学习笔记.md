

# Hadoop

***命令一般在hadoop普通用户下执行***

>**环境变量：${HADOOP_HOME}=/export/server/hadoop**
>
>jps:查看不同进程情况
>
>Linux超级用户是root；HDFS的超级用户：是启动namenode的用户；



### 集群启停命令
- Hadoop一键启停：
  - 一键启动:`$HADOOP_HOME/sbin/start-dfs.sh`、
  - 一键停止:`$HADOOP_HOME/sbin/stop-dfs.sh`、
- Hadoop单进程启停：
  - `$HADOOP_HOME/sbin/hadoop-daemon.sh`：`hadoop-daemon.sh (start|status|stop) (namenode|secondarynamenode|datanode)`、
  - `$HADOOP_HOME/bin/hdfs`：`hdfs --daemon (start|status|stop) (namenode|secondarynamenode|datanode)`

### HDFS文件系统
- `hadoop fs [generic options Linux命令]`、`hdfs dfs [generic options Linux命令]`
- 协议头：
  - Linux协议头：`file://`
  - HDFS协议头：`hdfs://ndoe1:8020`
- HDFS WEB UI：浏览器输入网站`node1:9870`（只读权限简单浏览）；

### YARN资源调度器
- Hadoop_YARN应用管理界面：`node1:8088`
- YARN一键启停：
  - 一键启动：`$HADOOP_HOME/sbin/start-yarn.sh`
  - 一键停止：`$HADOOP_HOME/sbin/stop-yarn.sh`
- YARN单进程启停：
  - `$HADOOP_HOME/bin/yarn --daemon (start|stop) (resourcemanager|nodemanager|proxyserver) `
  - `$HADOOP_HOME/bin/mapred --daemon (start|stop) (resourcemanager|nodemanager|proxyserver)`


# Hive

- Hive中创建的库和表的数据，存储在HDFS中，默认存放在：`hdfs://node1:8020/user/hive/warehouse`；

### 初始化元数据库（仅需一次）
>启动Hive之前，需要先初始化Hive所需的元数据库
- 在MySQL中创建数据库hive：`CREATE DATABASE hive CHARSET UTF8;`
  - 进入MySQL：`mysql -uroot -p`
- 初始化元数据库（成功后会在MySQL的hive库中新建74张元数据管理的表）：
  - `cd /export/server/hive`
  - `bin/schematool -itheima -dbType mysql -verbos`

### 启动Hive（使用hadoop用户）
>以下的`/bin/hive`都在`/export/server/hive`下
- 确保有hive的日志文件夹：`mkdir /export/server/hive/logs`
- **启动元数据管理服务**（必须项）：
  - 前台启动：`bin/hive --service metastore`
  - 后台启动：`nohup bin/hive --service metastore >> logs/metastore.log 2>&1 &`
- 启动客户端：
  - Hive Shell方式（直接写SQL）：`bin/hive`（Hive 4.x直接进入beeline；Hive 2.x 3.x进入hive）
  - Hive ThriftServer方式（需连接外部客户端）：`bin/hive --service hiveserver2`
    - 后台启动：`nohup bin/hive --service hiveserver2 >> logs/hiveserver2.log 2>&1 &`
- `ps -ef | grep [端口ID]`来查看是hiveserver2还是Hivemetastore；`netstat -anp | grep 10000`查看谁占用了10000端口；
- Beeline是JDBC的客户端，进入方式`bin/beeline`，通信协议地址：`jdbc:hive2://node1:10000`，连接方式`!connect + [通信协议地址]`；
- 连接外部客户端：DBeaver、DataGrip等（需要JDBC驱动，Hive完全启动完成）


### Hive的操作语法
- 















