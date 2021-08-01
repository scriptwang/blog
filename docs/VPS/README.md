本系列是对本人使用过的VPS进行评测，主要包括全国时延测试（ping值）、`LemonBenchIntl`脚本测试、`superbench`脚本测试

## 全国时延测试

在站长工具找到Ping检测，输入VPS的IP即可进行全国测试，运营商包括移动、联通和电信

- https://ping.chinaz.com/

## `superbench`脚本

脚本地址：https://git.io/superbench.sh

测试内容包括系统基本信息、硬盘IO性能测试、网络性能测试

使用方法

```
wget -qO- git.io/superbench.sh | bash
```

为避免脚本失效，此处备份一份，有需要的老铁自取：[`superbench.sh`](assets/README/superbench.sh)



## `LemonBenchIntl`脚本

脚本地址：https://ilemonra.in/LemonBenchIntl 

测试内容十分丰富，包括系统信息、网络信息、媒体解锁测试、CPU性能测试、内存性能测试、硬盘性能测试、网络性能测试、路由（IPv4/IPv6）测试

使用方法 

```bash
curl -fsL https://ilemonra.in/LemonBenchIntl | bash -s fast
```

为避免脚本失效，此处备份一份，有需要的老铁自取：[`LemonBenchIntl.sh`](assets/README/LemonBenchIntl.sh)

该脚本支持多种模式，如`fast`、`full`、`mediaunlocktest`，具体请看脚本的入口函数

## 去程/回程路由测试

去程指的是本地到VPS的路由信息，可以用`besttrace`这款工具查询，这是一款客户端工具，支持全平台，此处以Windows为例

- Windows：[点此下载](https://cdn.ipip.net/17mon/besttrace.exe)


下载安装，输入VPS的IP后点击里面的**路由追踪**即可

当然也提供了网页版，输入IP就可以测试，网页版有个优势是可以选择不同的监测点

- https://tools.ipip.net/traceroute.php

---

回程测试指的就是VPS到本地或者到国内监测点的路由测试，`LemonBenchIntl`脚本已经做了该测试，可结合起来看

## 评测归档

- [2021-08-01_DMIT香港双程CN2评测](2021-08-01_DMIT香港双程CN2评测.md)

