# 原理

原理是创建两个docker容器，一个是nginx，一个是ss，用nginx反代ss实现流量转发，nginx绑定域名并用Let`s Encrypt认证域名绑定Https证书走TLS流量。

功能：

- 认证域名（基于`certbot-nginx`，背后是用的Let's Encrypt实现的认证）

- 流量统计（基于`fcgiwrap` ，原理是通过Http请求到Nginx，Nginx通过fcgiwrap执行shell脚本，脚本通过分析nginx访问日志进行流量统计）

- 添加代理path（基于fcgiwrap）

- 删除代理path（基于fcgiwrap）

欲知更多关于fcgiwrap的信息，请参考[fcgiwrap 的简单使用](https://blog.twofei.com/642/)

# 手动使用

假设代理根path为v2rayRootPath，域名为domain.com 

## 定义变量

```bash
domain=domain.com        #你的域名
v2rayPath=v2rayRootPath  #v兔瑞根路径
sspwd=sspwd              #ss密码
ssmethod=chacha20-ietf-poly1305 #ss加密方式
ssport=8388              #ss端口
```

## 运行ss容器

```bash
docker run -d \
--name ss \
--restart=always \
-e "ARGS=--plugin v2ray-plugin --plugin-opts server;path=/$v2rayPath;loglevel=none" \
-e PASSWORD=$sspwd \
-e SERVER_PORT=$ssport \
-e METHOD=$ssmethod \
-v /etc/localtime:/etc/localtime \
kimoqi/ssv2ray

```

## 获取ss的容器(内网)IP

```bash
ssip=`docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'  ss`
```

## 运行nginx容器，映射80/443端口

```bash
docker run --name ngx \
--restart=always \
-e v2rayPath=$v2rayPath \
-e domain=$domain \
-e ssipport=${ssip}:${ssport} \
-p 80:80 -p 443:443 \
-v /etc/letsencrypt:/etc/letsencrypt \
-v /etc/ngx:/etc/ngx \
-v /etc/localtime:/etc/localtime \
-d kimoqi/nginx4ss
```

访问试试是否成功`curl  http://$domain`

## 域名HTTPS认证

```bash
docker exec -it ngx certbot --nginx --register-unsafely-without-email
```

接下来跟着提示输入就行，最后，访问试试是否成功`curl  https://$domain`

## 添加代理path

例：添加test
访问```https://domain.com/v2rayRootPath/add?test```即可

## 删除代理path

例：删除test
访问```https://domain.com/v2rayRootPath/del?test```即可

## 流量统计

访问```https://domain.com/v2rayRootPath/statis```即可

## 客户端参数

```
服务器名称: $domain
服务器端口: 443（切记这里的端口是nginx监听的443端口）
密码: $sspwd
加密方式: chacha20-ietf-poly1305
插件程序: v2ray
插件参数:tls;host=$domain;path=/$v2rayPath
```

插件

- PC：https://github.com/shadowsocks/v2ray-plugin/releases

- 安卓：https://github.com/shadowsocks/v2ray-plugin-android/releases

- IOS：2.1.37版本以上小火箭（shadowrocket）才支持

