  ELK其实并不是一款软件，而是一整套解决方案，是三个软件产品的首字母缩写，Elasticsearch，Logstash 和 Kibana。这三款软件都是开源软件，通常是配合使用，而且又先后归于 Elastic.co 公司名下，故被简称为ELK协议栈。
 >  Logstash分布于各个节点上搜集相关日志、数据，并经过分析、过滤后发送给远端服务器上的Elasticsearch进行存储。Elasticsearch将数据以分片的形式压缩存储并提供多种API供用户查询，操作。用户亦可以更直观的通过配置Kibana Web方便的对日志查询，并根据数据生成报表。
# 1、Docker安装

* 安装基础组件
 ```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```
* 安装docker
```shell yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce
```
* 加速docker
```shell
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://tzc8ht4r.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```


* 开机启动docker
```shell
systemctl enable docker
```
# 2、Elasticsearch安装
* 下载Elasticsearch7.17.3的docker镜像：
```shell
docker pull elasticsearch:7.17.3
```
* 修改虚拟内存区域大小，否则会因为过小而无法启动:
```shell
 sysctl -w vm.max_map_count=262144
 ```
* 使用如下命令启动Elasticsearch服务，内存小的服务器可以通过ES_JAVA_OPTS来设置占用内存大小：
```shell
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-e "ES_JAVA_OPTS=-Xms512m -Xmx1024m" \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:7.17.3
```
* 启动时会发现/usr/share/elasticsearch/data目录没有访问权限，只需要修改/mydata/elasticsearch/data目录的权限，再重新启动即可；
```shell
chmod 777 /mydata/elasticsearch/data/ -R
```
 安装中文分词器IKAnalyzer，注意下载与Elasticsearch对应的版本，下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases


下载完成后解压到Elasticsearch的/mydata/elasticsearch/plugins目录下；


重新启动服务：
```shell
docker restart elasticsearch
```
开启防火墙：
```shell
firewall-cmd --zone=public --add-port=9200/tcp --permanent
firewall-cmd --reload
```
访问会返回版本信息：http://192.168.2.133:9200
```json
{
  "name" : "2b344c3c693e",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "Km7MwTZqQKKMI5k5B1z49w",
  "version" : {
    "number" : "7.17.3",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "5ad023604c8d7416c9eb6c0eadb62b14e766caff",
    "build_date" : "2022-04-19T08:11:19.070913226Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
# 3、Logstash安装
* 下载Logstash7.17.3的docker镜像：
```shell
docker pull logstash:7.17.3
```
修改Logstash的配置文件logstash.conf中output节点下的Elasticsearch连接地址为es:9200，配置文件地址：
> https://github.com/macrozheng/mall/blob/master/document/elk/logstash.conf
```shell
output {
  elasticsearch {
    hosts => "es:9200"
    index => "mall-%{type}-%{+YYYY.MM.dd}"
  }
}
```
* 创建/mydata/logstash目录，并将Logstash的配置文件logstash.conf拷贝到该目录；

> mkdir /mydata/logstash

使用如下命令启动Logstash服务；
```shell
docker run --name logstash -p 4560:4560 -p 4561:4561 -p 4562:4562 -p 4563:4563 \
--link elasticsearch:es \
-v /mydata/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
-d logstash:7.17.3
```
* 进入容器内部，安装json_lines插件。

> logstash-plugin install logstash-codec-json_lines
# 4、 Kibana安装
* 下载Kibana7.17.3的docker镜像：
```shell
docker pull kibana:7.17.3
```
使用如下命令启动Kibana服务：
```shell
docker run --name kibana -p 5601:5601 \
--link elasticsearch:es \
-e "elasticsearch.hosts=http://es:9200" \
-d kibana:7.17.3
```
开启防火墙：
```shell
firewall-cmd --zone=public --add-port=5601/tcp --permanent
firewall-cmd --reload
```
访问地址进行测试：http://192.168.2.133:5601
![在这里插入图片描述](https://img-blog.csdnimg.cn/251a7ca121e143cda0ccc15023f150cf.png)


你需要做的第一件事是将一些数据输入 Kibana 中进行处理。 选择部署并运行 Elasticsearch 后，你可以首次登录 Kibana。
要探索 Kibana，你可以使用 Kibana 示例数据或你自己的数据。 如果选择后者，则 Kibana 提供了多种方法来提取数据。 例如，如果你使用 Beats（专用于 Elastic 的数据采集代理），则只需选择 Beats 应该从哪个系统收集数据，然后让 Beats 连续为你收集数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/bce30cd4903445d4ba2de9f369db825e.png)
