# 常用知识
## 1.整体启动/停止
`start-dfs.sh(yarn.sh)`/`stop-dfs.sh(yarn.sh)`
## 2.各个服务组逐一启动/停止
- **HDFS** ： `hdfs --daemon start/stop namenode/datanode/secondarynamenode`
- **YARN** :  `yarn --daemon start/stop resourcemanager/nodemanager`
## 3.Hadoop集群启停脚本（包含HDFS、YARN、HistoryServer）：myhadoop.sh
```bash
#!/bin/bash
if [ $# -lt 1 ]
then
 echo "No Args Input..."
 exit ;
fi
case $1 in
"start")
 echo " =================== 启动 hadoop 集群 ==================="
 echo " --------------- 启动 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
 echo " --------------- 启动 yarn ---------------"
 ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
 echo " --------------- 启动 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
 echo " =================== 关闭 hadoop 集群 ==================="
 echo " --------------- 关闭 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
 echo " --------------- 关闭 yarn ---------------"
 ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
 echo " --------------- 关闭 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
 echo "Input Args Error..."
;;
esac
```
（保存退出后记得赋予脚本执行权限
chmod 777 myhadoop.sh )

## 4.同时查看三台服务器java进程脚本：jpsall
```bash
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
 echo =============== $host ===============
 ssh $host jps 
done
```
（保存退出后记得赋予脚本执行权限
chmod 777 jpsall )
## 5.常用端口号说明
<img width="890" height="330" alt="image" src="https://github.com/user-attachments/assets/485da2b9-01e9-476d-8a95-bea17cd9dc28" />
## 6.常用配置文件
- **3.x**：core-site.xml  hdfs-site.xml  mapred-site.xml yarn-site.xml workers
- **2.x**: core-site.xml  hdfs-site.xml  mapred-site.xml yarn-site.xml slaves

