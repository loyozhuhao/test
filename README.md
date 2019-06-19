# test

### ElasticSearch(搜索服务)

- [ElasticSearch](https://www.elastic.co://www.elastic.co/)(简称ES)是一个搜索服务，使用它的主要目的在于解决搜索效率比较糟糕的几个场景，最开始也尝试过通过MongoDB的文本索引，但是效果不太理想. 

- ES部署在env1服务器上, 主目录在`/mnt/es`, 使用docker-compose部署，并限制了其对CPU和内存的使用率, 你可以修改docker-compose.yml文件中的配置项来改变限制行为.
```
mem_limit: 4096m
cpu_quota: 240000
```

- 常用命令：
```
# 查看服务状态 
docker-compose ps
# 实时查看日志 
docker-compose logs -f es
# 停止es服务
docker-compose down
# 启用es服务
docker-compose up -d
# 查看服务资源占用情况
docker stats es
```

### es集群搭建，基于docker swarm

- 镜像迁移需要重新安装docker
```	
#!/bin/bash
	apt  autoremove -y docker*
	rm /etc/docker
	rm /var/lib/docker -Rf
	reboot
	apt update -y
	apt install -y docker.io
```

[参考原文](http://derpturkey.com/elasticsearch-cluster-with-docker-engine-swarm-mode/)

- 前置条件：
	- 安装`docker`，参考[安装教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/)( `docker`版本请使用 `17.12.1-ce` )
	- 安装`nginx`
	- 在`/mnt/es/`放`docker-compose.yml`，内容见[[docker-compose.yml]]
	- 在`/mnt/es/`创建`data`目录(否则会报错：`invalid mount config for type "bind": bind source path does not exist`)
	- 编辑`/etc/sysctl.conf`，加入`vm.max_map_count=262144`(否则会报错：`max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`)，并执行`sysctl -p`使配置生效

- 初始化Swarm集群 

```
#init swarm master node - app3
docker swarm init --advertise-addr=10.26.209.6
#join node - app4, app5
docker swarm join-token worker
会出现类似结果：
docker swarm join \
    --token SWMTKN-1-0ru2j3y7rlp859i2869c6lf3ra1vdqagrdl21g7dlpyjmkg4i6-d7p4o9fk5j61qpp3hfqclhsa0 \
    10.26.209.6:2377
在每台worker节点执行该命令
#list node
docker node ls
```

- 部署ES服务

```
#app3, docker-compose.yml
cd /mnt/es/
#deploy stack on swarm 
#修改配置以后重新执行
docker stack deploy --compose-file docker-compose.yml es
#list stack && service
docker stack ls && docker service ls
#add node lable, node id get by `docker node ls`
docker node update --label-add app_role=elasticsearch `node_id`
#update es service endpoint
docker service update --endpoint-mode=dnsrr --detach=false es_elasticsearch
```

- 备注

	* 数据保存在每台服务的`/mnt/es/data`目录下，在`docker-compose.yml`文件中由`/mnt/es/data:/usr/share/elasticsearch/data`指定。

	* 部署模式为`global`，会在每个节点上创建容器。

	* `node.max_local_storage_nodes=3`指定节点数。




