# docker 部署redis、mongo等环境

一、MySQL安装

```
搜索镜像
	docker search mysql
拽镜像
	docker pull  [选项] [Docker Registry地址]<仓库名>:<标签>
运行镜像
	docker run -p 3308:3306 --name senguo_mysql -e MYSQL_ROOT_PASSWORD=ensky123.(数据库密码) -d 8dbbe042b8f7(容器id)
进入容器
	docker exec -it senguo_mysql /bin/bash
```

二、Mongo 安装

```
运行容器
	docker run -itd --name mongo-test -p 27017:27017 mongo[容器名字或者容器id] --auth
进入容器
	 docker exec -it mongo mongo admin
创建用户和密码
	db.createUser({ user:'admin',pwd:'ensky123.',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
验证密码
	db.auth('admin', 'ensky123.')
```

三、redis安装

```
pull docker 镜像
	docker pull redis:3.2
运行容器
	docker run -itd --name redis-test -p 6379:6379 7614ae9453d1
进入容器
	docker exec -it redis-test bash
执行redis-cli
查看现有密码
	config get requirepass
设置密码	
	config set requirepass "ensky123."
```

四、rabbitMq安装

```
docker pull rabbitmq
docker run --name rabbitmq -d -p 15672:15672 -p 5672:5672 31b721acc90a
docker exec -it rabbitmq bash
执行rabbitmqctl add_user root ensky123. 添加用户，用户名为root,密码为ensky123.
rabbitmqctl set_permissions -p / root ".*" ".*" ".*"  赋予root用户所有权限
rabbitmqctl set_user_tags root administrator赋予root用户administrator角色
rabbitmqctl list_users查看所有用户即可看到root用户已经添加成功
```

