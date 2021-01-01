# 浏览器js做简单分页（JQuery）
- 注意，正经需求千万不要用js在浏览器分页，数据太多浏览器压力会很大，实测4W张图片浏览器卡死，如果不想加班老老实实在服务器端做分页吧！

- 此js分页适用于数据量较小的情况

- 使用：

  1. 引入js文件，参考https://www.jq22.com/jquery-info10344

  2. 按文件内结构引入相关资源 

  3. 创建div 

  4. 在此div上初始化分页

- 更多模板访问http://www.jq22.com
```html
<div class="zxf_pagediv"></div>
<script type="text/javascript">
	/*
	 * js实现分页，适用于数据div被外层div包含的情况
	 * <div id="masonry">
	 *     <div src="data"/>
	 *     <div src="data"/>
	 *     <div src="data"/>
	 *     ...
	 * </div>
	 */
	var len = 12;//定义每页长度
	var $wrapper = $("div#" + "masonry");

	/*
	 * 业务逻辑
	 */
	var all = $wrapper.find("div").length;//所有记录数
	var cnt = Math.ceil(all / len);.//总页数
	var pre = 1;//记录之前的页数
	//
	function show(page,len,attr) {
		var start = (page-1) * len;
		var end = start + len;
		for (var i = start ; i < end;i++)
			$wrapper.find("div:eq(" + i + ")").attr("style","display:"+attr+";");
	}
	//show(1,all,'none');//关闭所有div显示，性能更好一点应该是服务器端在每个div上加display:none
	show(1,len,'block');//显示第一页
	//翻页
	$(".zxf_pagediv").createPage({
		pageNum: cnt,
		current: 1,
		backfun: function(e) {
			if (e.current < 1 || e.current > cnt){
				alert('当前页数不存在，回到首页！');
				e.current = 1;
			}
			show(pre,len,'none');//隐藏上一页
			show(e.current,len,'block');//显示当前页
			pre = e.current;//记录当前页
		}
	});
</script>
```



