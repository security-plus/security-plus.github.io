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

vehicleType

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="renderer" content="webkit"/>
    <title>SP-车辆类型</title>
    <%include("/layout/_css.html"){}%>
</head>

<body class="gray-bg">
    <div class="wrapper wrapper-content animated fadeIn">
        <div class="row">
            <div class="col-sm-12">
                <div class="ibox">
                    <div class="ibox-content">
                        <div class="row">
                            <div class="col-sm-3">
                                <div class="panel panel-primary">
                                    <div class="panel-heading">
                                        <i class="fa fa-tree"></i> 车辆类型树
                                    </div>
                                    <div class="panel-body">
                                        <ul id="asyncTree" class="ztree"></ul>
                                    </div>
                                </div>
                            </div>

                            <div class="col-sm-9">
                                <div class="panel panel-info">
                                    <div class="panel-heading">
                                        <h5 class="panel-title">
                                            <a data-toggle="collapse" data-parent="#pageAccordion" href="#pageCollapse">
                                                <i class="fa fa-list"></i> 车辆类型列表
                                            </a>
                                        </h5>
                                    </div>
                                    <div class="panel-body">
                                        <div class="row">
                                            <div id="pageAccordion" class="col-sm-12">
                                                <div id="pageCollapse" class="panel-collapse collapse in">
                                                    <form id="queryForm" class="form-horizontal">
                                                        <#input id="id" name="id" type="hidden"/>
                                                        <div class="col-sm-4">
                                                            <#bsSuggest id="nameSuggest" name="name" placeholder="请输入车辆类型名称" />
                                                        </div>
                                                        <div class="col-sm-4">
                                                            <#daterangepicker startDateId="startDate" startDateName="startDate" endDateId="endDate" endDateName="endDate"/>
                                                        </div>
                                                        <div class="col-sm-4">
                                                            <#btnGroup onResetClick="reset()" onSearchClick="search()" ></#btnGroup>
                                                        </div>
                                                    </form>
                                                </div>
                                            </div>
                                        </div>

                                        <div class="row">
                                            <div class="col-sm-12">
                                                <div class="btn-group hidden-xs" id="vehicleTypeTableToolbar" role="group">
                                                    <%if(shiroTools.hasPermission("/system/vehicleType/add.page")){%>
                                                    <button type="button" class="btn btn-outline btn-success" onclick="indexInstance.method.addPage()">
                                                        <i class="glyphicon glyphicon-plus" aria-hidden="true"></i>
                                                    </button>
                                                    <%}%>

                                                    <%if(shiroTools.hasPermission("/system/vehicleType/update.page/{id}")){%>
                                                    <button type="button" class="btn btn-outline btn-warning" onclick="indexInstance.method.updatePage()">
                                                        <i class="glyphicon glyphicon-edit" aria-hidden="true"></i>
                                                    </button>
                                                    <%}%>

                                                    <%if(shiroTools.hasPermission("/system/vehicleType/delete.json/{id}")){%>
                                                    <button type="button" class="btn btn-outline btn-danger" onclick="indexInstance.method.deleteJson()">
                                                        <i class="glyphicon glyphicon-trash" aria-hidden="true"></i>
                                                    </button>
                                                    <%}%>
                                                </div>
                                                <#bsTable modulePrefix="vehicleType" />
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <%include("/layout/_js.html"){}%>
    <script src="${ctxPath}/js/module/system/vehicleType/index.js"></script>
</body>

</html>
```

[beetl-online-doc]: http://ibeetl.com/guide/#beetl "beetl-online-doc"
[vehicleType-split-1]: ../../../static/image/vehicleType-split-1.png "vehicleType-split-1"
[vehicleType-split-2]: ../../../static/image/vehicleType-split-2.png "vehicleType-split-2"
[vehicleType-split-3]: ../../../static/image/vehicleType-split-3.png "vehicleType-split-3"
[vehicleType-index-preview]: ../../../static/image/vehicleType-index-preview.png "vehicleType-index-preview"
[vehicleType-index-debug]: ../../../static/image/vehicleType-index-debug.png "vehicleType-index-debug"
[vehicleType-index-html]: ../../../static/image/vehicleType-index-html.png "vehicleType-index-html"
