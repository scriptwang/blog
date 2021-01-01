其实我是一名后端开发者（笑），事情是这样的，每次改页面或js的时候都要去IDE里面更新一下，页面才能生效，我用的IDE是IDEA，众所周知IDEA必须选择exploded包改动静态资源并update source才能生效，麻烦....

于是，作为一名自力更生的开发者就在思考：既然Chrome能修改js文件，为何不在第一次修改后将文件存在本地，然后再次请求的时候把本地文件拿出来作为response返回，这样不就能在线修改静态资源了吗？

Chrome当然是可以这样的（March 19, 2018更新的，也就是说这个时间点更新后的Chrome才有此功能），但是如果你懂点前端，你就可以随意更改js逻辑，破解网页限制，所以安全性不仅在前端要做，更重要的是要在后端做验证，如果你的服务挂在外网还没有后端数据验证的话，呵呵o(*￣︶￣*)o
# 开启Chrome Local Overrides
1. 打开Chrome调试工具（如果打开调试工具发现不能修改js文件则用ctrl + shift + i试试）
2. 切换到Sources面板
3. 找到左上角的Overrides Tab，点击Select folder for overrides，给Chrome指定一个目录存放修改的资源
4. 此时Chrome会有权限询问，点击允许
5. 回到Sources面板，修改js并ctrl+s保存，然后刷新页面试试！


# 引用
> Local Overrides let you make changes in DevTools, and keep those changes across page loads. Previously, any changes that you made in DevTools would be lost when you reloaded the page. Local Overrides work for most file types

> How it works:

> You specify a directory where DevTools should save changes. When you make changes in DevTools, DevTools saves a copy of the modified file to your directory.
> When you reload the page, DevTools serves the local, modified file, rather than the network resource.
> To set up Local Overrides:

> 1. Open the Sources panel.
> 2. Open the Overrides tab.
> 3. Click Setup Overrides.
> 4. Select which directory you want to save your changes to.
> 5. At the top of your viewport, click Allow to give DevTools read and write access to the directory.
> Make your changes.

# 参考
https://stackoverflow.com/questions/16494237/chrome-dev-tools-modify-javascript-and-reload
https://developers.google.com/web/updates/2018/01/devtools#overrides
