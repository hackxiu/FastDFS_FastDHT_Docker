#基础镜像
FROM alpine:latest

MAINTAINER Hackxiu <hackxiu@gmail.com>

# 环境变量 # 配置启动环境变量脚本，在启动时中根据环境变量替换nginx端口、fastdfs端口
ENV TZ=Asia/Shanghai HOME=/root Data_tracker="\/home\/dfs\/tracker" Data_storage="\/home\/dfs\/storage" Data_fdht="\/home\/dfs\/fdht" WEB_PORT=8080 FDHT_PORT=11411 FDFS_PORT=22122 FDFSD_PORT=23000

# 复制工具
ADD soft ${HOME}

# conf_配置
COPY /conf/fdfs/* /etc/fdfs/
COPY /conf/fdht/* /etc/fdht/

# 执行命令
RUN set -x \
    && apk add -U tzdata \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && apk update \
    && apk add --no-cache --virtual .build-deps alpine-sdk build-base perl-dev openssl-dev pcre-dev zlib-dev git bash gcc libc-dev make linux-headers curl gnupg libxslt-dev gd-dev geoip-dev autoconf \
    && echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories \
    && echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories \

    #进入目录解压所需源码
    && cd ${HOME} \
    && tar xvf libfastcommon-1.0.43.tar.gz \
    && tar xvf db-4.7.25.tar.gz \
    && tar xvf fastdfs-6.06.tar.gz \
    && tar xvf fastdfs-nginx-module-1.22.tar.gz \
    && tar xvf fastdht-master.tar.gz \
    && tar xvf jemalloc-5.2.1.tar.gz \
    && tar xvf nginx-1.16.1.tar.gz \

    # 安装_libfastcommon
    && cd ${HOME}/libfastcommon-1.0.43 \
    && ./make.sh \
    && ./make.sh install \
    #&& ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so \

    #安装_berkeley db
    && cd ${HOME}/db-4.7.25/build_unix \
    && ../dist/configure -prefix=/usr \
    && make \
    && make install \

    # 安装_fastdfs
    && cd ${HOME}/fastdfs-6.06/ \
    && ./make.sh \
    && ./make.sh install \

    # 安装_fastdht
    && cd ${HOME}/fastdht-master/ \
    && sed -i "s?CFLAGS='-Wall -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE'?CFLAGS='-Wall -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -I/usr/include/ -L/usr/lib/'?" make.sh \
    && ./make.sh clean \
    && ./make.sh \
    && ./make.sh install \

    # 安装_jemalloc_内存优化模块
    && cd ${HOME}/jemalloc-5.2.1/ \
    && ./autogen.sh \
    #&& make install_bin install_include install_lib \
    && make \
    && make install \

    # 获取nginx源码，与fastdfs插件一起编译
    && mkdir -p /usr/local/nginx/module \
    && cp -a -r ${HOME}/fastdfs-nginx-module-1.22/ /usr/local/nginx/module/ \
    && cd ${HOME}/nginx-1.16.1/ \
    && ./configure --with-pcre-jit --with-ld-opt='-ljemalloc' --add-module=/usr/local/nginx/module/fastdfs-nginx-module-1.22/src/ \
    && make && make install \
    && mv /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx_conf.baks \
    && cp /etc/fdfs/nginx.conf /usr/local/nginx/conf \

    # 清理文件
    && rm -rf ${HOME}/* \
    && apk del .build-deps gcc libc-dev make openssl-dev linux-headers curl gnupg libxslt-dev gd-dev geoip-dev autoconf \
    && apk add --no-cache bash pcre-dev zlib-dev

# PORT
EXPOSE 8080 11411 22122 23000

# 创建启动脚本
RUN echo -e "\
#!/bin/bash; \n\
mkdir -p /home/dfs/storage/data /home/dfs/tracker /home/dfs/fdht; \n\
ln -s /home/dfs/storage/data/ /home/dfs/storage/data/M00; \n\
sed -i \"s/^base_path = .*$/base_path = \$Data_storage/\" /etc/fdfs/client.conf; \n\
sed -i \"s/^base_path = .*$/base_path = \$Data_storage/\" /etc/fdfs/storage.conf; \n\
sed -i \"s/^store_path0 = .*$/store_path0 = \$Data_storage/\" /etc/fdfs/storage.conf; \n\
sed -i \"s/^base_path = .*$/base_path = \$Data_tracker/\" /etc/fdfs/tracker.conf; \n\
sed -i \"s/^store_path0=.*$/store_path0=\$Data_storage/\" /etc/fdfs/mod_fastdfs.conf; \n\
sed -i \"s/^base_path=.*$/base_path=\$Data_fdht/\" /etc/fdht/fdht_client.conf; \n\
sed -i \"s/^base_path=.*$/base_path=\$Data_fdht/\" /etc/fdht/fdhtd.conf; \n\
sed -i \"s/http.tracker_server_port = .*$/http.tracker_server_port = \$WEB_PORT/g\" /etc/fdfs/client.conf; \n\
sed -i \"s/http.server_port = .*$/http.server_port = \$WEB_PORT/g\" /etc/fdfs/storage.conf; \n\
sed -i \"s/http.server_port = .*$/http.server_port = \$WEB_PORT/g\" /etc/fdfs/tracker.conf; \n\
sed -i \"s/listen \ .*$/listen \ \$WEB_PORT;/g\" /usr/local/nginx/conf/nginx.conf; \n\
if [ \"\$IP\" = \"\" ]; then \n\
    IP=\`ifconfig eth0 | grep inet | awk '{print \$2}'| awk -F: '{print \$2}'\`; \n\
fi \n\
sed -i \"s/^tracker_server = .*$/tracker_server = \$IP:\$FDFS_PORT/\" /etc/fdfs/client.conf; \n\
sed -i \"s/^bind_addr = .*$/bind_addr = \$IP/\" /etc/fdfs/storage.conf; \n\
sed -i \"s/^port = .*$/port = \$FDFSD_PORT/\" /etc/fdfs/storage.conf; \n\
sed -i \"s/^bind_addr = .*$/bind_addr = \$IP/\" /etc/fdfs/tracker.conf; \n\
sed -i \"s/^port = .*$/port = \$FDFS_PORT/\" /etc/fdfs/tracker.conf; \n\
sed -i \"s/^tracker_server=.*$/tracker_server=\$IP:\$FDFS_PORT/\" /etc/fdfs/mod_fastdfs.conf; \n\
sed -i \"s/^storage_server_port=.*$/storage_server_port=\$FDFSD_PORT/\" /etc/fdfs/mod_fastdfs.conf; \n\
sed -i \"s/^bind_addr=.*$/bind_addr=\$IP/\" /etc/fdht/fdhtd.conf; \n\
sed -i \"s/^port=.*$/port=\$FDHT_PORT/\" /etc/fdht/fdhtd.conf; \n\
sed -i \"s/^group0 .*$/group0 = \$IP:\$FDHT_PORT/\" /etc/fdht/fdht_servers.conf; \n\
sed -i \"4d\" /etc/fdht/fdht_servers.conf; \n\
sed -i \"s/^check_file_duplicate = .*$/check_file_duplicate = 1/g\" /etc/fdfs/storage.conf; \n\
sed -i \"s/^keep_alive = .*$/keep_alive = 1/g\" /etc/fdfs/storage.conf; \n\
sed -i \"s/^##include \/home\/yuqing\/fastdht\/conf\/fdht_servers.conf/#include \/etc\/fdht\/fdht_servers.conf/g\" /etc/fdfs/storage.conf; \n\
/etc/init.d/fdfs_trackerd start; \n\
/usr/local/bin/fdhtd /etc/fdht/fdhtd.conf; \n\
/etc/init.d/fdfs_storaged start; \n\
/usr/local/nginx/sbin/nginx; \n\
tail -f /usr/local/nginx/logs/access.log \
">/start.sh \
&& chmod u+x /start.sh
ENTRYPOINT ["/bin/bash","/start.sh"]