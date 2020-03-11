# FastDFS+FastDHT
使用docker-compose创建FastDFS+FastDHT版服务(tracker,storage,fastdht,nginx)
## 搭建教程
1. 安装docker和docker-compose  
2. 安装git    
3. clone项目    
 ```
 git clone https://hackxiu@github.com/hackxiu/FastDFS_FastDHT_Docker.git
 ```    
4. 进入fastdfs目录  
 ```    
 cd fastdfs
 ```    
5. 自动获取 Docker IP,不需要设置IP  
6. 设置数据存储目录 默认 /home/dfs , 需要把目录挂载到  /home/dfs 后
即可存储数据到指定目录
 ```    
7. 至此一个单机版的fastdfs文件系统已经搭建完成，下面测试fastdfs是否搭建成功
 ```    
8. 默认开启端口 nginx:8080 ,fastdht: 11411 , fastdfs:22122 , 22122 , 23000
 ```    
 docker exec -it fastdfs /bin/bash 

 echo "Hello FastDFS!">index.html

 fdfs_test /etc/fdfs/client.conf upload index.html
```      

 重启tracker_server
```
 /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
```
 重启storage_server
```
 /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```
 重启fastdht_server
```
 /usr/local/bin/fdhtd /etc/fdht/fdhtd.conf restart
```
 查看storage状态
```
 fdfs_monitor /etc/fdfs/client.conf
```
 该镜像已经上传到公开镜像仓库，也可跳过镜像构建步骤，直接从Docker Hub或者阿里云容器镜像仓库上拉取，linux环境下可用如下命令拉取镜像后直接运行容器
```
 docker pull lovetutu/fastdfs_fastdht 或者 docker pull registry.cn-hangzhou.aliyuncs.com/lovetutu/fastdfs_fastdht

 docker run -d --restart=always --net=host --name=fastdfs -e IP=192.168.0.105 -v ～/fastdfs:/var/local lovetutu/fastdfs
```