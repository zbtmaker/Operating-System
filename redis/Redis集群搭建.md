
## Step1、安装redis
使用homebrew下载redis
```bash
brew install redis
```
下载好以后，redis相关的命令在文件夹中
```bash
/usr/local/Cellar/redis/7.0.0/bin
```
redis相关的配置在另一个文件夹中
```bash
/usr/local/etc
```

## step2、构建集群
在~目录下（其实这个目录时用户的自己目录，例如：/Users/xxx），创建一个redisCluster目录
```bash
mkdir redisCluster
```
进入到redisCluster目录下，然后依次创建6个目录7500、7501、7502、7503、7504、7505，为什么是六个呢？因为后面我们使用命令创建集群的时候，需要三主三重。然后我们将/usr/local/etc目录下的redis.conf文件分贝拷贝到7500、7501、7502、7503、7504、7505目录下。这里我们只需要示范7500目录下的redis.conf文件，其他目录类似，只需要替换其中的port端口配置即可
```
port 7500
cluster-enabled yes
cluster-node-timeout 5000
cluster-config-file nodes.conf
appendonly yes
```

然后依次进入7500、7501、7502、7503、7504、7505目录并执行下面的命令
```bash
redis-server redis.conf
```
执行完上述的步骤后，所有的redis服务器都已经启动，此时需要做的事情就是进入到7500目录下，然后执行
```bash
redis-cli -c -p 7500
```
此时启动了7500的客户端，然后执行下面的命令，建立起整个集群。
```bash
redis-cli --cluster create 127.0.0.1:7500 127.0.0.1:7501 127.0.0.1:7502 127.0.0.1:7503 127.0.0.1:7504 127.0.0.1:7505 --cluster-replicas 1
```

## Step3、执行slot迁移