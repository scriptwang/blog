# OpenResty+Lua限流实战

当业务量越来越大的时候，为了能保证服务的运行，限流是必不可少的！OpenResty是一个高性能网关

> OpenResty® is a dynamic web platform based on NGINX and LuaJIT.

OpenResty = Nginx + Lua，Lua是高性能脚本语言，有着C语言的执行效率但是又比C简单，能很方便的扩展OpenResty 的功能。

> Lua 是由巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组于1993年开发的一种轻量、小巧的脚本语言，用标准 C 语言编写，其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。
>
> 官网：http://www.lua.org/

# 实战环境

docker + CentOS8 + Openresty 1.17.8.2

# Lua限流模块
https://github.com/openresty/lua-resty-limit-traffic

Lua的库一般都是小巧轻便且功能都具备，这个限流库核心文件一共就四个，几百行代码就能实现限流功能，Lua的其他库也是这样，比如redis的库还是Http的库，麻雀虽小五脏俱全！

# 环境准备

```bash
docker run -dit --name gw  --privileged centos /usr/sbin/init
docker exec -it gw bash 
```

在gw中

```bash
# 安装openresty
yum install -y yum-utils
yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
yum install -y openresty

# 安装工具等
yum install -y net-tools vim telnet git httpd

# Openresty自带了lua-resty-limit-traffic组件，如果没有带，下载到/usr/local/openresty/lualib/resty/limit/文件夹即可
# 下载lua-resty-limit-traffic组件
[ `ls /usr/local/openresty/lualib/resty/limit/ | wc -l` = 0 ] &&  echo '请安装限速组件' || echo '已经安装限速组件'
# 安装了请忽略
cd ~ && git clone https://github.com/openresty/lua-resty-limit-traffic.git
mkdir -p /usr/local/openresty/lualib/resty/limit/
cp  lua-resty-limit-traffic/lib/resty/limit/*.lua /usr/local/openresty/lualib/resty/limit/

# 启动openresy
openresty
```

# 限并发

场景：按照 ip 限制其并发连

参考：
https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx_lua/lua-limit.html
https://github.com/openresty/lua-resty-limit-traffic/blob/master/lib/resty/limit/conn.md
https://developer.aliyun.com/article/759299

原理：lua_share_dict是nginx所有woker和lua runtime共享的，当一个请求来，往lua_share_dict记录键值对`ip地址:1`，当请求完成时再-1，再来一个在+1，设置一个上限5，当超过5时则拒绝请求，一定要注意**内部重定向**的问题！

- [OpenResty执行阶段](参考文章/OpenResty执行阶段.md) tag：lua执行流程;执行阶段;openresty执行流程
- [为啥access_by_lua执行两次](参考文章/openresty why does access_by_lua_file call twice when accessing the root directory - Stack Overflow.md)

## 环境搭建

### 新建utils/limit_conn.lua模块

```bash
mkdir -p /usr/local/openresty/lualib/utils
cat > /usr/local/openresty/lualib/utils/limit_conn.lua <<EOF
-- utils/limit_conn.lua
local limit_conn = require "resty.limit.conn"

-- new 的第四个参数用于估算每个请求会维持多长时间，以便于应用漏桶算法
local limit, limit_err = limit_conn.new("limit_conn_store", 8, 2, 0.05)
if not limit then
    error("failed to instantiate a resty.limit.conn object: ", limit_err)
end

local _M = {}

function _M.incoming()
    local key = ngx.var.binary_remote_addr
    local delay, err = limit:incoming(key, true)
    if not delay then
        if err == "rejected" then
            return ngx.exit(503) -- 超过的请求直接返回503
        end
        ngx.log(ngx.ERR, "failed to limit req: ", err)
        return ngx.exit(500)
    end

    if limit:is_committed() then
        local ctx = ngx.ctx
        ctx.limit_conn_key = key
        ctx.limit_conn_delay = delay
    end

    if delay >= 0.001 then
        ngx.log(ngx.WARN, "delaying conn, excess ", delay,
                "s per binary_remote_addr by limit_conn_store")
        ngx.sleep(delay)
    end
end

function _M.leaving()
    local ctx = ngx.ctx
    local key = ctx.limit_conn_key
    if key then
        local latency = tonumber(ngx.var.request_time) - ctx.limit_conn_delay
        local conn, err = limit:leaving(key, latency)
        if not conn then
            ngx.log(ngx.ERR,
            "failed to record the connection leaving ",
            "request: ", err)
        end
    end
end

return _M

EOF
```

