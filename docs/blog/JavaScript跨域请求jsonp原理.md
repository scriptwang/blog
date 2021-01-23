最近在做一个项目，a域名要向b域名发起ajax请求，普通的ajax请求会被浏览器拦截，如果是Chrome，可通过如下手段关闭浏览器的跨域。

Chrome关闭跨域
```
1. 新建目录用于Chrome保存个人信息C:\MyChromeDevUserData
2. 找到Chrome的快捷方式，右键属性，在目标字段后面加上
 --disable-web-security --user-data-dir=C:\MyChromeDevUserData
3. 重新打开Chrome，如果顶部有“--disable-web-security”提示，说明正确关闭了
4. 以上操作只在Chrome版本号49之后有效，49之前不需要添加--user-data-dir，直接加上--disable-web-security即可
```

当然，不可能要求每个客户都关闭浏览器跨域，于是就得用jsonp做跨域请求，注意，jsonp只支持get方式，这与jsonp的实现原理有关。

#### 什么是jsonp
JSONP(JSON with Padding)是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题。

#### 为什么使用jsonp
众所周知，这是因为同源策略。同源策略，它是由Netscape提出的一个著名的安全策略，现在所有支持JavaScript 的浏览器都会使用这个策略。

#### jsonp原理
众所周知，浏览器请求head里面的远程资源时是可以跨域的，比如：
```html
<html>
<head>
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">
</head>
</html>
```
此时，你就可以开心的试用bootstrap了，css可以，当然js也可以。如下
```html
<head>
    <script type="text/javascript" src="http://remoteserver.com/remote.js"></script>
</head>
<body>
 
</body>

```
http://remoteserver.com/remote.js内容如下：
```javascript
alert('我是remote.js');
```
毫无疑问，此时浏览器会弹窗。如果再写灵活一些
```html
<head>
    <script type="text/javascript">
    var localHandler = function(data){
        alert(data.res);
    };
    </script>
    <script type="text/javascript" src="http://remoteserver.com/remote.js"></script>
</head>
<body>
```
http://remoteserver.com/remote.js内容为
```javascript
localHandler({"res":"我是remote.js"});
```
毫无疑问，此时浏览器也会弹窗，但与上面最大的区别是，此时，通过script标签，把remote.js的数据带到了本地，因此我们发起请求的时候，可以动态的添加一个script标签，让跨域接口返回一段调用本地方法的js代码，里面包含我们需要的数据，这不就是jsonp吗？具体实现请看：（以下代码就是jsonp的核心，不管什么框架的jsonp，最终都是通过以上核心代码的调用实现跨域的。）
```html
<head>
    <script type="text/javascript">
    // 得到航班信息查询结果后的回调函数
    var flightHandler = function(data){
        alert('你查询的航班结果是：票价 ' + data.price + ' 元，' + '余票 ' + data.tickets + ' 张。');
    };
    // 提供jsonp服务的url地址（不管是什么类型的地址，最终生成的返回值都是一段javascript代码）
    var url = "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998&callback=flightHandler";
    // 创建script标签，设置其属性
    var script = document.createElement('script');
    script.setAttribute('src', url);
    // 把script标签加入head，此时调用开始
    document.getElementsByTagName('head')[0].appendChild(script); 
    </script>
</head>
<body>
 
</body>
```
http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998&callback=flightHandler可以返回这样一段数据
```javascript
flightHandler({
    "code": "CA1998",
    "price": 1780,
    "tickets": 5
});
```
当然flightHandler可以动态传给服务端，反正最终都是要返回一段调用本地方法的js代码。这就是为什么编写jQuery服务端的时候要写这样一句话
```java
return callback + "(" + relJsonData + ")";//callback是jQuery传给服务器的回调函数名称，相当于上面的flightHandler，relJsonData为真正的数据
```
#### jQuery对jsonp的封装
```html
<head>
      <script type="text/javascript" src=jquery.min.js"></script>
      <script type="text/javascript">
         jQuery(document).ready(function(){ 
            $.ajax({
                 type: "get",
                 async: false,
                 url: "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998",
                 dataType: "jsonp",
                 jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
                 success: function(json){
                     alert('您查询到航班信息：票价： ' + json.price + ' 元，余票： ' + json.tickets + ' 张。');
                 },
                 error: function(){
                     alert('fail');
                 }
             });
         });
     </script>
     </head>
  <body>
  </body>

```
jQuery会自动生成回调函数，并把数据取出来给success调用。虽然jQuery把jsonp归为ajax，但和ajax不是一回事儿。

#### 注意事项
1. jsonp不是ajax，本质上是不同的东西，ajax是通过XmlHttpRequest动态获取数据，jsonp是通过添加script标签动态获取数据
2. jsonp只能发送get请求，这是由其原理决定的，所以并没有jsonp+post
3. ajax返回的是数据，jsonp返回的是js函数调用（函数调用里面包含真正的数据）

#### 参考
https://blog.csdn.net/hansexploration/article/details/80314948