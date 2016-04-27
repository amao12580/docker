#定义基础镜像
FROM centos:6.7

#定义nginx版本
ENV NGINX_VERSION 1.9.14

#准备安装环境
RUN yum install -y wget && \ 
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo  && \ 
yum clean all  && \ 
yum makecache && \

#安装依赖组件
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-* && \
yum install -y epel-release && \
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6 && \
yum install -y patch pcre-devel openssl-devel zlib-devel gd-devel tar gcc git supervisor && \

#下载安装包和补丁
mkdir -p /var/run/nginx/ && \
wget -c http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
git clone https://github.com/cuber/ngx_http_google_filter_module.git && \
git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module.git && \
git clone https://github.com/aperezdc/ngx-fancyindex.git && \
git clone https://github.com/yaoweibin/nginx_upstream_check_module.git && \

#进行编译安装，同时打上补丁
tar xf nginx-${NGINX_VERSION}.tar.gz && \
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
--add-module=../nginx_upstream_check_module && \
#开始编译
make -j $(awk '/processor/{i++}END{print i}' /proc/cpuinfo) && make install && \
#设置一些工作目录
mkdir -p /usr/local/nginx/cache/ && \ 
mkdir -p /usr/local/nginx/temp/ && \ 
rm -rf ../{ngx*,nginx*} && \
yum clean packages

#启动nginx，保留一个前台进程，以免被docker强制退出
CMD ./usr/local/nginx/sbin/nginx && tail -f /usr/local/nginx/logs/error.log