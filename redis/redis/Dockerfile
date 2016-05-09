#定义基础镜像
FROM redis:3.0.5

#构建工作文件夹
RUN mkdir -p /usr/local/bin/conf/ && \
mkdir -p /usr/local/bin/run/ && \
mkdir -p /usr/local/bin/logs/ && \
mkdir -p /usr/local/bin/backup/

#定义启动redis，需要挂载配置文件
CMD cd /usr/local/bin/ && ./redis-server ./conf/redis.conf  && tail -f ./logs/redis.log