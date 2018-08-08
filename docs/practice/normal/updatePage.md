# updatePage

设计思想:

更新页面作为菜单页面功能的一部分,采用内页窗口模式弹出,但本质上是独立的页面,因此采用尽量简单方式开发更新页面;

1. 找到 `role` 目录下面的 `update.html` ,和 `vehicleType` 目录下面的 `update.html`,打开左右两个窗口,同时对比查看;
![vehicleType-split-update-html][vehicleType-split-update-html]

2. 根据实际情况,编写 `vehicleType` 目录下面的 `update.html` ;
![vehicleType-update-debug][vehicleType-update-debug]

3. 还未修改 `update.js` ,因此 `update.html` 页面请求可能不正确显示,效果如图;
![vehicleType-update-html][vehicleType-update-html]

[vehicleType-split-update-html]: ../../../static/image/vehicleType-split-update-html.png "vehicleType-split-update-html"
[vehicleType-update-html]: ../../../static/image/vehicleType-update-html.png "vehicleType-update-html"
[vehicleType-update-debug]: ../../../static/image/vehicleType-update-debug.png "vehicleType-update-debug"
