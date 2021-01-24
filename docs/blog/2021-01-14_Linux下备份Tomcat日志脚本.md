# 场景
Tomcat运行久了之后logs目录下会产生大量的日志文件，并且文件大小都很大，所以需要定时任务，每天将Tomcat日志目录下的日志压缩备份到其他目录

# 脚本
TOMCAT_DIR为Tomcat的路径，BACKUP_DIR为日志备份路径，整个逻辑就是将Tomcat日志目录下昨天的日志进行压缩并且转移到备份路径
```bash
#!/bin/bash
TOMCAT_DIR=$1
BACKUP_DIR="~/logBack/tomcat"
YESTERDAY=`date --date="yesterday" "+%Y-%m-%d"`
STARTDATE=$(date "+%Y-%m-%d %H:%M:%S")

echo "*********** ${STARTDATE} *************"
echo "*********** Start Backup *************"
[ ! -d $BACKUP_DIR ] && mkdir -p $BACKUP_DIR
APP_NAME=`echo $TOMCAT_DIR | awk -F"/" '{print $3}'`
[ ! -d $BACKUP_DIR/$APP_NAME ] && mkdir $BACKUP_DIR/$APP_NAME

if [ -d $TOMCAT_DIR/logs ];then
        cd $TOMCAT_DIR/logs
        ls | grep $YESTERDAY > /dev/null
        if [ $? -eq 0 ];then
                gzip *.$YESTERDAY.*
                mv *.gz $BACKUP_DIR/$APP_NAME
                echo "$APP_NAME : successful !!"
        else
                echo "there are not files about logs of tomcat"
        fi
else
        echo "${TOMCAT_DIR}/logs : doesn't exist"
        exit 1
fi
echo "*********** End Backup *************"
```
# 添加定时任务
每天零点5分的时候备份昨天的日志文件
```bash
5 0 * * * ~/tomcat_log_backup.sh /app/tomcat >> ~/tomcat_log_backup.log
```