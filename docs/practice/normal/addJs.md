# addJs

1. 找到 `role` 目录下面的 `add.js` ,和 `vehicleType` 目录下面的 `add.js`,打开左右两个窗口,同时对比查看;
![vehicleType-split-add-js][vehicleType-split-add-js]

2. 根据实际情况,编写 `vehicleType` 目录下面的 `add.js` ;
![vehicleType-add-debug][vehicleType-add-debug]

3. 调试完成, `add.html` 页面正确显示,效果如图;
![vehicleType-add-html][vehicleType-add-html]

add.js

```js
/**
 * 新增实例对象
 *
 * @author snowdreams1006
 * @date 2018-08-07
 */
var addInstance = {
    data: {
        isAdmin: false
    },
    method: {
        /**
         * 初始化新增同步树
         */
        initAddSyncTree: function (onSuccess) {
            //同步获取必须传入回调函数
            if (!onSuccess || !$.isFunction(onSuccess)) {
                return Feng.error("请传入成功回调函数");
            }

            //同步获取新增同步树
            $.ajax({
                url: Feng.ctxPath + "/system/vehicleType/syncTree.json",
                contentType: "application/json",
                dataType: "json",
                type: "GET",
                success: function (responseData, textStatus) {
                    if (responseData.code == 0) {
                        var root = responseData.data;

                        onSuccess(root);
                    } else {
                        Feng.info(responseData.message);
                    }
                },
                error: function (XMLHttpRequest, textStatus, errorThrown) {
                    Feng.error(XMLHttpRequest.responseJSON.message);
                }
            });
        },
        /**
         * 显示新增同步树
         */
        showAddSyncTree: function () {
            var parentIdNameDom = $("#parentIdName");

            $("#addMenuContent").css({
                left: parentIdNameDom.offset().left + "px",
                top: parentIdNameDom.offset().top + parentIdNameDom.outerHeight() + "px"
            }).slideDown("fast");

            $("body").bind("mousedown", addInstance.method.onBodyDown);
        },
        /**
         * body键盘按下隐藏新增同步树
         */
        onBodyDown: function (event) {
            var showingFlag = (event.target.id == "menuBtn" || event.target.id == "parentIdName" || event.target.id == "addMenuContent" || $(event.target).parents("#addMenuContent").length > 0);
            if (!showingFlag) {
                addInstance.method.hideAddSyncTree();
            }
        },
        /**
         * 隐藏新增同步树
         */
        hideAddSyncTree: function (event) {
            $("#addMenuContent").fadeOut("fast");

            $("body").unbind("mousedown", addInstance.method.onBodyDown);
        },
        /**
         * 初始化
         */
        init: function () {
            //获取用户基本信息
            addInstance.data.isAdmin = JSON.parse($("#isAdmin").val()) || false;

            //初始化类型
            var type = $("#type").data("radio-value");
            type && $('input:radio[name="type"][value="' + type + '"]').prop("checked", "checked");

            //初始化状态
            var state = $("#state").data("radio-value");
            state && $('input:radio[name="state"][value="' + state + '"]').prop("checked", "checked");

            //初始化隐藏状态的新增同步树
            addInstance.method.initAddSyncTree(function (root) {
                //初始化ztree设置
                var ztreeSetting = {};
                ztreeSetting = $.extend(true, ztreeSetting, Default.ztree.setting, addInstance.plugins.ztree.setting);
                //初始化新增同步树
                addInstance.plugins.ztree.ztreeInstance = $.fn.zTree.init($("#addSyncTree"), ztreeSetting, root);

                //初始化选中顶级车辆类型
                var ztreeInstance = addInstance.plugins.ztree.ztreeInstance;
                var rootNode = ztreeInstance.getNodeByParam("id", 0, null);
                if (rootNode) {
                    ztreeInstance.checkNode(rootNode, true, true, true);
                }
            });

            //初始化新增表单校验
            var validatorSetting = {};
            validatorSetting = $.extend(true, validatorSetting, addInstance.plugins.validator.setting);
            addInstance.plugins.validator.validatorInstance = $("#addForm").validate(validatorSetting);

            //初始化有"管理员"车辆类型的专属功能
            var isAdmin = addInstance.data.isAdmin;
            if (isAdmin) {
                //初始化管理员所有者企业搜索
                var ownerEnterpriseNameSetting = $.extend(true, {}, Default.bsSuggest.setting, addInstance.plugins.bsSuggest.ownerEnterpriseNameSetting);
                $("#ownerEnterpriseName").bsSuggest(ownerEnterpriseNameSetting).on('onSetSelectValue', function (e, selectedData, selectedRawData) {
                    //自动填充所有者企业 id
                    var ownerEnterpriseId = selectedRawData.id;

                    $("#ownerEnterpriseId").val(ownerEnterpriseId);
                });
                //初始化所有者企业校验
                $("#ownerEnterpriseName").rules("remove");
                $("#ownerEnterpriseName").rules("add", {
                    required: true,
                    minlength: 1,
                    maxlength: 50,
                    messages: {
                        required: "<i class='fa fa-times-circle'></i> 所有者企业名称不能为空",
                        minlength: "<i class='fa fa-times-circle'></i> 所有者企业名称至少1个字符",
                        maxlength: "<i class='fa fa-times-circle'></i> 所有者企业名称至多50个字符"
                    }
                });

                //初始化管理员所有者用户搜索
                var ownerUserNameSetting = $.extend(true, {}, Default.bsSuggest.setting, addInstance.plugins.bsSuggest.ownerUserNameSetting);
                $("#ownerUserName").bsSuggest(ownerUserNameSetting).on('onSetSelectValue', function (e, selectedData, selectedRawData) {
                    //自动填充所有者用户 id
                    var ownerUserId = selectedRawData.id;

                    $("#ownerUserId").val(ownerUserId);
                });
                //初始化所有者用户校验
                $("#ownerUserName").rules("remove");
                $("#ownerUserName").rules("add", {
                    required: true,
                    minlength: 1,
                    maxlength: 50,
                    messages: {
                        required: "<i class='fa fa-times-circle'></i> 所有者用户名称不能为空",
                        minlength: "<i class='fa fa-times-circle'></i> 所有者用户名称至少1个字符",
                        maxlength: "<i class='fa fa-times-circle'></i> 所有者用户名称至多50个字符"
                    }
                });

                //初始化租户类型并监听onchange事件
                var tenantType = $("#tenantType").data("radio-value");
                $('input:radio[name="tenantType"]').change(function () {
                    var selectType = parseInt($(this).val());
                    switch (selectType) {
                        case 1:
                            //更新管理员租户(企业租户)搜索
                            $("#tenantName").bsSuggest("destroy");
                            $("#tenantName").bsSuggest(ownerEnterpriseNameSetting).on('onSetSelectValue', function (e, selectedData, selectedRawData) {
                                //自动填充租户 id
                                var tenantId = selectedRawData.id;

                                $("#tenantId").val(tenantId);
                            });
                            break;
                        case 2:
                            //更新管理员租户(个人租户)搜索
                            $("#tenantName").bsSuggest("destroy");
                            $("#tenantName").bsSuggest(ownerUserNameSetting).on('onSetSelectValue', function (e, selectedData, selectedRawData) {
                                //自动填充租户 id
                                var tenantId = selectedRawData.id;

                                $("#tenantId").val(tenantId);
                            });
                            break;
                        default:
                            $("#tenantName").bsSuggest("disable");
                            Feng.log(selectType);
                    }
                });
                tenantType && $('input:radio[name="tenantType"][value="' + tenantType + '"]').prop("checked", "checked").trigger("change");
                //初始化租户校验
                $("#tenantName").rules("remove");
                $("#tenantName").rules("add", {
                    required: true,
                    minlength: 1,
                    maxlength: 50,
                    messages: {
                        required: "<i class='fa fa-times-circle'></i> 租户名称不能为空",
                        minlength: "<i class='fa fa-times-circle'></i> 租户用户名称至少1个字符",
                        maxlength: "<i class='fa fa-times-circle'></i> 租户用户名称至多50个字符"
                    }
                });
            }
        },
        /**
         * POST 请求新增车辆类型
         */
        addJson: function () {
            //校验通过再提交
            if (!$("#addForm").valid()) {
                return Feng.info("校验失败,请先核实数据!");
            }

            //表单序列化
            var addParamObj = $("#addForm").serializeObject();

            //新增
            $.ajax({
                url: Feng.ctxPath + "/system/vehicleType/add.json",
                contentType: "application/json",
                data: JSON.stringify(addParamObj),
                dataType: "json",
                processData: false,
                type: "POST",
                success: function (responseJson, textStatus) {
                    if (responseJson.code == 0) {
                        //父窗口更新后关闭当前页面
                        parent.indexInstance.method.refresh();
                        addInstance.method.cancel();

                        Feng.success("新增成功!");
                    } else {
                        Feng.info(responseJson.message);
                    }
                },
                error: function (XMLHttpRequest, textStatus, errorThrown) {
                    Feng.error(XMLHttpRequest.responseJSON.message);
                }
            });
        },
        /**
         * 关闭iframe新增页面
         */
        cancel: function () {
            parent.layer.close(parent.layer.getFrameIndex(window.name));
        }
    },
    plugins: {
        ztree: {
            setting: {
                callback: {
                    onClick: function (event, treeId, treeNode, clickFlag) {
                        //更新赋值
                        $("#parentIdName").attr("value", treeNode.name);
                        $("#parentId").val(treeNode.id);

                        //默认选中
                        var ztreeInstance = addInstance.plugins.ztree.ztreeInstance;
                        ztreeInstance.checkNode(treeNode, null, true, false);

                        //隐藏新增同步树
                        $("#addMenuContent").fadeOut("fast");
                    },
                    onCheck: function (event, treeId, treeNode) {
                        //更新赋值
                        $("#parentIdName").attr("value", treeNode.name);
                        $("#parentId").val(treeNode.id);

                        //默认选中
                        var ztreeInstance = addInstance.plugins.ztree.ztreeInstance;
                        ztreeInstance.selectNode(treeNode);

                        //隐藏新增同步树
                        $("#addMenuContent").fadeOut("fast");
                    }
                },
                check: {
                    enable: true,
                    chkStyle: "radio",
                    radioType: "all"
                }
            },
            ztreeInstance: null
        },
        validator: {
            setting: {
                errorPlacement: function (error, element) {
                    if (element.is(":radio") || element.is(":checkbox") || element.is(".bsSuggest-input") || element.parent().is(".input-group.date") || element.parent().is(".input-daterange.input-group")) {
                        error.appendTo(element.parent().parent());
                    } else {
                        error.appendTo(element.parent());
                    }
                },
                rules: {
                    type: {
                        required: true
                    },
                    name: {
                        required: true,
                        minlength: 1,
                        maxlength: 50
                    },
                    parentIdName: {
                        required: true
                    },
                    num: {
                        required: true,
                        min: 1
                    },
                    state: {
                        required: true
                    }
                },
                messages: {
                    type: {
                        required: "<i class='fa fa-times-circle'></i> 类型不能为空"
                    },
                    name: {
                        required: "<i class='fa fa-times-circle'></i> 名称不能为空",
                        minlength: "<i class='fa fa-times-circle'></i> 名称至少1个字符",
                        maxlength: "<i class='fa fa-times-circle'></i> 名称至多50个字符"
                    },
                    parentIdName: {
                        required: "<i class='fa fa-times-circle'></i> 上级车辆类型不能为空"
                    },
                    num: {
                        required: "<i class='fa fa-times-circle'></i> 排序不能为空",
                        min: "<i class='fa fa-times-circle'></i> 排序至少为1"
                    },
                    state: {
                        required: "<i class='fa fa-times-circle'></i> 状态不能为空"
                    }
                }
            },
            validatorInstance: null
        },
        bsSuggest: {
            ownerEnterpriseNameSetting: {
                url: Feng.ctxPath + "/system/enterprise/listCondition.json",
                idField: "id",
                keyField: "entName",
                showHeader: true,
                allowNoKeyword: false,
                effectiveFields: ["id", "entName"],
                effectiveFieldsAlias: {
                    id: "企业id",
                    entName: "企业名称"
                },
                searchFields: ["id", "entName"],
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
                            "entName": responseData.data[i].entName
                        });
                    }
                    return data;
                },
                fnAdjustAjaxParam: function (keyword, opts) {
                    return {
                        data: {
                            entName: keyword
                        }
                    };
                }
            },
            ownerUserNameSetting: {
                url: Feng.ctxPath + "/system/user/user_listCondition.json",
                idField: "id",
                keyField: "name",
                showHeader: true,
                allowNoKeyword: false,
                effectiveFields: ["id", "name"],
                effectiveFieldsAlias: {
                    id: "用户id",
                    name: "用户名称"
                },
                searchFields: ["id", "name"],
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
        }
    }
};

/**
 * ready 初始化
 */
$(document).ready(function () {
    //初始化
    addInstance.method.init();
});
```

[vehicleType-split-add-js]: ../../../static/image/vehicleType-split-add-js.png "vehicleType-split-add-js"
[vehicleType-add-html]: ../../../static/image/vehicleType-add-html.png "vehicleType-add-html"
[vehicleType-add-debug]: ../../../static/image/vehicleType-add-debug.png "vehicleType-add-debug"