重点在于这句话`local limit, limit_err = limit_conn.new("limit_conn_store", 8, 2, 0.05)`，允许的最大并发为常规的8个，突发的2个，一共8+2=10个并发，详情参考https://github.com/openresty/lua-resty-limit-traffic/blob/master/lib/resty/limit/conn.md#new

被拒绝的请求直接返回503

```LUA
if err == "rejected" then
    return ngx.exit(503) -- 超过的请求直接返回503
end
```

### 修改nginx配置文件

```bash
# 备份一下配置文件
cd /usr/local/openresty/nginx/conf/ && \cp nginx.conf nginx.conf.bak

# 添加配置
echo '' > /usr/local/openresty/nginx/conf/nginx.conf
vim /usr/local/openresty/nginx/conf/nginx.conf
```

添加如下内容

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    lua_code_cache on;    
   
    # 注意 limit_conn_store 的大小需要足够放置限流所需的键值。
    # 每个 $binary_remote_addr 大小不会超过 16 字节(IPv6 情况下)，算上 lua_shared_dict 的节点大小，总共不到 64 字节。
    # 100M 可以放 1.6M 个键值对
    lua_shared_dict limit_conn_store 100M;
    
    server {
        listen 80;
        location / {
            access_by_lua_block {
                local limit_conn = require "utils.limit_conn"
                -- 对于内部重定向或子请求，不进行限制。因为这些并不是真正对外的请求。
                if ngx.req.is_internal() then
                    ngx.log(ngx.INFO,">> 内部重定向")
                    return
                end
                limit_conn.incoming()
                ngx.log(ngx.INFO,">>> 请求进来了！")
            }
            content_by_lua_block {
                -- 模拟请求处理时间，很重要，不加可能测试不出效果
                -- 生产中没有请求是只返回一个静态的index.html的！
                ngx.sleep(0.5)
            }
            log_by_lua_block {
                local limit_conn = require "utils.limit_conn"
                limit_conn.leaving()
                ngx.log(ngx.INFO,">>> 请求离开了！")
            }
            
        }
    }
}

```

重点在于这句话，模拟每个请求0.5秒处理完成

```nginx
content_by_lua_block {
	ngx.sleep(0.5)
}

```

注意在限制连接的代码里面，我们用 `ngx.ctx` 来存储 `limit_conn_key`。这里有一个坑。内部重定向（比如调用了 `ngx.exec`）会销毁 `ngx.ctx`，导致 `limit_conn:leaving()` 无法正确调用。 如果需要限连业务里有用到 `ngx.exec`，可以考虑改用 `ngx.var` 而不是 `ngx.ctx`，或者另外设计一套存储方式。只要能保证请求结束时能及时调用 `limit:leaving()` 即可。

### 重新加载配置文件

```bash
openresty -s reload 
```

## 测试

上面的配置是每个请求处理0.5秒，并发是10

- 10个请求，并发为1

```bash
ab -n 10 -c 1  127.0.0.1/

# 请求全部成功，用时5s左右
Concurrency Level:      1
Time taken for tests:   5.012 seconds 
Complete requests:      10 
Failed requests:        0

```

- 10个请求，并发为10

```bash
ab -n 10 -c 10  127.0.0.1/

# 请求全部成功，用时1.5s左右
Concurrency Level:      10
Time taken for tests:   1.505 seconds
Complete requests:      10
Failed requests:        0

```

- 20个请求，并发为10，并发为10并不会触发限制条件，所以能成功！注意和下面并发11的区别！

```bash
ab -n 20 -c 10  127.0.0.1/

# 请求全部成功，用时2s左右
Concurrency Level:      10
Time taken for tests:   2.005 seconds
Complete requests:      20
Failed requests:        0
```

- 22个请求，并发为11
  重点解释一下：
  - 并发不是qps，并发11不是说第一秒发11个请求，然后第二秒再发送11个请求，而是发完第一波紧接着发第二波，每一波的间隔时间不一定是1秒，下面的1.506 seconds就能看出来，按理应该是2s但是并不是
  
  - 第一波11个请求发送过去了，但是只能处理10个，所以成功了10个，紧接着第二波11个请求发过去了，但是第一波大部分未处理完成所以第二波的都失败了，也有处理完成了的可以接着处理，所以至少会成功10个，下面显示的是11个
  
  - 此处的大量失败应该是并发超过了10，触发了限制条件让nginx worker线程睡眠了，所以导致后面的请求大量失败
  
  - ```lua
    -- 触发限制条件
    if delay >= 0.001 then
        ngx.sleep(delay) -- ngx worker睡眠
    end
    ```
  
```bash
ab -n 22 -c 11  127.0.0.1/

