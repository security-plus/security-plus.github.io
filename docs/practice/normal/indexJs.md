# indexJs

设计思想:
考虑到后续项目可能会采用 `vuejs` 框架,前后端分离,因此当前 `js` 层面采用类[vuejs][vuejs]语法设计模式;

基本规则如下:

- `data` 数据层,进行数据绑定,存放公共变量;
- `method` 方法层,绑定公共方法,方便统一管理;
- `plugins` 插件层,第三方插件配置项;


1. 找到 `role` 目录下面的 `index.js` ,和 `vehicleType` 目录下面的 `index.js`,打开左右两个窗口,同时对比查看;
![vehicleType-split-index-js][vehicleType-split-index-js]

2. 根据实际情况,编写 `vehicleType` 目录下面的 `index.js` ;
![vehicleType-index-js][vehicleType-index-js]

3. 调试完成,`index.html` 数据请求正常显示,效果如图;
![vehicleType-index-debug][vehicleType-index-debug]

4. 删除测试,默认 `无子节点` 允许删除,`有子节点` 禁用删除;
![vehicleType-delete-success][vehicleType-delete-success]
![vehicleType-delete-failure][vehicleType-delete-failure]

index.js

```js
/**
 * 首页实例对象
 *
 * @author snowdreams1006
 * @date 2018-08-07
 */
var indexInstance = {
    data: {
        layerIndex: -1
    },
    method: {
        /**
         * 初始化分页表格显示字段
         */
        initColumns: function () {

            var columns = [
                {
                    radio: true
                },
                {
                    title: "序号",
                    titleTooltip: "序号",
                    visible: true,
                    showSelectTitle: true,
                    formatter: function (value, row, index, field) {
                        try {
                            var bsTableInstance = indexInstance.plugins.bootstrapTable.bsTableInstance;
                            var pageSize = bsTableInstance.bootstrapTable('getOptions').pageSize;
                            var pageNumber = bsTableInstance.bootstrapTable('getOptions').pageNumber;
                            return pageSize * (pageNumber - 1) + index + 1;
                        } catch (error) {
                            Feng.log(error);
                            Feng.info("分页表格尚未实例化,无法刷新!");

                            return index + 1;
                        }
                    }
                },
                {
                    field: "id",
                    title: "id",
                    titleTooltip: "车辆类型id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "name",
                    title: "名称",
                    titleTooltip: "车辆类型名称",
                    sortable: true,
                    visible: true,
                    showSelectTitle: true
                },
                {
                    field: "num",
                    title: "排序",
                    titleTooltip: "排序",
                    sortable: true,
                    visible: true,
                    showSelectTitle: true
                },
                {
                    field: "type",
                    title: "类型",
                    titleTooltip: "类型:1表示牵引车,2表示半挂车,3表示全挂车",
                    sortable: true,
                    visible: true,
                    showSelectTitle: true,
                    formatter: function (value, row, index, field) {
                        if (value == 1) {
                            return '<i class="fa fa-institution"></i>';
                        }
                        if (value == 2) {
                            return '<i class="fa fa-user"></i>';
                        }
                        return '<i class="fa fa-question"></i>';
                    }
                },
                {
                    field: "remark",
                    title: "备注",
                    titleTooltip: "车辆类型备注",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "state",
                    title: "状态",
                    titleTooltip: "状态:1表示启用中,2表示已冻结,3表示已删除",
                    sortable: true,
                    visible: true,
                    showSelectTitle: true,
                    formatter: function (value, row, index, field) {
                        if (value == 1) {
                            return '<i class="fa fa-eye"></i>';
                        }
                        if (value == 2) {
                            return '<i class="fa fa-lock"></i>';
                        }
                        if (value == 3) {
                            return '<i class="fa fa-ban"></i>';
                        }
                        return '<i class="fa fa-question"></i>';
                    }
                },
                {
                    field: "ownerEnterpriseId",
                    title: "所有者企业id",
                    titleTooltip: "所有者企业id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "ownerEnterpriseName",
                    title: "所有者企业名称",
                    titleTooltip: "所有者企业名称",
                    sortable: false,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "ownerUserId",
                    title: "所有者用户id",
                    titleTooltip: "所有者用户id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "ownerUserName",
                    title: "所有者用户名称",
                    titleTooltip: "所有者用户名称",
                    sortable: false,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "createEnterpriseId",
                    title: "创建企业id",
                    titleTooltip: "创建企业id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "createEnterpriseName",
                    title: "创建企业名称",
                    titleTooltip: "创建企业名称",
                    sortable: false,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "modifiedEnterpriseId",
                    title: "更新企业id",
                    titleTooltip: "更新企业id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "modifiedEnterpriseName",
                    title: "更新企业名称",
                    titleTooltip: "更新企业名称",
                    sortable: false,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "createUserId",
                    title: "创建用户id",
                    titleTooltip: "创建用户id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "createUserName",
                    title: "创建用户名称",
                    titleTooltip: "创建用户名称",
                    sortable: false,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "modifiedUserId",
                    title: "更新用户id",
                    titleTooltip: "更新用户id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "modifiedUserName",
                    title: "更新用户名称",
                    titleTooltip: "更新用户名称",
                    sortable: false,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "tenantId",
                    title: "租户id",
                    titleTooltip: "租户id",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "tenantType",
                    title: "租户类型",
                    titleTooltip: "租户类型,1表示企业租户,2表示个人租户",
                    sortable: false,
                    visible: false,
                    showSelectTitle: true,
                    formatter: function (value, row, index, field) {
                        if (value == 1) {
                            return '<i class="fa fa-institution"></i>';
                        }
                        if (value == 2) {
                            return '<i class="fa fa-user"></i>';
                        }
                        return '<i class="fa fa-question"></i>';
                    }
                },
                {
                    field: "tenantName",
                    title: "租户名称",
                    titleTooltip: "租户名称",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "gmtCreate",
                    title: "创建时间",
                    titleTooltip: "创建时间",
                    sortable: true,
                    visible: false,
                    showSelectTitle: true
                },
                {
                    field: "gmtModified",
                    title: "更新时间",
                    titleTooltip: "更新时间",
                    sortable: true,
                    visible: true,
                    showSelectTitle: true
                }
            ];

            return columns;
        },
        /**
         * 初始化
         */
        init: function () {
            //初始化异步树
            var ztreeSetting = {};
            ztreeSetting = $.extend(true, ztreeSetting, Default.ztree.setting, indexInstance.plugins.ztree.setting);
            indexInstance.plugins.ztree.ztreeInstance = $.fn.zTree.init($("#asyncTree"), ztreeSetting);

            //初始化分页表格显示字段
            indexInstance.plugins.bootstrapTable.setting.columns = indexInstance.method.initColumns();

            //初始化分页表格
            var bootstrapTableSetting = {};
            bootstrapTableSetting = $.extend(true, bootstrapTableSetting, Default.bootstrapTable.setting, indexInstance.plugins.bootstrapTable.setting);
            indexInstance.plugins.bootstrapTable.bsTableInstance = $('#vehicleTypeTable').bootstrapTable(bootstrapTableSetting);

            //初始化搜索建议
            var bsSuggestSetting = {};
            bsSuggestSetting = $.extend(true, bsSuggestSetting, Default.bsSuggest.setting, indexInstance.plugins.bsSuggest.setting);
            $('#nameSuggest').bsSuggest(bsSuggestSetting);

            //初始化日期范围
            var datepickerSetting = {};
            datepickerSetting = $.extend(true, datepickerSetting, Default.datepicker.setting, indexInstance.plugins.datepicker.setting);
            $('.input-daterange').datepicker(datepickerSetting);
        },
        /**
         * 获取ztree选中记录
         */
        getZtreeSelectedNodes: function () {
            var ztreeSelectNodes = [];
            try {
                //获取 zt 已选中的节点列表
                var ztreeInstance = indexInstance.plugins.ztree.ztreeInstance;
                ztreeSelectNodes = ztreeInstance.getSelectedNodes();

                return ztreeSelectNodes;
            } catch (error) {
                Feng.log(error);
                Feng.info("异步树尚未实例化,无法获取数据!");

                return ztreeSelectNodes;
            }
        },
        /**
         * 获取bsTable选中记录
         */
        getBsTableSelections: function () {
            var bsTableSelections = [];
            try {
                var bsTableInstance = indexInstance.plugins.bootstrapTable.bsTableInstance;
                bsTableSelections = bsTableInstance.bootstrapTable('getSelections');

                return bsTableSelections;
            } catch (error) {
                Feng.log(error);
                Feng.info("分页表格尚未实例化,无法获取数据!");

                return bsTableSelections;
            }
        },
        /**
         * 从 bs 和 zt 已选列表中获取唯一 id
         */
        getSingleSelectedId: function () {
            var selectedId = null;

            //获取 bs 表格选中记录
            var bsTableSelections = indexInstance.method.getBsTableSelections();

            // 获取 zt 树选中记录
            var ztreeSelectNodes = indexInstance.method.getZtreeSelectedNodes();

            //优先 bs 表格记录,其次 zt 树记录
            if (bsTableSelections.length == 0 && ztreeSelectNodes.length == 1) {
                selectedId = ztreeSelectNodes[0].id;
            } else if (bsTableSelections.length == 1) {
                selectedId = bsTableSelections[0].id;
            }

            return selectedId;
        },
        /**
         * GET 请求跳转到新增车辆类型页面
         */
        addPage: function () {
            var index = Feng.openWindow({
                id: "vehicleType_add.page",
                title: "新增车辆类型",
                content: Feng.ctxPath + "/system/vehicleType/add.page"
            });
            indexInstance.data.layerIndex = index;
        },
        /**
         * GET 请求跳转到更新车辆类型页面
         */
        updatePage: function () {
            var selectedId = indexInstance.method.getSingleSelectedId();
            if (!selectedId) {
                return Feng.info("请先选中某一条记录再操作~");
            }
            var index = Feng.openWindow({
                id: "vehicleType_update.page",
                title: "更新车辆类型",
                content: Feng.ctxPath + "/system/vehicleType/update.page/" + selectedId
            });
            indexInstance.data.layerIndex = index;
        },
        /**
         * DELETE 请求删除车辆类型
         */
        deleteJson: function () {
            var selectedId = indexInstance.method.getSingleSelectedId();
            if (!selectedId) {
                return Feng.info("请先选中某一条记录再操作~");
            }
            Feng.confirm("是否刪除选中数据?", {
                title: "删除确认"
            }, function (index, layero) {
                $.ajax({
                    type: "DELETE",
                    url: Feng.ctxPath + "/system/vehicleType/delete.json/" + selectedId,
                    contentType: "application/json",
                    dataType: "json",
                    success: function (data, textStatus) {
                        indexInstance.method.refresh();
                        Feng.success("删除成功!");
                    },
                    error: function (XMLHttpRequest, textStatus, errorThrown) {
                        Feng.error(XMLHttpRequest.responseJSON.message);
                    }
                });
            });
        },
        /**
         * 刷新 ztree
         */
        refreshZtree: function () {
            try {
                var ztreeInstance = indexInstance.plugins.ztree.ztreeInstance;

                //重置初始化节点
                ztreeInstance.reAsyncChildNodes(null, "refresh", true);
            } catch (error) {
                Feng.log(error);
                Feng.info("异步树尚未实例化,无法刷新!");

                location.reload(true);
            }
        },
        /**
         * 刷新 BsTable
         */
        refreshBsTable: function () {
            try {
                var bsTableInstance = indexInstance.plugins.bootstrapTable.bsTableInstance;

                //重置默认分页信息
                bsTableInstance.bootstrapTable('refreshOptions', {
                    pageNumber: 1,
                    pageSize: 10
                });

                //重置视图高度
                bsTableInstance.bootstrapTable('resetView', {
                    height: $(window).height()
                });
            } catch (error) {
                Feng.log(error);
                Feng.info("分页表格尚未实例化,无法刷新!");

                location.reload(true);
            }
        },
        /**
         *  刷新
         */
        refresh: function () {
            try {
                //刷新zTree
                indexInstance.method.refreshZtree();

                //刷新bsTable
                indexInstance.method.refreshBsTable();
            } catch (error) {
                Feng.log(error);
                Feng.info("异步树或分页表格尚未实例化,无法刷新!");

                location.reload(true);
            }
        },
        /**
         * 重置
         */
        reset: function () {
            //清空表单
            $("#queryForm").clearForm(true);

            //刷新
            indexInstance.method.refresh();
        },
        /**
         * 搜索
         */
        search: function () {
            try {
                //结合自定义条件刷新表格
                var bsTableInstance = indexInstance.plugins.bootstrapTable.bsTableInstance;
                var searchParamObj = $("#queryForm").serializeObject();
                var queryData = {
                    query: searchParamObj
                };
                bsTableInstance.bootstrapTable('refresh', queryData);
            } catch (error) {
                Feng.log(error);
                Feng.info("分页表格尚未实例化,无法刷新!");

                location.reload(true);
            }
        }
    },
    plugins: {
        ztree: {
            setting: {
                async: {
                    url: Feng.ctxPath + "/system/vehicleType/asyncTree.json",
                    autoParam: function (treeId, treeNode) {
                        if (!treeNode) {
                            return [];
                        } else {
                            return ["id"];
                        }
                    }
                },
                callback: {
                    onClick: function (event, treeId, treeNode, clickFlag) {
                        //设置当前选中父节点
                        $("#id").val(treeNode.id);
                        indexInstance.method.search();
                    }
                }
            },
            ztreeInstance: null
        },
        bootstrapTable: {
            setting: {
                url: Feng.ctxPath + '/system/vehicleType/listPage.json',
                queryParams: function (params) {
                    var searchParamObj = $("#queryForm").serializeObject();
                    searchParamObj = $.extend(searchParamObj, params);
                    return searchParamObj;
                },
                sortName: "num",
                sortOrder: "asc",
                idField: "id",
                uniqueId: "id",
                toolbar: "#vehicleTypeTableToolbar",
                columns: []
            },
            bsTableInstance: null
        },
        bsSuggest: {
            setting: {
                url: Feng.ctxPath + "/system/vehicleType/listCondition.json",
                idField: "id",
                keyField: "name",
                showHeader: true,
                effectiveFields: ["name"],
                effectiveFieldsAlias: {
                    name: "车辆类型名称"
                },
                searchFields: ["name"],
                fnProcessData: function (responseData) {
                    if (!responseData || responseData.code != 0) {
                        return false;
                    }
                    var i, len, data = {
                        value: []
                    };
                    len = responseData.data.length;
                    for (i = 0; i < len; i++) {
                        data.value.push({
                            "id": responseData.data[i].id,
                            "name": responseData.data[i].name
                        });
                    }
                    return data;
                },
                fnAdjustAjaxParam: function (keyword, opts) {
                    return {
                        data: {
                            name: keyword
                        }
                    };
                }
            }
        },
        datepicker: {
            setting: {
                todayBtn: false,
                clearBtn: true
            }
        }
    }
};

/**
 * ready 初始化
 */
$(document).ready(function () {
    //初始化
    indexInstance.method.init();
});
```

[vuejs]: https://cn.vuejs.org/v2/guide/ "vuejs"
[vehicleType-split-index-js]: ../../../static/image/vehicleType-split-index-js.png "vehicleType-split-index-js"
[vehicleType-index-debug]: ../../../static/image/vehicleType-index-debug.png "vehicleType-index-debug"
[vehicleType-index-js]: ../../../static/image/vehicleType-index-js.png "vehicleType-index-js"
[vehicleType-delete-success]: ../../../static/image/vehicleType-delete-success.png "vehicleType-delete-success"
[vehicleType-delete-failure]: ../../../static/image/vehicleType-delete-failure.png "vehicleType-delete-failure"