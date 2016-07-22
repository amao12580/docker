FROM java:openjdk-8-jre
RUN sed -i 's/http:\/\/archive\.ubuntu\.com\/ubuntu\//http:\/\/mirrors\.163\.com\/ubuntu\//g' /etc/apt/sources.list
RUN apt-get update && apt-get upgrade -y

#安装solr运行需要的工具
RUN apt-get update && \
  apt-get -y install lsof && \
  rm -rf /var/lib/apt/lists/*
	
ADD solr-5.3.0 /root/solr-5.3.0/

EXPOSE 8983

#启动solr
CMD /root/solr/bin/solr start -m 1g -f  