- MAC：[点击下载小飞机 v1.9.4](http://dash.videohub.top/accounttest/Test4User/tutoral?name=ShadowsocksX-NG.1.9.4.zip)，此版本需要Mac OS版本大于**v10.12**，如果不行，[请到github进行下载](https://github.com/shadowsocks/ShadowsocksX-NG/releases)，注意版本需**大于 v1.9.0!!!**

  

# 一键脚本使用

如果`SSoverTLS_Adv.sh`和`AddDomain.sh`脚本内容失效，直接复制下面的脚本源码

## 帮助
```bash
bash <(curl https://raw.githubusercontent.com/scriptwang/SSoverTLS/master/sh/SSoverTLS_Adv.sh) -h
```

## 安装
```bash
bash <(curl https://raw.githubusercontent.com/scriptwang/SSoverTLS/master/sh/SSoverTLS_Adv.sh) -i
```

## 更新伪站
```bash
bash <(curl https://raw.githubusercontent.com/scriptwang/SSoverTLS/master/sh/SSoverTLS_Adv.sh) -w
```

## 认证新域名
需要在安装之后认证
```bash
bash <(curl https://raw.githubusercontent.com/scriptwang/SSoverTLS/master/sh/AddDomain.sh)
```

# 脚本源码SSoverTLS_Adv.sh

此脚本用于实现一键安装，更新伪站的功能，基于docker

```bash
#!/bin/bash


# 产生随机密码
function getRandomPwd(){
    num=32
    if [ $# == 1 ];then
        num=$1
    fi
    echo "$(date +%s)$(shuf -i 10000-65536 -n 1)" | sha256sum | base64 | head -c $num ; echo
}

# url编码
function urlencode() {
  local length="${#1}"
  for (( i = 0; i < length; i++ )); do
    local c="${1:i:1}"
    case $c in
      [a-zA-Z0-9.~_-]) printf "$c" ;;
    *) printf "$c" | xxd -p -c1 | while read x;do printf "%%%s" "$x";done
  esac
done
}

function get_random_webs(){
    webs=(
    "http://down.cssmoban.com/cssthemes6/sstp_7_Kairos.zip"
    "http://down.cssmoban.com/cssthemes6/foun_8_Sinclair.zip"
    "http://down.cssmoban.com/cssthemes6/tmag_23_Infinity.zip"
    "http://down.cssmoban.com/cssthemes6/dash_1_simple.zip"
    "http://down.cssmoban.com/cssthemes6/inva_2_evolo.zip"
    "http://down.cssmoban.com/cssthemes6/oplv_9_stage.zip"
    "http://down.cssmoban.com/cssthemes6/oplv_2_html5updimension.zip"
    "http://down.cssmoban.com/cssthemes6/bpus_10_showtracker.zip"
    "http://down.cssmoban.com/cssthemes6/sstp_9_Typerite.zip"
    "http://down.cssmoban.com/cssthemes6/resu_1_designer-Portfolio.zip"
    "http://down.cssmoban.com/cssthemes6/wsdp_30_invictus.zip"
    "http://down.cssmoban.com/cssthemes6/zero_57_zMatcha.zip"
    "http://down.cssmoban.com/cssthemes6/zero_54_zHarvest.zip"
    "http://down.cssmoban.com/cssthemes6/zero_38_zPhotoGrap.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1927_dag.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1913_daq.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1893_dem.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1876_czx.zip"
    "http://down.cssmoban.com/cssthemes6/wsdp_20_union.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1873_cyq.zip"
    "http://down.cssmoban.com/cssthemes6/wsdp_11_lambda.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1861_cto.zip"
    "http://down.cssmoban.com/cssthemes6/fish_30_rapoo.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1796_cvc.zip"
    "http://down.cssmoban.com/cssthemes6/zero_21_zCreative.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1807_cti.zip"
    "http://down.cssmoban.com/cssthemes6/mzth_14_unicorn.zip"
    "http://down.cssmoban.com/cssthemes6/crui_1_agnes.zip"
    "http://down.cssmoban.com/cssthemes6/crui_3_ava.zip"
    "http://down.cssmoban.com/cssthemes6/cpts_1845_dcy.zip"
    "http://down.cssmoban.com/cssthemes6/tlip_5_material-app.zip"
    "http://down.cssmoban.com/cssthemes6/tlip_10_wedding.zip"
    "http://down.cssmoban.com/cssthemes6/quar_2_atlantis.zip"
    "http://down.cssmoban.com/cssthemes5/tope_11_steak.zip"
    "http://down.cssmoban.com/cssthemes1/azmind_1_xb.zip"
    "http://down.cssmoban.com/cssthemes4/ft5_67_simple.zip"
    "http://down.cssmoban.com/cssthemes3/dgfp_27_hm.zip"
    "http://down.cssmoban.com/cssthemes/frt_26.zip"
    "http://down.cssmoban.com/cssthemes1/ftmp_135_up.zip"
    "http://down.cssmoban.com/cssthemes6/mdbo_14_Landing-Page.zip"
    "http://down.cssmoban.com/cssthemes6/tmag_23_Infinity.zip"
    "http://down.cssmoban.com/cssthemes6/zero_39_zPinPin.zip"
    "http://down.cssmoban.com/cssthemes3/cpts_137_elv.zip"
    "http://down.cssmoban.com/cssthemes6/zero_50_zAnimal.zip"
    "http://down.cssmoban.com/cssthemes2/dgfp_12_cvh.zip"
    "http://down.cssmoban.com/cssthemes3/sbtp_2_fb.zip"
    "http://down.cssmoban.com/cssthemes3/npts_10_cvl.zip"
    "http://down.cssmoban.com/cssthemes1/ftmp_24.zip"
    "http://down.cssmoban.com/cssthemes4/dstp_28_P2.zip"
    "http://down.cssmoban.com/cssthemes5/tpmo_520_highway.zip"
    "http://down.cssmoban.com/cssthemes5/twts_168_awesplash.zip"
    "http://down.cssmoban.com/cssthemes6/wsdp_7_edge.zip"
    "http://down.cssmoban.com/cssthemes3/mstp_89_rock4life.zip"
    )
    RANDOM=$$$(date +%s)
    rand=$[$RANDOM % ${#webs[@]}]
    echo ${webs[$rand]}
}

# echo $(gen_ss_link_new $ssmethod $sspwd $domain 443 $v2rayPath $(urlencode 中文))
function gen_ss_link_new(){
    enc=$1
    pwd=$2
    host=$3
    port=$4
    path=$5
    mark=$6

    # -n flag used to remove newline
    # -w 0 ==>> --wrap=COLS Wrap encoded lines after COLS character (default 76). Use 0 to disable line wrapping.
    base64encode=$(echo -n "$enc:$pwd" | base64 -w 0)
    pluginStr="plugin=v2ray%3btls%3bhost%3d"$host"%3bpath%3d%2f"$path
    echo "ss://"$base64encode"@"$host":"$port"/?"$pluginStr"#"$mark
}

# 获取ip
function getIp(){
    echo `curl ifconfig.me`

    # 如果上面的方法失效，请放开下面的方法，用ifconfig获取外网ip（不一定准确）
    #name=""
    #if [ $# -ne 0 ];then
    #   name=$1
    #fi
    #hash ifconfig 2>/dev/null || {
    #    yum -y install net-tools 2 > /dev/null
    #}
    #r=`ifconfig $name | grep "inet.*broadcast.*" | cut -d' ' -f10`
    #echo $r
}

# 从域名获取IP
function get_ip_from_domain(){
    ADDR=$1
    IP=`ping ${ADDR} -c 1 | sed '1{s/[^(]*(//;s/).*//;q}'`
    echo ${IP}
}

# 安装相应工具软件
function install_tools(){
    hash docker 2>/dev/null || {
        # 安装docker
        wget -qO- https://get.docker.com/ | bash
    }
    # 启动并且自启
    systemctl restart docker && systemctl enable docker
    # install zip
    hash zip 2>/dev/null || {
        yum -y install zip
    }
    # install unzip
    hash unzip 2>/dev/null || {
        yum -y install unzip
    }
    # install wget if not exists
    hash wget 2>/dev/null || {
        yum -y install wget
    }
    # install ifconfig
    hash ifconfig 2>/dev/null || {
        yum -y install net-tools
    }
}



# 修改时间为东八区
function check_datetime(){
    # 覆盖系统时间
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    # 更新系统硬件时间
    hwclock
}

# 添加路径
function add_path(){
    path=$1
    name=$2
    # 添加链接
    curl https://$domain/$v2rayPath/add?$path 2>/dev/null
    # 生成分享链接
    echo "当前分享链接为:"
    echo ''
    export share=$(gen_ss_link_new $ssmethod $sspwd $domain 443 $path $(urlencode $name))
    echo $share
    echo ''
}

function add_mynet(){
    # 先清除所有使用该网络的容器才能删除该网络
    docker stop ngx 2>/dev/null
    docker rm ngx 2>/dev/null
    docker stop ss 2>/dev/null
    docker rm ss 2>/dev/null
    docker network rm mynet 2>/dev/null
    # 创建网络
    docker network create --subnet=172.18.0.0/16 mynet
}


function add_ngx(){
    # 清理一些资源，避免重复创建出现问题
    rm -rf /etc/letsencrypt
    rm -rf /etc/ngx
    rm -rf /usr/share/nginx
    docker stop ngx 2>/dev/null
    docker rm ngx 2>/dev/null
    # 创建ngx容器
    docker run --name ngx \
    --network mynet --ip ${nginxip} \
    --restart=always \
    -e v2rayPath=$v2rayPath \
    -e domain=$domain \
    -e ssipport=${ssip}:${ssport} \
    -p 80:80 -p 443:443 \
    -v /etc/letsencrypt:/etc/letsencrypt \
    -v /etc/ngx:/etc/ngx \
    -v /etc/localtime:/etc/localtime \
    -v /usr/share/nginx/html:/usr/share/nginx/html \
    -d kimoqi/nginx4ss 2>/dev/null
}

function domain_register(){
    # 域名认证
    # 自动输入A 1 2 -->> A为同意  1为选择第一个域名 2开启80端口跳转
    docker exec -it ngx /bin/sh -c '{ echo "A"; echo "1"; echo "2"; } | /usr/bin/certbot --nginx --register-unsafely-without-email'
}

function add_website(){
    # 设置站点模板
    path=/usr/share/nginx/html/
    # 删除原来的文件
    rm -rf ${path}*
    # 取出文件名
    filename=$(basename $website .zip)
    # 下载解压
    wget $website -O $path/tmp.zip && unzip $path/tmp.zip -d $path
    # 拷贝文件到外面
    cp -r $path/$filename/* $path
}

function add_ss(){
    # 清理一些资源，避免重复创建出现问题
    docker stop ss 2>/dev/null
    docker rm ss 2>/dev/null
    # 设置带有v2ray插件的shadowsocks
    docker run -d \
    --network mynet --ip ${ssip} \
    --name ss \
    --restart=always \
    -e "ARGS=--plugin v2ray-plugin --plugin-opts server;path=/$v2rayPath;loglevel=none" \
    -e PASSWORD=$sspwd \
    -e SERVER_PORT=$ssport \
    -e METHOD=$ssmethod \
    -v /etc/localtime:/etc/localtime \
    kimoqi/ssv2ray 2>/dev/null
}


function define_var(){
    export nginxip=172.18.0.2
    export ssip=172.18.0.3
    # ss使用端口，内网使用，保持默认即可
    export ssport=8388
}


function input_domain(){
    read -p "请输入域名：" inputdomain
    # 检查域名
    localip=$(getIp eth0)
    domainip=$(get_ip_from_domain ${inputdomain})
    if [[ "$localip" != "$domainip" ]];then
        echo "该域名 ${inputdomain}指向IP为 ${domainip}，非本机IP ${localip}，请在域名控制台将域名指向本机IP，脚本退出，再见！"
        exit 0
    fi
    # 定义域名
    export domain=$inputdomain
    echo '设置域名为 '${domain}
}


function input_v2rayPath(){
    read -p "请输入v2ray路径(输入为空则随机生成)：" inputpath
    if [[ "$inputpath" == "" ]];then
        inputpath=$(getRandomPwd 20)
    fi
    export v2rayPath=$inputpath
    echo '设置v2ray路径为 '${v2rayPath}
}


function input_sspwd(){
    read -p "请输入shadowsocks密码(输入为空则随机生成)：" inputsspwd
    if [[ "$inputsspwd" == "" ]];then
        inputsspwd=$(getRandomPwd 16)
    fi
    export sspwd=$inputsspwd
    echo '设置shadowsocks密码为 '${sspwd}
}


function input_ssmethod(){
    read -p "请输入shadowsocks加密方式(输入为空则默认为chacha20-ietf-poly1305，请务必输入ss支持的加密方式，否则连接不上！)：" inputssmethod
    if [[ "$inputssmethod" == "" ]];then
        inputssmethod=chacha20-ietf-poly1305
    fi
    export ssmethod=$inputssmethod
    echo '设置shadowsocks加密方式为 '${ssmethod}
}


function init_website(){
    read -p "请输入站点模板路径(只支持后缀名为.zip的文件)，为空将随机选择站点模板(注意:不知道怎么输入直接回车即可!)：" inputweb
    if [[ "$inputweb" == "" ]];then
        inputweb=$(get_random_webs)
    fi
    # 定义站点模板
    export website=$inputweb
    echo '设置站点模板为 '${website}
}



# 更新.bashrc
function update_bashrc(){
    cat >> ~/.bashrc<<EOF
export domain=${domain}
export v2rayPath=${v2rayPath}
export sspwd=${sspwd}
export ssmethod=${ssmethod}
export website=${website}
export share=${share}

EOF
. ~/.bashrc
}



function print_help(){
    echo '帮助信息:'
    echo ' -h 打印本信息'
    echo ' -i 安装'
    echo ' -w 更新伪装站点'
    echo "默认分享链接:"${share}
}


function install(){
    input_domain
    input_v2rayPath
    input_sspwd
    input_ssmethod
    init_website
    define_var
    echo "安装所需工具中 docker/zip/unzip/wget/ifconfig..."
    install_tools 2>/dev/null

    echo "调整时间为东八区中..."
    check_datetime 2>/dev/null

    echo "添加网络中..."
    add_mynet 2 > /dev/null

    echo "添加shadowsocks容器中..."
    add_ss

    echo "添加Nginx容器中..."
    add_ngx 2>/dev/null

    echo "认证域名开始..."
    sleep 2
    domain_register
    echo "认证域名成功..."

    echo "添加站点模板中..."
    add_website 2>/dev/null
    echo "添加站点模板成功!"

    print_help

    echo "添加分享链接中..."
    add_path $domain 默认路径

    update_bashrc
}


function update_website(){
    init_website
    add_website 2>/dev/null
    echo '设置完毕！'
}


# 调用入口
if [[ "$1" == "-h" ]];then
    # 帮助
    print_help
elif [[ "$1" == "-i" ]];then
    # 安装
    install
elif [[ "$1" == "-w" ]];then
    # 更新伪站
    update_website
else
    echo "没有这个选项${1}"
fi
```

## nginx4ss

这是脚本中用到的nginx的docker镜像，基于alpine，其中的start.sh实现了添加、删除路径、统计流量等功能

### DockerFile

```dockerfile
FROM nginx:1.17.5-alpine

ENV v2rayPath examplepath
ENV domain example.com

# shadowsocks ip and port
ENV ssipport 127.0.0.1:8388

COPY ./start.sh /root/start.sh

RUN \
apk add --no-cache --virtual .run-deps \
certbot-nginx \
python \
fcgiwrap \
openssh \
sshpass \
&& mkdir -p /etc/ngx/conf.d \
&& echo '0 0,12 * * * sleep 10 && /usr/bin/certbot renew >> /tmp/renew.log' >> /var/spool/cron/crontabs/root \
&& chmod 777 /root/start.sh

EXPOSE 80

ENTRYPOINT ["/root/start.sh"]
```

### start.sh

```bash
#!/bin/sh
echo 'starting.....'

rm -rf /var/run/fcgiwrap.socket
echo 'fcgiwrap.socket deled'

fcgiwrap -f -s unix:/var/run/fcgiwrap.socket &
echo 'fcgiwrap.socket started'

sleep 2 # wait fcgiwrap run 
chmod a+rw /var/run/fcgiwrap.socket
echo 'chmod fcgiwrap.socket complete'

# 创建脚本

# 添加
if [ ! -f "/etc/ngx/add" ]; then
echo 'creating /etc/ngx/add'
cat > /etc/ngx/add<<EOF
#!/bin/sh
echo -ne 'Status: 200\r\n'
echo -ne 'Content-Type: application/json\r\n'
echo -ne '\r\n'

path=\$QUERY_STRING
if [[ "\$path" == "" ]];then
    echo '{"rescode",-1}'
else 
    echo "rewrite ^/\$path\$ /$v2rayPath;" > /etc/ngx/conf.d/\$path.path
    nginx -s reload
    echo '{"rescode",0}'
fi
exit 0
EOF
fi


# 删除
if [ ! -f "/etc/ngx/del" ]; then
echo 'creating /etc/ngx/del'
cat > /etc/ngx/del<<EOF
#!/bin/sh

echo -ne 'Status: 200\r\n'
echo -ne 'Content-Type: application/json\r\n'
echo -ne '\r\n'

path=\$QUERY_STRING
rm -rf /etc/ngx/conf.d/\$path.path
nginx -s reload
echo '{"rescode",0}'
exit 0
EOF
fi

# 统计流量
if [ ! -f "/etc/ngx/statis" ]; then
echo 'creating /etc/ngx/statis'
cat > /etc/ngx/statis<<EOF
#!/bin/sh

echo -ne 'Status: 200\r\n'
echo -ne 'Content-Type: text/plain\r\n'
echo -ne '\r\n'

host=\$HTTP_HOST
log=/etc/ngx/\$host.log 
res=\`cat \$log | awk '{ print \$1";"\$7" "\$10 }' |awk -v h=\$host -v date="\$(date +"%Y-%m-%d %H:%M:%S")"  '{a[\$1]+=\$2}END{for (i in a)print i";"a[i]";"h";"date}' | sort -n -k1\`
echo "\${res}" >> statis.log
echo "\${res}"
echo '' > \$log
EOF
fi

# 赋权
echo 'chmod add del statis'
chmod 777 /etc/ngx/add
chmod 777 /etc/ngx/del
chmod 777 /etc/ngx/statis


# 创建Nginx配置文件
rm -rf /etc/nginx/conf.d/default.conf
if [ ! -f "/etc/nginx/conf.d/$domain.conf" ];then
cat > /etc/nginx/conf.d/$domain.conf<<EOF
server {
    listen       80;
    server_name  $domain;
    access_log  /etc/ngx/$domain.log  main;
    location / {
        include /etc/ngx/conf.d/*.path;
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    location /$v2rayPath {
        proxy_redirect off;
        proxy_pass http://$ssipport;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \$http_host;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    #add path script 添加路径脚本
    location /$v2rayPath/add {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/add";
        fastcgi_param       PATH_INFO           \$uri;
        fastcgi_param       QUERY_STRING        \$args;
        fastcgi_param       HTTP_HOST           \$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
    #del path script 删除路径脚本
    location /$v2rayPath/del {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/del";
        fastcgi_param       PATH_INFO           \$uri;
        fastcgi_param       QUERY_STRING        \$args;
        fastcgi_param       HTTP_HOST           \$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
    #statistic traffic script 流量统计脚本
    location /$v2rayPath/statis {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/statis";
        fastcgi_param       PATH_INFO           \$uri;
        fastcgi_param       QUERY_STRING        \$args;
        fastcgi_param       HTTP_HOST           \$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
}
EOF
echo 'nginx conf file init complete'
else 
echo 'nginx conf file has exists!'
fi

# 判断conf.d文件是否存在
if [ ! -d "/etc/ngx/conf.d" ]; then
    mkdir -p /etc/ngx/conf.d
    echo '/etc/ngx/conf.d created!'
else 
    echo '/etc/ngx/conf.d dir has exists!'
fi

# =====================================================gen_domain.sh脚本=====================================================
if [ ! -f "/etc/ngx/gen_domain.sh" ]; then
cat > /etc/ngx/gen_domain.sh<<EOFO
#!/bin/bash

# echo '' > /etc/ngx/gen_domain.sh && chmod +x /etc/ngx/gen_domain.sh && vi /etc/ngx/gen_domain.sh
newDmain=\$1
v2rayPath=\$2
cat > ~/\$newDmain.conf<<EOF
server {
    server_name  \$newDmain;
    access_log  /etc/ngx/\$newDmain.log  main;
    location / {
        include /etc/ngx/conf.d/*.path;  # 需要添加原来服务器下的路径信息
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #add path script 添加路径脚本
    location /\$v2rayPath/add {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/add";
        fastcgi_param       PATH_INFO           \\\$uri;
        fastcgi_param       QUERY_STRING        \\\$args;
        fastcgi_param       HTTP_HOST           \\\$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
    #del path script 删除路径脚本
    location /\$v2rayPath/del {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/del";
        fastcgi_param       PATH_INFO           \\\$uri;
        fastcgi_param       QUERY_STRING        \\\$args;
        fastcgi_param       HTTP_HOST           \\\$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
    #statistic traffic script 流量统计脚本
    location /\$v2rayPath/statis {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/statis";
        fastcgi_param       PATH_INFO           \\\$uri;
        fastcgi_param       QUERY_STRING        \\\$args;
        fastcgi_param       HTTP_HOST           \\\$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }

    location /\$v2rayPath {
        proxy_redirect off;
        proxy_pass http://$ssipport;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \\\$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \\\$http_host;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/\$newDmain/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/\$newDmain/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

# 301 REDIRECT
server {
    if (\\\$host = \$newDmain) {
        return 301 https://\\\$host\\\$request_uri;
    } # managed by Certbot
    listen       80;
    server_name  \$newDmain;
    return 404; # managed by Certbot
}
EOF
docker cp ~/\$newDmain.conf  ngx:/etc/nginx/conf.d/\$newDmain.conf
docker exec -i ngx nginx -s reload
rm -rf ~/\$newDmain.conf
echo "\$newDmain with \$v2rayPath has success!"
EOFO
fi
# =====================================================gen_domain.sh脚本=====================================================

sleep 1
echo 'crond started.....'
crond

sleep 1
echo 'nginx started....'
nginx -g "daemon off;"
```

## ssv2ray

这是脚本中用到的shadowsocks的镜像，含有v2ray插件，基于alpine

```dockerfile
FROM golang:alpine AS golang

ENV V2RAY_PLUGIN_VERSION v1.2.0
ENV GO111MODULE on

# Build v2ray-plugin
RUN apk add --no-cache git build-base \
    && mkdir -p /go/src/github.com/shadowsocks \
    && cd /go/src/github.com/shadowsocks \
    && git clone https://github.com/shadowsocks/v2ray-plugin.git \
    && cd v2ray-plugin \
    && git checkout "$V2RAY_PLUGIN_VERSION" \
    && go get -d \
    && go build

FROM alpine


ENV SHADOWSOCKS_LIBEV_VERSION v3.3.3

# Build shadowsocks-libev
RUN set -ex \
    # Install dependencies
    && apk add --no-cache --virtual .build-deps \
               autoconf \
               automake \
               build-base \
               libev-dev \
               libtool \
               linux-headers \
               udns-dev \
               libsodium-dev \
               mbedtls-dev \
               pcre-dev \
               tar \
               udns-dev \
               c-ares-dev \
               git \
    # Build shadowsocks-libev
    && mkdir -p /tmp/build-shadowsocks-libev \
    && cd /tmp/build-shadowsocks-libev \
    && git clone https://github.com/shadowsocks/shadowsocks-libev.git \
    && cd shadowsocks-libev \
    && git checkout "$SHADOWSOCKS_LIBEV_VERSION" \
    && git submodule update --init --recursive \
    && ./autogen.sh \
    && ./configure --disable-documentation \
    && make install \
    && ssRunDeps="$( \
        scanelf --needed --nobanner /usr/local/bin/ss-server \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk add --no-cache --virtual .ss-rundeps $ssRunDeps \
    && cd / \
    && rm -rf /tmp/build-shadowsocks-libev \
    # Delete dependencies
    && apk del .build-deps

# Copy v2ray-plugin
COPY --from=golang /go/src/github.com/shadowsocks/v2ray-plugin/v2ray-plugin /usr/local/bin

# Shadowsocks environment variables
ENV SERVER_ADDR 0.0.0.0
ENV SERVER_PORT 8388
ENV PASSWORD ChangeMe!!!
ENV METHOD chacha20-ietf-poly1305
ENV TIMEOUT 86400
ENV DNS_ADDRS 1.1.1.1,1.0.0.1
ENV ARGS -u

EXPOSE $SERVER_PORT/tcp $SERVER_PORT/udp

# Start shadowsocks-libev server
CMD exec ss-server \
    -s $SERVER_ADDR \
    -p $SERVER_PORT \
    -k $PASSWORD \
    -m $METHOD \
    -t $TIMEOUT \
    -d $DNS_ADDRS \
    --reuse-port \
    --no-delay \
    $ARGS
```



# 脚本源码AddDomain.sh

此脚本用于实现认证新域名的功能

```bash
#!/bin/bash


# 检查环境
function check_env(){
    hash docker 2>/dev/null || {
        echo "没有安装docker，退出！"
        exit 0
    }

    [ ! "$(docker ps  | grep ngx)" ] && {
	    echo 'nginx容器不存在或已经停止运行，退出'
	    exit 0
    }

    [ ! "$(docker ps  | grep ss)" ] && {
	    echo 'ss容器不存在或已经停止运行，退出'
	    exit 0
    }

}

# 从域名获取IP
function get_ip_from_domain(){
    ADDR=$1
    IP=`ping ${ADDR} -c 1 | sed '1{s/[^(]*(//;s/).*//;q}'`
    echo ${IP}
}

# 获取本机ip
function getIp(){
    name=""
    if [ $# -ne 0 ];then
        name=$1
    fi
    # install ifconfig
    hash ifconfig 2>/dev/null || {
        yum -y install net-tools 2 > /dev/null
    }
    r=`ifconfig $name | grep "inet.*broadcast.*" | cut -d' ' -f10`
    echo $r
}

# 输入域名
function input_domain(){
    read -p "请输入域名：" inputdomain
    # 检查域名
    localip=$(getIp eth0)
    domainip=$(get_ip_from_domain ${inputdomain})
    if [[ "$localip" != "$domainip" ]];then
        echo "该域名 ${inputdomain}指向IP为 ${domainip}，非本机IP ${localip}，请在域名控制台将域名指向本机IP，脚本退出，再见！"
        exit 0
    fi
    # 定义域名
    export domain=$inputdomain
    echo '设置域名为 '${domain}
}

# 输入v2ray path
function input_v2rayPath(){
    read -p "请输入v2ray根路径(即安装脚本所用的路径，不知道就执行cat ~/.bashrc 找到v2rayPath字段的值)：" inputpath
    if [[ "$inputpath" == "" ]];then
        echo "v2ray根路径不能为空，退出！"
        exit 0
    fi
    export v2rayPath=$inputpath
    echo '设置v2ray根路径为 '${v2rayPath}
}

# 获取ss容器的ip
function get_ss_ip(){
    export ssip=`docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'  ss`
}

