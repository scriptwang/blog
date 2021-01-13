## JS实现图片懒加载
在实际的项目开发中，我们通常会遇见这样的场景：一个页面有很多图片，而首屏出现的图片大概就一两张，那么我们还要一次性把所有图片都加载出来吗？显然这是愚蠢的，不仅影响页面渲染速度，还浪费带宽。这也就是们通常所说的首屏加载，技术上现实其中要用的技术就是图片懒加载--到可视区域再加载。

## 思路
用data-xxx属性替换img标签的src属性，当页面滚动到图片区域时候再将src属性替换成data-xxx属性

## 各种宽高
```javascript
页可见区域宽： document.body.clientWidth;
网页可见区域高： document.body.clientHeight;
网页可见区域宽： document.body.offsetWidth (包括边线的宽);
网页可见区域高： document.body.offsetHeight (包括边线的宽);
网页正文全文宽： document.body.scrollWidth;
网页正文全文高： document.body.scrollHeight;
网页被卷去的高： document.body.scrollTop;
网页被卷去的左： document.body.scrollLeft;
网页正文部分上： window.screenTop;
网页正文部分左： window.screenLeft;
屏幕分辨率的高： window.screen.height;
屏幕分辨率的宽： window.screen.width;
屏幕可用工作区高度： window.screen.availHeight;
```

## 实现
[echo.js](https://github.com/toddmotto/echo)，不依赖外部插件，压缩后1kb

```html
<div class="demo">
    <!--src属性为空白图片，data-echo中为图片真实路径-->
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
    <img class="lazy" src="./img/blank.gif" data-echo="./img/bg.jpg">
</div>

<style>
	/*固定宽高*/
　　.demo img { 
　　　　width: 1024; 
　　　　height: 260px; 
　　　　background: url(images/loading.gif) 50% no-repeat;}
</style>

<script type="text/javascript" src="./echo.min.js"></script>
<script>
Echo.init({
    offset: 0,//离可视区域多少像素的图片可以被加载
　　 throttle: 100 //图片延时多少毫秒加载
}); 

</script>

```
## 注意事项
如果在AJAX中获取图片数据后使用懒加载，需要在AJAX请求成功后执行初始化，因为初始化是在有数据之后进行的
```javascript
$.ajax({
    type: 'GET',
    url: 'resources/js/frontmb/json/more.json',
    dataType: 'json',
    success: function(data){
        result =   '<a class="item opacity" href="'+data.lists[i].link+'">'
            +'<img src="'+basePath+'resources/image/common/blank.gif"  data-echo="'+data.lists[i].pic+'" alt="">'
            +'<h3>'+data.lists[i].title+'</h3>'
            +'<span class="date">'+data.lists[i].date+'</span>'
            +'</a>';
        $('.list').append(result);
        //等AJAX数据加载完成后再初始化
        Echo.init({
            offset: 0,//离可视区域多少像素的图片可以被加载
            throttle: 0 //图片延时多少毫秒加载
        });
    },
    error: function(xhr, type){
        alert('Ajax error!');
    }
});
        
```
