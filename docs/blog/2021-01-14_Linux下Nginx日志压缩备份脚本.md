# 场景
当Nginx运行久了之后，日志文件会很庞大，所以需要对日志文件进行滚动备份，可以用定时任务实现


# 脚本
```bash
#!/bin/bash
LOG_PATH="/usr/nginx/logs/"
PID_PATH="/usr/nginx/logs/nginx.pid"
BACKUP_PATH="~/logBack/nginx/"
BACKUP_DATE=`date -d "yesterday" +"%Y-%m-%d"`

[ ! -d $BACKUP_PATH ] && mkdir -p $BACKUP_PATH
#rename logfile 
mv ${LOG_PATH}access.log ${LOG_PATH}access_bak_${BACKUP_DATE}.log
#produce a new logfile
kill -USR1 `cat ${PID_PATH}`
# back logfile
if [ -d $LOG_PATH ];then
     cd $LOG_PATH
     tar zcf access_log_${BACKUP_DATE}.tar.gz access_bak*.log
     mv access_log*.tar.gz $BACKUP_PATH
     rm -f access_bak*.log
fi
```

# 定时任务
每天零点五分执行
```bash
5 0 * * * ~/nginx_log_rotate.sh >> ~/nginx_log_rotate.log
```