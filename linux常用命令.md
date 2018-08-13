#### 查看内存占用
free -m
ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' 

#### 云主机下buff/cache内存占用过多
echo 1 > /proc/sys/vm/drop_caches

echo 2 > /proc/sys/vm/drop_caches

echo 3 > /proc/sys/vm/drop_caches

#### docker exec 命令能让你在一个容器中额外地运行新命令
docker exec -it some-mysql bash

#### 查看docker容器的host信息
cat /etc/hosts
