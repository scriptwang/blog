# 原理
通过top找到占用CPU高的进程pid，通过ps找到该进程中占用CPU高的线程tid，最后通过jstack找到该线程的堆栈信息，最后根据堆栈信息排查问题。

# top找到高占用CPU的java进程pid
命令：```top```
```
 PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                                      
28832 root      20   0 1417908 208592   6884 S  110.0 20.7   4:14.21 java                                                                                                                                         
 2597 root      20   0   10728   1240      0 S  0.3  0.1  84:02.05 containerd-shim                                                                                                                              
21233 100       20   0   16544   2548    636 S  0.3  0.3 103:36.06 nginx                                                                                                                                        
21387 root      20   0  149004   9772   8252 S  0.3  1.0   0:00.14 sshd    
```
可以看到pid为28832的java程序占用大量cpu

# 找占用高的线程tid
命令：```ps -mp ${pid} -o THREAD,tid,time```
第一行是表头，第二行是统计，从第三行开始看
```
[root@izwz9d0sy7ulokpdr023wlz ~]# ps -mp 28832 -o THREAD,tid,time
USER     %CPU PRI SCNT WCHAN  USER SYSTEM   TID     TIME
root      110.6   -    - -         -      -     - 00:04:17
root      0.0  19    - futex_    -      - 28832 00:00:00
root      0.0  19    - poll_s    -      - 28893 00:00:10
root      18.0  19    - futex_    -      - 28894 00:00:00
root      0.1  19    - futex_    -      - 28895 00:00:25
root      0.0  19    - futex_    -      - 28896 00:00:00
root      0.0  19    - futex_    -      - 28897 00:00:00
root      0.0  19    - futex_    -      - 28898 00:00:00
root      90.0  19    - futex_    -      - 28899 00:00:00 # 此进程cpu占用高达90%
root      0.0  19    - futex_    -      - 28900 00:00:00
root      0.0  19    - futex_    -      - 28901 00:00:00
root      0.0  19    - futex_    -      - 28902 00:00:00
```
可以看到，tid为28899的线程占用CPU很高，拿它开刀，先将tid转换成八进制
命令：```printf "%x\n" ${tid}```
```
printf "%x\n" 28899
70e3
```
# 用jstack查看该线程调用堆栈信息
命令：```jstack ${pid} | grep ${tid} -A 30```
```
jstack 28832 | grep 70e3 -A 30
```

# 最后
上面的命令有点不方便，还要转换八进制什么的，这里提供一条命令，直接查询某进程占用CPU量前10的线程tid的八进制格式（把其中的${PID}换成进程id），然后复制八进制tid直接用jstack查看堆栈信息
```
ps -mp ${PID} -o THREAD,tid | gawk 'NR!=1 && NR!=2 { printf "%s %x\n",$2,$8 }' | sort -rn | head -10
```
输出如下，第一列是CPU占用百分比，第二列是线程八进制字符串，也就是tid
```
1.4 712a
0.1 70df
0.0 714a
0.0 7149
0.0 7148
0.0 7147
0.0 7146
0.0 7145
0.0 7144
0.0 7143
```

- ```jstack ${PID} | grep ${tid} -A 30```直接打印该线程

# 第二种方法
- top找到占用高的进程，找到pid
- ```top -Hp ${pid}```直接查看线程，观看占用CPU资源多的线程pid即tid
- ```jstack ${pid} | grep ${tid} -A 30```直接打印该线程