# 11个成功，11个失败
Concurrency Level:      11
Time taken for tests:   1.506 seconds
Complete requests:      22
Failed requests:        11
Non-2xx responses:      11 # HTTP状态非2xx的有11个，说明限并发成功（只有有非2xx的返回才会显示这句话）
```

## 反向代理

上面测试的是`content_by_lua`，也就是内容直接在lua中生成，但是实际中内容有可能是后端服务器生成的，所以可以设置反向代理或者负载均衡，如下为反向代理配置

```nginx
location / {
    access_by_lua_block {
        local limit_conn = require "utils.limit_conn"
        -- 对于内部重定向或子请求，不进行限制。因为这些并不是真正对外的请求。
        if ngx.req.is_internal() then
        	return
        end
        limit_conn.incoming()
    }
    log_by_lua_block {
        local limit_conn = require "utils.limit_conn"
        limit_conn.leaving()
    }
    
    # 反向代理
    proxy_pass http://172.17.0.3:8080;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_connect_timeout 60;
    proxy_read_timeout 600;
    proxy_send_timeout 600;

}

```



## 内部重定向

```nginx
location / {
  access_by_lua_block {...}
  content_by_lua_block {...}
  log_by_lua_block {...}
}
```

nginx是按照阶段来执行指令的，和配置文件顺序没有关系，nginx是先执行`access_by_lua_block`，再执行`content_by_lua_block`，最后执行`log_by_lua_block`的，当在访问`curl 127.0.0.1/`时，如果没有`content_by_lua_block`，这里有一个**内部重定向**，会将127.0.0.1/的请求重定向到`127.0.0.1/index.html`，所以会按顺序再次执行`access_by_lua_block`，所以`access_by_lua_block`执行了两次，`log_by_lua_block`却执行了一次，当时的我十分懵逼，而**加上`content_by_lua`或者`proxy_pass`则不会导致重定向**，总之有内容来源时不会重定向，没有则会去找`index.html`导致重定向！

测试

```bash
vim /usr/local/openresty/nginx/conf/nginx.conf

# 修改成如下内容
server {
  listen 80;
  location / {
      access_by_lua_block {
          ngx.log(ngx.ERR,">>> access")
      }
      log_by_lua_block {
          ngx.log(ngx.ERR,">>> log")
      }
  }
}

# 查看日志
tail -f /usr/local/openresty/nginx/logs/error.log
```

- 测试`curl 127.0.0.1`日志输出如下
  `access_by_lua_block`执行了两次，并且页面上的内容是`index.html`的内容，说明发生了重定向
  如果加上`index.html`，即`curl 127.0.0.1/index.html`，则不会发生重定向

```
...[lua] access_by_lua(nginx.conf:24):2: >>> access, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1"
...[lua] access_by_lua(nginx.conf:24):2: >>> access, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1"
...[lua] log_by_lua(nginx.conf:27):2: >>> log while logging request, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1"
```

- 加上content_by_lua则访问http://127.0.0.1不会发生重定向

## lua初始化

这句话`local limit_conn = require "utils.limit_conn"`，`limit_conn`中的`local limit, limit_err = limit_conn.new("limit_conn_store", 8, 2, 0.05)`只会初始化一次，之后都是用的都一个实例，不会每个请求进来都要new一个`limit_conn`有点浪费性能而且还把参数都重置了，是不可取的，所以封装到了`utils.limit_conn`中！

# 限制接口时间窗请求数(非平滑)

场景：限制 ip 每1s只能调用 10 次（允许在时间段开始的时候一次性放过10个请求）也就是说，速率不是固定的

也可以设置成别的，比如120/min，只需要修改个数和时间窗口（`resty.limit.count`和`resty.limit.req`区别在于：前者传入的是个数，后者传入的是速率）

## 新建utils/limit_count.lua模块

```lua
mkdir -p /usr/local/openresty/lualib/utils
cat > /usr/local/openresty/lualib/utils/limit_count.lua <<EOF
-- utils/limit_count.lua
local limit_count = require "resty.limit.count"

-- rate:  10/s
local lim, err = limit_count.new("my_limit_count_store", 10, 1) -- 第二个参数次数，第三个参数时间窗口，单位s
if not lim then
    ngx.log(ngx.ERR, "failed to instantiate a resty.limit.count object: ", err)
    return ngx.exit(500)
