
- [编写配置文件](#编写配置文件)
- [编写dockerfile](#编写dockerfile)

# 使用docker 创建具有utf8字符集的mysql

### 编写配置文件
```shell
vi my.cnf
# 输入以下内容：
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
collation-server=utf8_general_ci
character-set-server=utf8
init-connect='SET NAMES utf8'

# 保存退出
:wq
```


### 编写dockerfile
```shell
vi Dockerfile
# 输入以下内容：
from mysql:5.7
COPY my.cnf /etc/mysql/conf.d/mysqlutf8.cnf
CMD ["mysqld", "--character-set-server=utf8", "--collation-server=utf8_unicode_ci"]
```

### 通过Dockerfile构建Docker镜像
```
docker build -t mysql5.7utf8 .
```

### 启动新的docker

```shell
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=mysql -itd mysql5.7utf8:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
# or

docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=mysql -itd mysql5.7utf8:latest
```

### 链接
```
mysql -h127.0.0.1 -uroot -P3306 -p 
```

### 查看字符集
```
mysql> show variables like '%char%'; 
```

如果忘记了密码可以用`docker inspect`查看环境变量