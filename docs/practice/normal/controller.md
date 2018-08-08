# controller

基本规则如下:

- `controller` 继承统一基类 `AbstractController` ,可根据实际情况决定是否重写某些方法;
- 类名：`大驼峰命名+Controller` 规范,默认注释业务模型,并加上 `@Controller @RequestMapping @Api` 等注解说明;
- 方法名: 返回页面的请求以 `.page` 结尾,返回数据的请求以`.json`结尾,例如请求路径: `/system/vehicleType/index.page` 则方法名: `indexPage`; 


1. 找到 `SysRoleController` 角色 `web` 控制类,并复制重命名为 `SysVehicleTypeController`,同时打开两个窗口,方便编辑;
![sysVehicleTypeController-split-1][sysVehicleTypeController-split-1]
![sysVehicleTypeController-split-2][sysVehicleTypeController-split-2]

2. 根据实际情况,编写 `SysVehicleTypeController` 文件;
![sysVehicleTypeController][sysVehicleTypeController]

SysVehicleTypeController

```java
package com.snowdreams1006.securityplus.browser.module.system.web;

import com.baomidou.mybatisplus.plugins.Page;
import com.fasterxml.jackson.annotation.JsonView;
import com.google.common.base.Preconditions;
import com.snowdreams1006.securityplus.browser.base.beans.PageInfoBT;
import com.snowdreams1006.securityplus.browser.base.tools.PageTools;
import com.snowdreams1006.securityplus.browser.base.web.AbstractController;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysVehicleTypeDto;
import com.snowdreams1006.securityplus.browser.module.system.entity.SysVehicleType;
import com.snowdreams1006.securityplus.browser.module.system.param.SysVehicleTypeParam;
import com.snowdreams1006.securityplus.browser.module.system.query.SysVehicleTypeQuery;
import com.snowdreams1006.securityplus.browser.module.system.service.ISysVehicleTypeService;
import com.snowdreams1006.securityplus.core.enums.ResultEnum;
import com.snowdreams1006.securityplus.core.result.Result;
import com.snowdreams1006.securityplus.core.tools.ResultTools;
import io.swagger.annotations.*;
import org.apache.commons.collections.CollectionUtils;
import org.apache.shiro.authz.annotation.RequiresPermissions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import springfox.documentation.annotations.ApiIgnore;

import javax.validation.Valid;
import java.util.List;
import java.util.Objects;

/**
 * 车辆类型管理 controller
 *
 * @author snowdreams1006
 * @date 2018-05-17
 */
@Controller
@RequestMapping("/system/vehicleType")
@Api(value = "vehicleType", tags = {"/system/vehicleType"}, description = "车辆类型管理 controller")
public class SysVehicleTypeController extends AbstractController {

    /**
     * 车辆类型管理页面路径前缀
     */
    private static final String PREFIX = "module/system/vehicleType/";

    private final ISysVehicleTypeService sysVehicleTypeService;

    @Autowired
    public SysVehicleTypeController(ISysVehicleTypeService sysVehicleTypeService) {
        this.sysVehicleTypeService = sysVehicleTypeService;
    }

    /**
     * GET 请求跳转到车辆类型管理菜单页面(有权限控制)
     */
    @RequiresPermissions("/system/vehicleType/index.page")
    @GetMapping("/index.page")
    @ApiOperation(value = "GET 请求跳转到车辆类型管理菜单页面(有权限控制)", httpMethod = "GET")
    public String indexPage() {
        return PREFIX + "index.html";
    }

    /**
     * GET 请求跳转到新增车辆类型页面(有权限控制)
     */
    @RequiresPermissions("/system/vehicleType/add.page")
    @GetMapping("/add.page")
    @ApiOperation(value = "GET 请求跳转到新增车辆类型页面(有权限控制)", httpMethod = "GET")
    public String addPage() {
        return PREFIX + "add.html";
    }

    /**
     * GET 请求跳转到更新车辆类型页面(有权限控制)
     */
    @RequiresPermissions("/system/vehicleType/update.page/{id}")
    @GetMapping("/update.page/{id}")
    @ApiOperation(value = "GET 请求跳转到更新车辆类型页面(有权限控制)", httpMethod = "GET")
    public String updatePage(@PathVariable @ApiParam(value = "车辆类型id", required = true) Long id, @ApiIgnore Model model) {
        SysVehicleTypeDto vehicleType = sysVehicleTypeService.getOne(id);
        Preconditions.checkNotNull(vehicleType, String.format("待更新的车辆类型id[%s]不存在", id));

        model.addAttribute("vehicleType", vehicleType);

        return PREFIX + "update.html";
    }

    /**
     * GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)
     */
    @RequiresPermissions("/system/vehicleType/syncTree.json")
    @GetMapping("/syncTree.json")
    @ResponseBody
    @JsonView(SysVehicleType.DetailView.class)
    @ApiOperation(value = "GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)", httpMethod = "GET")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "上级车辆类型id", defaultValue = "0", dataType = "long", paramType = "query", required = true),
            @ApiImplicitParam(name = "name", value = "名称", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "type", value = "类型", dataType = "int", paramType = "query", allowMultiple = true),
            @ApiImplicitParam(name = "startDate", value = "创建日期开始日期", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "endDate", value = "创建日期结束日期", dataType = "string", paramType = "query")
    })
    public Result syncTreeJson(@ApiIgnore SysVehicleTypeQuery sysVehicleTypeQuery) {
        SysVehicleTypeDto parent = sysVehicleTypeService.getSyncTree(sysVehicleTypeQuery);

        if (Objects.isNull(parent)) {
            return ResultTools.inexistence("同步查询的父车辆类型不存在");
        }
        if (CollectionUtils.isEmpty(parent.getChildren())) {
            return ResultTools.inexistence("同步查询的子车辆类型不存在");
        }

        return ResultTools.success(parent);
    }

    /**
     * GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)
     */
    @RequiresPermissions("/system/vehicleType/asyncTree.json")
    @GetMapping("/asyncTree.json")
    @ResponseBody
    @JsonView(SysVehicleType.SimpleView.class)
    @ApiOperation(value = "GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)", httpMethod = "GET")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "上级车辆类型id", defaultValue = "0", dataType = "long", paramType = "query", required = true),
            @ApiImplicitParam(name = "name", value = "名称", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "type", value = "类型", dataType = "int", paramType = "query", allowMultiple = true),
            @ApiImplicitParam(name = "startDate", value = "创建日期开始日期", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "endDate", value = "创建日期结束日期", dataType = "string", paramType = "query")
    })
    public Result asyncTreeJson(@ApiIgnore SysVehicleTypeQuery sysVehicleTypeQuery) {
        List<SysVehicleTypeDto> children = sysVehicleTypeService.listAsyncTree(sysVehicleTypeQuery);

        if (Objects.isNull(children)) {
            return ResultTools.inexistence("异步查询的父车辆类型不存在");
        }
        if (CollectionUtils.isEmpty(children)) {
            return ResultTools.inexistence("异步查询的子车辆类型不存在");
        }

        return ResultTools.success(children);
    }

    /**
     * GET 请求分页查询车辆类型(有权限控制,相当于全部视图)
     */
    @RequiresPermissions("/system/vehicleType/listPage.json")
    @GetMapping("/listPage.json")
    @ResponseBody
    @ApiOperation(value = "GET 请求分页查询车辆类型(有权限控制,相当于全部视图)", httpMethod = "GET")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "limit", value = "请求记录数", defaultValue = "10", dataType = "int", paramType = "query"),
            @ApiImplicitParam(name = "offset", value = "请求偏移量", defaultValue = "0", dataType = "int", paramType = "query"),
            @ApiImplicitParam(name = "sort", value = "排序字段名称", defaultValue = "id", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "order", value = "升序或降序", defaultValue = "asc", allowableValues = "asc,desc", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "id", value = "上级车辆类型id", defaultValue = "0", dataType = "long", paramType = "query", required = true),
            @ApiImplicitParam(name = "name", value = "名称", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "type", value = "类型", dataType = "int", paramType = "query", allowMultiple = true),
            @ApiImplicitParam(name = "startDate", value = "创建日期开始日期", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "endDate", value = "创建日期结束日期", dataType = "string", paramType = "query")
    })
    public Result listPageJson(@ApiIgnore SysVehicleTypeQuery sysVehicleTypeQuery) {
        Page<SysVehicleTypeDto> page = new PageTools<SysVehicleTypeDto>().buildPage();

        page = sysVehicleTypeService.listPage(page, sysVehicleTypeQuery);
        if (Objects.isNull(page)) {
            return ResultTools.inexistence("分页查询的父车辆类型不存在");
        }

        PageInfoBT<SysVehicleTypeDto> pageInfoBT = new PageTools<SysVehicleTypeDto>().packForBT(page);
        if (Objects.equals(pageInfoBT.getTotal(), 0L)) {
            return ResultTools.inexistence("分页查询的子车辆类型不存在");
        }

        return ResultTools.success(pageInfoBT);
    }

    /**
     * GET 请求条件查询车辆类型(有权限控制,并返回简单视图)
     */
    @RequiresPermissions("/system/vehicleType/listCondition.json")
    @GetMapping("/listCondition.json")
    @ResponseBody
    @JsonView(SysVehicleType.SimpleView.class)
    @ApiOperation(value = "GET 请求条件查询车辆类型(有权限控制,并返回简单视图)", httpMethod = "GET")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "上级车辆类型id", defaultValue = "0", dataType = "long", paramType = "query", required = true),
            @ApiImplicitParam(name = "name", value = "名称", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "type", value = "类型", dataType = "int", paramType = "query", allowMultiple = true),
            @ApiImplicitParam(name = "startDate", value = "创建日期开始日期", dataType = "string", paramType = "query"),
            @ApiImplicitParam(name = "endDate", value = "创建日期结束日期", dataType = "string", paramType = "query")
    })
    public Result listConditionJson(@ApiIgnore SysVehicleTypeQuery sysVehicleTypeQuery) {
        List<SysVehicleTypeDto> results = sysVehicleTypeService.listCondition(sysVehicleTypeQuery);

        if (CollectionUtils.isEmpty(results)) {
            return ResultTools.inexistence("条件查询的车辆类型不存在");
        }

        return ResultTools.success(results);
    }

    /**
     * POST 请求新增车辆类型(有权限控制,并返回详细视图)
     */
    @RequiresPermissions("/system/vehicleType/add.json")
    @PostMapping("/add.json")
    @ResponseBody
    @JsonView(SysVehicleType.DetailView.class)
    @ApiOperation(value = "POST 请求新增车辆类型(有权限控制,并返回详细视图)", httpMethod = "POST")
    public Result addJson(@Valid @RequestBody @ApiParam(value = "请提供json格式的车辆类型参数", required = true) SysVehicleTypeParam sysVehicleTypeParam, BindingResult errors) {
        Result validateResult = super.validate(errors);
        if (!Objects.equals(ResultEnum.SUCCESS.getCode(), validateResult.getCode())) {
            return validateResult;
        }

        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.addOne(sysVehicleTypeParam);

        return ResultTools.success(sysVehicleTypeDto);
    }

    /**
     * PUT 请求更新车辆类型(有权限控制,并返回详细视图)
     */
    @RequiresPermissions("/system/vehicleType/update.json")
    @PutMapping("/update.json")
    @ResponseBody
    @JsonView(SysVehicleType.DetailView.class)
    @ApiOperation(value = "PUT 请求更新车辆类型(有权限控制,并返回详细视图)", httpMethod = "PUT")
    public Result updateJson(@Valid @RequestBody @ApiParam(value = "请提供json格式的车辆类型参数", required = true) SysVehicleTypeParam sysVehicleTypeParam, BindingResult errors) {
        Result validateResult = super.validate(errors);
        if (!Objects.equals(ResultEnum.SUCCESS.getCode(), validateResult.getCode())) {
            return validateResult;
        }
        if (Objects.isNull(sysVehicleTypeParam.getId())) {
            return ResultTools.invalid(String.format("待更新的车辆类型id[%s]不存在", sysVehicleTypeParam.getName()));
        }

        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.updateOne(sysVehicleTypeParam);

        return ResultTools.success(sysVehicleTypeDto);
    }

    /**
     * DELETE 请求删除车辆类型(有权限控制)
     */
    @RequiresPermissions("/system/vehicleType/delete.json/{id}")
    @DeleteMapping("/delete.json/{id}")
    @ResponseBody
    @ApiOperation(value = "DELETE 请求删除车辆类型(有权限控制)", httpMethod = "DELETE")
    public Result deleteJson(@PathVariable("id") @ApiParam(value = "车辆类型id", required = true) Long id) {
        sysVehicleTypeService.deleteOne(id);

        return ResultTools.success();
    }

}
```


[sysVehicleTypeController-split-1]: ../../../static/image/sysVehicleTypeController-split-1.png "sysVehicleTypeController-split-1"
[sysVehicleTypeController-split-2]: ../../../static/image/sysVehicleTypeController-split-2.png "sysVehicleTypeController-split-2"
[sysVehicleTypeController]: ../../../static/image/sysVehicleTypeController.png "sysVehicleTypeController"