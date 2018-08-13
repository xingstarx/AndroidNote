#### 查看内存占用
free -m
ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' 

#### 云主机下buff/cache内存占用过多
echo 1 > /proc/sys/vm/drop_caches

echo 2 > /proc/sys/vm/drop_caches

echo 3 > /proc/sys/vm/drop_caches