# 生成配置文件
function gen_domain_conf(){
    echo "生成配置文件"$(pwd)/$domain.conf
    get_ss_ip
    cat > $(pwd)/$domain.conf<<EOF
server {
    listen       80;
    server_name  $domain;
    access_log  /etc/ngx/$domain.log  main;
    location / {
        include /etc/ngx/conf.d/*.path;  # 需要添加原来服务器下的路径信息
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    location /$v2rayPath {
        proxy_redirect off;
        proxy_pass http://$ssip:8388;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \$http_host;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    #add path script 添加路径脚本
    location /$v2rayPath/add {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/add";
        fastcgi_param       PATH_INFO           \$uri;
        fastcgi_param       QUERY_STRING        \$args;
        fastcgi_param       HTTP_HOST           \$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
    #del path script 删除路径脚本
    location /$v2rayPath/del {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/del";
        fastcgi_param       PATH_INFO           \$uri;
        fastcgi_param       QUERY_STRING        \$args;
        fastcgi_param       HTTP_HOST           \$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
    #statistic traffic script 流量统计脚本
    location /$v2rayPath/statis {
        fastcgi_param       SCRIPT_FILENAME     "/etc/ngx/statis";
        fastcgi_param       PATH_INFO           \$uri;
        fastcgi_param       QUERY_STRING        \$args;
        fastcgi_param       HTTP_HOST           \$server_name;
        fastcgi_pass        unix:/var/run/fcgiwrap.socket;
        include             fastcgi_params;
    }
}
EOF
    docker cp $(pwd)/$domain.conf  ngx:/etc/nginx/conf.d/$domain.conf
    docker exec -it ngx nginx -s reload
}


# 域名HTTPS证书注册
function domain_register(){
    # 域名认证
    # 自动输入A 1 2 -->> A为同意  1为选择第一个域名 2开启80端口跳转
    # docker exec -it ngx /bin/sh -c '{ echo "A"; echo "1"; echo "2"; } | /usr/bin/certbot --nginx --register-unsafely-without-email'
    docker exec -it ngx certbot --nginx --register-unsafely-without-email
}



function main(){
    # 检查环境
    check_env
    # 输入域名
    input_domain
    # 输入v2ray path
    input_v2rayPath
    # 生成配置文件
    gen_domain_conf
    # 注册域名（需要手动选择自己要注册的域名）
    domain_register
}

main
```



