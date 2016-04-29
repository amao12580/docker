#定义基础镜像
FROM alpine:latest

#定义nginx版本
ENV NGINX_VERSION 1.9.14

#将安装源切换为国内环境(中国科学技术大学)，大大加快了安装速度，同时稳定性也有了保障
ENV MIRROR_URL http://mirrors.ustc.edu.cn/alpine/

ENV MIRROR_URL_BACKUP http://alpine.gliderlabs.com/alpine/

ENV MIRROR_URL_SLOWEST http://dl-cdn.alpinelinux.org/alpine/

#准备安装环境
RUN echo '' > /etc/apk/repositories && \
    echo "${MIRROR_URL}v3.3//main"     >> /etc/apk/repositories && \
    echo "${MIRROR_URL}v3.3//community" >> /etc/apk/repositories && \
    echo '185.31.17.249 github.com' >> /etc/hosts && \
    echo '202.141.160.110 mirrors.ustc.edu.cn' >> /etc/hosts && \
    echo '206.251.255.63 nginx.org' >> /etc/hosts

#安装必要的组件(如果发生  ERROR: Service 'nginx' failed to build: The command '/bin/sh -c apk add... returned a non-zero code: 12。  这是网络问题：请删干净未完成container和images，10分钟后再来一遍)
RUN apk add --no-cache --virtual .build-deps \
    gcc \
    libc-dev \
    make \
    openssl-dev \
    pcre-dev \
    zlib-dev \
    linux-headers \
    curl \
    jemalloc-dev \
    gd-dev \
    git
#下载安装包和补丁
RUN mkdir -p /var/run/nginx/
RUN wget -c http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
RUN git clone https://github.com/cuber/ngx_http_google_filter_module.git
RUN git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module.git
RUN git clone https://github.com/aperezdc/ngx-fancyindex.git
RUN git clone https://github.com/yaoweibin/nginx_upstream_check_module.git

#进行编译安装，同时打上补丁
RUN tar -xzvf nginx-${NGINX_VERSION}.tar.gz && \
cd nginx-${NGINX_VERSION} && \
cd src/ && \
#打补丁
patch -p1 < /nginx_upstream_check_module/check_1.9.2+.patch && \
cd .. && \
#去除nginx的对外版本号
sed -i -e 's/${NGINX_VERSION}//g' -e 's/nginx\//ERROR/g' -e 's/"NGINX"/"ERROR"/g' src/core/nginx.h  && \
./configure --prefix=/usr/local/nginx \
--with-pcre \
--with-ipv6 \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_mp4_module \
--with-http_image_filter_module \
--with-http_addition_module \
--with-http_sub_module  \
--with-http_dav_module  \
--http-client-body-temp-path=/usr/local/nginx/client/ \
--http-proxy-temp-path=/usr/local/nginx/proxy/ \
--http-fastcgi-temp-path=/usr/local/nginx/fcgi/ \
--http-uwsgi-temp-path=/usr/local/nginx/uwsgi \
--http-scgi-temp-path=/usr/local/nginx/scgi \
--add-module=../ngx_http_google_filter_module \
--add-module=../ngx_http_substitutions_filter_module \
--add-module=../ngx-fancyindex \
--add-module=../nginx_upstream_check_module \
--with-ld-opt="-ljemalloc" && \
#开始编译
make -j $(awk '/processor/{i++}END{print i}' /proc/cpuinfo) && make install && \

#设置一些工作目录
mkdir -p /usr/local/nginx/cache/ && \ 
mkdir -p /usr/local/nginx/temp/ && \ 
rm -rf ../{ngx*,nginx*}

#启动nginx，保留一个前台进程，以免被docker强制退出
CMD ./usr/local/nginx/sbin/nginx && tail -f /usr/local/nginx/logs/error.log