项目地址：https://gitee.com/ScriptWang/ssadmin_cname
nginx + websocket + tls + cname管理面板，注意此面板要配合后端一起使用

[【富强系列-一键脚本】nginx+websocket+ls一键脚本](【富强系列-一键脚本】nginx+websocket+ls一键脚本.md)

# 功能
- 添加用户，管理用户(添加/删除/续费/查看)
- 流量统计
- 主机监控，流量统计，命令执行，日志查看，域名指向查看，域名扩散
- 测试节点管理
- 安卓/IOS/MAC/PC端视频教程
- 管理员后台/客户后台


# 原理
- 流量统计/添加用户/删除用户：后台用java配合nginx配合fcgwrap
- 主机监控：用sshpass免交互登录其他主机


# 部署
将项目打成jar包
maven：```maven clean package```
执行jar文件：```java -jar admin-2.0.1.RELEASE.jar```
后台执行：```nohup java -jar admin-2.0.1.RELEASE.jar &```


# 配置
- application.yml里面spring.datasource.url配置数据库连接
- ss.ss.task.Task.statis：统计流量使用情况定时任务
- ss.ss.util.IpUtil里面配置了ip数据库的路径
- ss.ss.util.Constant.SRC_PATH 定义了基本资源路径，包含视频教程，客户端等的路径

