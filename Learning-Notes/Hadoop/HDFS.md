## HDFS关键知识
## HDFS写数据流程
<img width="994" height="472" alt="image" src="https://github.com/user-attachments/assets/9f2aea65-5303-42b3-9883-ab337f9db1b1" />

1. 客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。
2. NameNode 返回是否可以上传。
3. 客户端请求第一个 Block 上传到哪几个 DataNode 服务器上。
4. NameNode 返回 3 个 DataNode 节点，分别为 dn1、dn2、dn3。
5. 客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用
dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。
6. dn1、dn2、dn3 逐级应答客户端。
7. 客户端开始往 dn1 上传第一个 Block（先从磁盘读取数据放到一个本地内存缓存），以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 每传一个 packet
会放入一个应答队列等待应答。
8. 当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务
器。（重复执行 3-7 步）。

## 网络拓扑距离计算
<img width="1028" height="483" alt="image" src="https://github.com/user-attachments/assets/e94cba6c-374d-4388-87cd-29eb74802b7d" />

## HDFS读数据流程
<img width="1076" height="489" alt="image" src="https://github.com/user-attachments/assets/693446a6-f448-4373-a153-6a236f1df54d" />

1. 客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查
询元数据，找到文件块所在的 DataNode 地址。
2. 挑选一台 DataNode（就近原则，然后随机）服务器，请求读取数据。
3. DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位
来做校验）。
4. 客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

## HDFS NN与2NN的工作机制
<img width="1079" height="542" alt="image" src="https://github.com/user-attachments/assets/071a290c-7001-4931-af8d-5de96ef1f95d" />

1. 第一阶段：NameNode 启动
（1）第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
（2）客户端对元数据进行增删改的请求。
（3）NameNode 记录操作日志，更新滚动日志。
（4）NameNode 在内存中对元数据进行增删改。
3. 第二阶段：Secondary NameNode 工作
（1）Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode是否检查结果。
（2）Secondary NameNode 请求执行 CheckPoint。
（3）NameNode 滚动正在写的 Edits 日志。
（4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。
（5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。
（6）生成新的镜像文件 fsimage.chkpoint。
（7）拷贝 fsimage.chkpoint 到 NameNode。
（8）NameNode 将 fsimage.chkpoint 重新命名成 fsimage。
