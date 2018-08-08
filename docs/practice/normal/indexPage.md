# indexPage

设计思想:
菜单页面作为功能的入口,因此采取完整 `html` 格式编写,页面渲染部分用 `beetl` 模板引擎绑定,如果您想了解更多语法,请访问[beetl 在线文档][beetl-online-doc]查看;

基本规则如下:

- `beetl` 模板除了标准的 `html` 标签外,还支持自定义标签 `<#tagName />` ,例如 `<#btnGroup onResetClick="reset()" onSearchClick="search()" ></#btnGroup>`;
- 如果需要处理功能权限,可以使用 `<%if(condition){statement}%>` 进行条件渲染;


1. 找到 `role` 角色页面,并复制重命名为 `vehicleType`,同时打开两个窗口,方便编辑;
![vehicleType-split-1][vehicleType-split-1]
![vehicleType-split-2][vehicleType-split-2]

2. 根据实际情况,编写 `vehicle` 目录下面的 `index.html` 页面;
![vehicleType-split-3][vehicleType-split-3]

3. 选中 `车辆类型管理` 菜单,右键 `新标签页` 打开,方便下一步 `js` 调试;
![vehicleType-index-debug][vehicleType-index-debug]

4. 最终效果如下;
![vehicleType-index-preview][vehicleType-index-preview]


[beetl-online-doc]: http://ibeetl.com/guide/#beetl "beetl-online-doc"
[vehicleType-split-1]: ../../../static/image/vehicleType-split-1.png "vehicleType-split-1"
[vehicleType-split-2]: ../../../static/image/vehicleType-split-2.png "vehicleType-split-2"
[vehicleType-split-3]: ../../../static/image/vehicleType-split-3.png "vehicleType-split-3"
[vehicleType-index-preview]: ../../../static/image/vehicleType-index-preview.png "vehicleType-index-preview"
[vehicleType-index-debug]: ../../../static/image/vehicleType-index-debug.png "vehicleType-index-debug"
[vehicleType-index-html]: ../../../static/image/vehicleType-index-html.png "vehicleType-index-html"