end

local _M = {}


function _M.incoming()
    local key = ngx.var.binary_remote_addr
    local delay, err = lim:incoming(key, true)
    if not delay then
        if err == "rejected" then
            ngx.header["X-RateLimit-Limit"] = "10"
            ngx.header["X-RateLimit-Remaining"] = 0
            return ngx.exit(503) -- 超过的请求直接返回503
        end
        ngx.log(ngx.ERR, "failed to limit req: ", err)
        return ngx.exit(500)
    end
    
    -- 第二个参数是指定key的剩余调用量
    local remaining = err

    ngx.header["X-RateLimit-Limit"] = "10"
    ngx.header["X-RateLimit-Remaining"] = remaining

end

return _M

EOF
```

## 修改nginx配置文件

```bash
echo '' > /usr/local/openresty/nginx/conf/nginx.conf
vim /usr/local/openresty/nginx/conf/nginx.conf
```

添加如下内容

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    lua_code_cache on;    
   
    lua_shared_dict my_limit_count_store 100M;
    
    # resty.limit.count 需要resty.core
    init_by_lua_block {
		require "resty.core"
	}
    
    server {
        listen 80;
        location / {
            access_by_lua_block {
                local limit_count = require "utils.limit_count"
                -- 对于内部重定向或子请求，不进行限制。因为这些并不是真正对外的请求。
                if ngx.req.is_internal() then
                    return
                end
                limit_count.incoming()
            }            
            
            content_by_lua_block {
                ngx.sleep(0.1)
                ngx.say('Hello')
            }
            # 如果内容源是反向代理
            #proxy_pass http://172.17.0.3:8080;
            #proxy_set_header Host $host;
            #proxy_redirect off;
            #proxy_set_header X-Real-IP $remote_addr;
            #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #proxy_connect_timeout 60;
            #proxy_read_timeout 600;
            #proxy_send_timeout 600;

        }
    }
}

```

## 重新加载配置文件

```bash
openresty -s reload 
```

## 测试

上面的配置是10/s，不叠加

- 10个请求，并发为10，1s内完成

```bash
ab -n 10 -c 10  127.0.0.1/

# 请求全部成功
Concurrency Level:      10
Time taken for tests:   0.202 seconds
Complete requests:      10
Failed requests:        0


```

- 20个请求，并发为20，1s内完成

```bash
ab -n 20 -c 20  127.0.0.1/

# 请求成功10个，其余全部失败
Concurrency Level:      20
Time taken for tests:   0.202 seconds
Complete requests:      20
Failed requests:        10
   (Connect: 0, Receive: 0, Length: 10, Exceptions: 0)
Non-2xx responses:      10


```

- 查看请求头`curl -I 127.0.0.1`，可以看到接口限流信息

```properties

HTTP/1.1 200 OK
Server: openresty/1.17.8.2
Date: Sat, 12 Sep 2020 09:46:06 GMT
Content-Type: application/octet-stream
Connection: keep-alive
X-RateLimit-Limit: 10 # 当前限制10个
X-RateLimit-Remaining: 9 # 剩余9个

```

# 限制接口时间窗请求数(平滑)

## 桶（无容量）

场景：限制 ip 每1min只能调用 120次（平滑处理请求，即每秒放过2个请求），速率是固定的，并且桶没有容量(容量为0)

### 新建utils/limit_req_bucket.lua模块

```lua
mkdir -p /usr/local/openresty/lualib/utils
cat > /usr/local/openresty/lualib/utils/limit_req_bucket.lua <<EOF
-- utils/limit_req_bucket.lua
local limit_req = require "resty.limit.req"

-- rate:  2/s即为120/min，burst设置为0，也就是没有桶容量，超过的都拒绝（rejected）
local lim, err = limit_req.new("my_limit_req_store", 2, 0)
if not lim then
    ngx.log(ngx.ERR, "failed to instantiate a resty.limit.req object: ", err)
    return ngx.exit(500)
end

local _M = {}


function _M.incoming()
    local key = ngx.var.binary_remote_addr
    local delay, err = lim:incoming(key, true)
    if not delay then
        if err == "rejected" then
            return ngx.exit(503) -- 超过的请求直接返回503
        end
        ngx.log(ngx.ERR, "failed to limit req: ", err)
        return ngx.exit(500)
    end
end

return _M

EOF
```

### 修改nginx配置文件

```bash
echo '' > /usr/local/openresty/nginx/conf/nginx.conf
vim /usr/local/openresty/nginx/conf/nginx.conf
```

