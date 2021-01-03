# Sina-Imgbed
新浪（微博）图床，对接新浪API，实现图片外链，采用Spring-boot + Maven，预览地址：http://sina.1024.services

[Github地址](https://github.com/scriptwang/Sina-Imgbed)



# 食用方法
1. clone项目（建议用IDEA clone，会自动配置maven项目）
2. 修改配置
```
1. Sina-Imgbed/src/main/java/img/xxoo/util/ImgBed.SINA_USERNAME 修改为你的微博账号
2. Sina-Imgbed/src/main/java/img/xxoo/util/ImgBed.SINA_PASSWORD 修改为你的微博密码
```
3. 启动项目，访问http://127.0.0.1:9080/sinaimg/index.html，注意，第一次上传需要登录，所以久等一会

# docker
由于精力有限，并未实现docker传入微博用户名与密码，需要手动更改java代码，因此docker部署需要build镜像，加入打包好的项目文件名为```Sina-Imgbed-1.0.jar```
1. 将该文件传到Linux任意目录，并在该目录创建Dockerfile，写入以下内容
```
FROM openjdk:8-jre-alpine
MAINTAINER bingo
ADD Sina-Imgbed-1.0.jar sina.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","sina.jar"]
```
2. 创建镜像
```
docker build -t bingo/sinaimgbed .
```
3. run
```
docker run --name sinaimgbed -p 9095:9080 -d bingo/sinaimgbed
```

4. 访问http://yourip:9095/sinaimg/index.html


# 图示

![](https://cdn.sinaimg.cn.52ecy.cn/large/6fcb98b3gy1g1xvdr00t8g21i70u0npk.gif)

GIF太大好像不能显示，戳链接https://ws1.sinaimg.cn/large/6fcb98b3gy1g1xvdr00t8g21i70u0npk.gif

其实录制好的动图也是通过该图床上传的



觉得还好用的点一个小星星吧^ ^ ~，或者请作者喝一杯咖啡~~~

![](https://ws3.sinaimg.cn/large/6fcb98b3gy1g1xvmg9l2kj20bc06edgt.jpg)
