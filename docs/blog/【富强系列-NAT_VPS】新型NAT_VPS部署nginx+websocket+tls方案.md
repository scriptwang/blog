# NAT VPS简介
现在出现了一种新型的NAT VPS，原理是多人共享一个公网IPV4地址（每个人只允许用一部分端口），节省IPV4地址的开销，降低成本。

NAT VPS又分为两种，一种是允许NAT建站，一种不允许。NAT VPS租户可用端口范围一般为10000~60000，闲置了80/443 web服务默认端口，一些商家就把这些端口配置Nginx用来NAT建站。

NAT VPS有个很重要的优点！那就是自动换IP，一般NAT VPS用一个CNAME域名给你而不会直接给你IP，这意味着商家换IP对你没有任何影响！不像传统VPS，IP被和谐就GG。

所以未来如果NAT VPS大行其道可以将服务向其转移，但是也说不定，因为IPV6也在发展了。


# 原理
目前我见过的NAT VPS有两种虚拟化，一种是KVM，这个就不说了，还有一种是LXC，是系统级的Linux容器，[LXC里面安装ubuntu可完整支持docker](http://blog.zxh.site/2018/02/28/Docker-in-LXC%E8%B8%A9%E5%9D%91/)（docker是应用级的容器），可以这样理解：
商家服务器是一台物理机有公网ip的，LXC容器这台物理机生出来的小鸡，有自己的内网ip，而LXC里面docker跑着的应用又有自己的docker网桥内网ip，这样相当于从公网ip到你的应用实际上穿过了两层网关，是不是有点像内网穿透呢~

Ubuntu请使用下面命令安装docker

```
apt-get install docker.io
```


写此文时我只用过一家NAT VPS https://anyhk.net，刚开起来的，官网一直在宕机。。。

一图胜千言，相对于有公网IP的多了一层：
```
                                                                                                            -->> 其他path -->> 404/403(自定义)
                                                                                                            |
你的流量 -->> 你的CNAME域名 -->> 商家域名 -->> 商家服务器(Nginx:80/443 HTTPS) -->> NAT VPS -->> Nginx:80 -->> -> 正常path -->> 网站(伪站)
                                                                                                            |
                                                                                                            -->> 代理path -->> 代理根path -->> 代理工具 --->> 和谐站点
```
- 商家域名 -->> 商家服务器(Nginx:80/443 HTTPS)：**如果商家换IP对你没有任何影响！**
- 商家服务器(Nginx:80/443 HTTPS)：此处公网IP的80/443为商家所用，**在这一层商家可以为你的域名开启HTTPS证书认证**（相当于TLS在这里实现了，不需要自己再认证了，看商家支持情况），他可以反向代理你的NAT小鸡（注意这里在商家提供的面板上设置，而不是你的NAT小鸡上），注意这里的配置要开启Nginx的websocket协议，如下（反向代理）：

```nginx
#PROXY-START/
location /
{
    proxy_pass http://10.0.20.140:80; # 假设你的NAT小鸡内网ip为10.0.20.140，Nginx端口为80
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header REMOTE-HOST $remote_addr;
    
    # 此处很重要！一定要有下面四行，代表升级http协议为websocket协议
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";

}

#PROXY-END/
```
-  NAT VPS -->> Nginx:80：此处就不用在认证域名了，只需要提供80端口（任意端口，随你指定）出去让商家 80/443 反代即可。

这套方案从头到尾实际上就开了一个端口用于ssh到NAT小鸡登录而已（一般NAT小鸡商家会提供20个端口供你你映射），代理其实用的是商家提供公网IP的80/443（准确的说是443）端口在工作。

# 实现方案
这是有独立公网IP的实现方式：https://github.com/scriptwang/SSoverTLS/blob/master/sh/SSoverTLS_Adv.sh
在原来的基础之上不需要nginx的域名认证，简化后大概进行如下操作：（需要docker Ubuntu请使用```apt-get install docker.io```安装）

```bash
# 变量定义(根据实际情况改)
v2rayPath=v2rayRootPath   #代理根path
sspwd=ss_password         #ss密码

# 常量定义(不用改)
ssport=8388               #默认ss端口
ssmethod=chacha20-ietf-poly1305  #默认加密方式
domain=localhost          #默认主机，因为是主机商Nginx反代，所以域名是localhost


# 修改时间为东八区
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock

# 创建ss
docker run -d \
--name ss \
--restart=always \
-e "ARGS=--plugin v2ray-plugin --plugin-opts server;path=/$v2rayPath;loglevel=none" \
-e PASSWORD=$sspwd \
-e SERVER_PORT=$ssport \
-e METHOD=$ssmethod \
-v /etc/localtime:/etc/localtime \
kimoqi/ssv2ray 

# 获取ss ip
ssip=`docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'  ss`

# 创建Nginx
docker run --name ngx \
--restart=always \
-e v2rayPath=$v2rayPath \
-e domain=$domain \
-e ssipport=${ssip}:${ssport} \
-p 80:80 \
-v /etc/letsencrypt:/etc/letsencrypt \
-v /etc/ngx:/etc/ngx \
-v /etc/localtime:/etc/localtime \
-v /usr/share/nginx/html:/usr/share/nginx/html \
-d kimoqi/nginx4ss
```

安装一个伪站（非必须），需要wget，unzip等工具，没有自行安装
```bash
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

# 获取随机站点
website=$(get_random_webs)

# 添加站点
add_website

```