添加如下内容

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    lua_code_cache on;    
   
    lua_shared_dict my_limit_req_store 100M;
    
    server {
        listen 80;
        location / {
            access_by_lua_block {
                local limit_count = require "utils.limit_req_bucket"
                -- 对于内部重定向或子请求，不进行限制。因为这些并不是真正对外的请求。
                if ngx.req.is_internal() then
                    return
                end
                limit_count.incoming()
            }            
            
            content_by_lua_block {
                ngx.sleep(0.1)
                ngx.say('Hello')
            }
            # 如果内容源是反向代理
            #proxy_pass http://172.17.0.3:8080;
            #proxy_set_header Host $host;
            #proxy_redirect off;
            #proxy_set_header X-Real-IP $remote_addr;
            #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #proxy_connect_timeout 60;
            #proxy_read_timeout 600;
            #proxy_send_timeout 600;

        }
    }
}

```

### 重新加载配置文件

```bash
openresty -s reload 
```

### 测试

上面的配置是2/s即为120/min

- 请求时间限制为1s

```bash
ab -t 1 127.0.0.1/

# 实际请求1.1s，成功3个请求，符合预期
Time taken for tests:   1.100 seconds
Complete requests:      8656
Failed requests:        8653
   (Connect: 0, Receive: 0, Length: 8653, Exceptions: 0)
Non-2xx responses:      8653



```

- 请求时间限制为5s

```bash
ab -t 5 127.0.0.1/

# 实际请求5.1s，成功11个请求，符合预期
Concurrency Level:      1
Time taken for tests:   5.100 seconds
Complete requests:      40054
Failed requests:        40043
   (Connect: 0, Receive: 0, Length: 40043, Exceptions: 0)
Non-2xx responses:      40043
```

## 漏桶（有桶容量）

场景：限制 ip 每1min只能调用 120次（平滑处理请求，即每秒放过2个请求），速率是固定的，并且桶的容量有容量（设置burst）

### 新建utils/limit_req_leaky_bucket.lua模块

只需要在**桶（无容量）**的基础之上增加burst的值即可，并且增加delay的处理

```lua
mkdir -p /usr/local/openresty/lualib/utils
cat > /usr/local/openresty/lualib/utils/limit_req_leaky_bucket.lua <<EOF
-- utils/limit_req_leaky_bucket.lua
local limit_req = require "resty.limit.req"

-- rate:  2/s即为120/min，增加桶容量为1/s，超过2/s不到(2+1)/s的delay，排队等候，这就是标准的漏桶
local lim, err = limit_req.new("my_limit_req_store", 2, 1)
if not lim then
    ngx.log(ngx.ERR, "failed to instantiate a resty.limit.req object: ", err)
    return ngx.exit(500)
end

local _M = {}


function _M.incoming()
    local key = ngx.var.binary_remote_addr
    local delay, err = lim:incoming(key, true)
    if not delay then
        if err == "rejected" then
            return ngx.exit(503) -- 超过的请求直接返回503
        end
        ngx.log(ngx.ERR, "failed to limit req: ", err)
        return ngx.exit(500)
    end
    
    -- 此方法返回，当前请求需要delay秒后才会被处理，和他前面对请求数
    -- 所以此处对桶中请求进行延时处理，让其排队等待，就是应用了漏桶算法
    -- 此处也是与令牌桶的主要区别
    if delay >= 0.001 then
        ngx.sleep(delay)
    end
end

return _M

EOF


```

### 修改nginx配置文件

```bash
echo '' > /usr/local/openresty/nginx/conf/nginx.conf
vim /usr/local/openresty/nginx/conf/nginx.conf
```

添加如下内容

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    lua_code_cache on;    
   
    lua_shared_dict my_limit_req_store 100M;
    
    server {
        listen 80;
        location / {
            access_by_lua_block {
                local limit_count = require "utils.limit_req_leaky_bucket"
                -- 对于内部重定向或子请求，不进行限制。因为这些并不是真正对外的请求。
                if ngx.req.is_internal() then
                    return
                end
                limit_count.incoming()
            }            
            
            content_by_lua_block {
                -- 模拟每个请求的耗时
                ngx.sleep(0.1)
                ngx.say('Hello')
            }
            # 如果内容源是反向代理
            #proxy_pass http://172.17.0.3:8080;
            #proxy_set_header Host $host;
            #proxy_redirect off;
            #proxy_set_header X-Real-IP $remote_addr;
            #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #proxy_connect_timeout 60;
            #proxy_read_timeout 600;
            #proxy_send_timeout 600;

        }
    }
}

```

### 重新加载配置文件

```bash
openresty -s reload 
```

### 测试

上面的配置是2/s，漏桶容量为1/s，即总共3/s，模拟的每个请求耗时为0.1s，那么1s内能处理至少10个请求

- 请求时间限制为1s

```bash
ab -t 1 127.0.0.1/

# 实际请求1.102s，成功3个请求，1s两个请求，一个是delay，符合预期
Time taken for tests:   1.103 seconds
Complete requests:      3
Failed requests:        0
```

## 令牌桶

场景：限制 ip 每1min只能调用 120次（平滑处理请求，即每秒放过2个请求），但是允许一定的突发流量（突发的流量，就是桶的容量（桶容量为60），超过桶容量直接拒绝

令牌桶其实可以看着是漏桶的**逆操作**，看我们对把超过请求速率而进入桶中的请求如何处理，如果是我们把这部分请求放入到**等待队列**中去，那么其实就是用了漏桶算法，但是如果我们允许**直接处理这部分的突发请求**，其实就是使用了令牌桶算法。

这边只要将上面漏桶算法关于桶中请求的延时处理的代码修改成直接送到后端服务就可以了，这样便是使用了令牌桶

### 新建utils/limit_req_token_bucket.lua模块

```lua
mkdir -p /usr/local/openresty/lualib/utils
cat > /usr/local/openresty/lualib/utils/limit_req_token_bucket.lua <<EOF
-- utils/limit_req_token_bucket.lua
local limit_req = require "resty.limit.req"

-- rate:  2/s即为120/min，增加桶容量为60/s，超过2/s不到(2+60)/s的突发流量直接放行
local lim, err = limit_req.new("my_limit_req_store", 2, 60)
if not lim then
    ngx.log(ngx.ERR, "failed to instantiate a resty.limit.req object: ", err)
    return ngx.exit(500)
end

local _M = {}


function _M.incoming()
    local key = ngx.var.binary_remote_addr
    local delay, err = lim:incoming(key, true)
    if not delay then
        if err == "rejected" then
            return ngx.exit(503) -- 超过的请求直接返回503
        end
        ngx.log(ngx.ERR, "failed to limit req: ", err)
        return ngx.exit(500)
    end
    
    if delay >= 0.001 then
        -- 不做任何操作，直接放行突发流量
        -- ngx.sleep(delay)
    end
end

return _M

EOF
```

### 修改nginx配置文件

```bash
echo '' > /usr/local/openresty/nginx/conf/nginx.conf
vim /usr/local/openresty/nginx/conf/nginx.conf
```

添加如下内容

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    lua_code_cache on;    
   
    lua_shared_dict my_limit_req_store 100M;
    
    server {
        listen 80;
        location / {
            access_by_lua_block {
                local limit_count = require "utils.limit_req_token_bucket"
                -- 对于内部重定向或子请求，不进行限制。因为这些并不是真正对外的请求。
                if ngx.req.is_internal() then
                    return
                end
                limit_count.incoming()
            }            
            
            content_by_lua_block {
                -- 模拟每个请求的耗时
                ngx.sleep(0.1)
                ngx.say('Hello')
            }
            # 如果内容源是反向代理
            #proxy_pass http://172.17.0.3:8080;
            #proxy_set_header Host $host;
            #proxy_redirect off;
            #proxy_set_header X-Real-IP $remote_addr;
            #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #proxy_connect_timeout 60;
            #proxy_read_timeout 600;
            #proxy_send_timeout 600;

        }
    }
}

```

### 重新加载配置文件

```bash
openresty -s reload 
```

### 测试

上面模拟的每个请求耗时为0.1s，那么1s内能处理至少10个请求

- 时间限制为1s

```bash
ab -n 10 -c 10  -t 1 127.0.0.1/

# 实际请求1s，成功13个请求，可以看到是远远超过2个请求的，多余就是在处理突发请求
Concurrency Level:      10
Time taken for tests:   1.000 seconds
Complete requests:      12756
Failed requests:        12743
   (Connect: 0, Receive: 0, Length: 12743, Exceptions: 0)
Non-2xx responses:      12743

```

# 组合各种limter

上面的三种限速器conn、count、req可以进行各种组合，比如一个限速器是限制主机名的，一个是限制ip的，可以组合起来使用

参考：https://github.com/openresty/lua-resty-limit-traffic/blob/master/lib/resty/limit/traffic.md