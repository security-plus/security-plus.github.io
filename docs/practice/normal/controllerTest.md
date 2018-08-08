# controllerTest

基本规则如下:

- `controllerTest` 继承统一基类 `BaseTest` ,可根据实际情况决定是否重写某些方法;
- 类名：`大驼峰命名+ControllerTest` 规范,默认注释同数据表注释,并加上`@FixMethodOrder`等注解说明;
- 测试方法名: `test+(大写字母)+(方法名)+SuccessWith+(条件)` 或者 `test+(大写字母)+(方法名)+FailureBecauseof+(原因)`;


1. 找到 `SysRoleControllerTest` 角色 `controllerTest` 测试类,并复制重命名为 `SysVehicleTypeControllerTest`,同时打开两个窗口,方便编辑;
![sysVehicleTypeControllerTest-split-1][sysVehicleTypeControllerTest-split-1]

2. 根据实际情况,编写 `SysVehicleTypeControllerTest` 文件,此时运行发现均无权限;
![sysVehicleTypeControllerTest-split-2][sysVehicleTypeControllerTest-split-2]

3. 新增相应的权限,两种方式: 一种是`数据库直接插入`,另一种是页面新增;
![sysVehicleTypeControllerTest-page][sysVehicleTypeControllerTest-page]
![sysVehicleTypeControllerTest-sql][sysVehicleTypeControllerTest-sql]

4. 登录系统为相应的角色分配权限,然后为用户分配该角色;
![sysVehicleTypeControllerTest-assign][sysVehicleTypeControllerTest-assign]

5. 再次执行测试用例,此时状态码由 `403` 变成 `200` ,并且测试全部通过;
![sysVehicleTypeControllerTest][sysVehicleTypeControllerTest]

插入 `车辆类型` 的权限 `sql`

```sql

SELECT * FROM zui56.sys_access;

INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `code`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (8, 0.8, 20, '车辆类型管理','vehicleType', '/system/vehicleType/index.page', '2', '自动生成: 系统模块:车辆类型管理',NOW());

set @newparentModuleId = @@identity;

INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '1', '跳转到新增车辆类型页面', '/system/vehicleType/add.page', '3', '自动生成: 从 车辆类型管理菜单页面跳转到新增车辆类型页面',NOW());
INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '2', '跳转到更新车辆类型页面', '/system/vehicleType/update.page/{id}', '3', '自动生成: 从车辆类型管理菜单页面跳转到更新车辆类型页面',NOW());

INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '3', '同步查询车辆类型树', '/system/vehicleType/syncTree.json', '4', '自动生成: 同步查询车辆类型树请求',NOW());
INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '4', '异步查询车辆类型树', '/system/vehicleType/asyncTree.json', '4', '自动生成: 异步查询车辆类型树请求',NOW());
INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '5', '分页查询车辆类型列表', '/system/vehicleType/listPage.json', '4', '自动生成: 分页查询车辆类型列表请求',NOW());
INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '6', '条件查询车辆类型列表', '/system/vehicleType/listCondition.json', '4', '自动生成: 条件查询车辆类型列表请求',NOW());

INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '7', '新增车辆类型', '/system/vehicleType/add.json', '4', '自动生成: 新增车辆类型请求',NOW());
INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '8', '更新车辆类型', '/system/vehicleType/update.json', '4', '自动生成: 更新车辆类型请求',NOW());
INSERT INTO `sys_access` (`parent_id`, `level`, `num`, `name`, `url`, `type`, `remark`,`gmt_create`)
    VALUES (@newparentModuleId, concat(0.8,'.',@newparentModuleId), '9', '删除车辆类型', '/system/vehicleType/delete.json/{id}', '4', '自动生成: 删除车辆类型请求',NOW());

SELECT * FROM sys_access ORDER BY id ASC;
```

SysVehicleTypeControllerTest

```java
package com.snowdreams1006.securityplus.browser.module.system.web;

import com.fasterxml.jackson.core.type.TypeReference;
import com.google.common.base.Preconditions;
import com.google.common.collect.Lists;
import com.snowdreams1006.securityplus.browser.base.BaseTest;
import com.snowdreams1006.securityplus.browser.base.enums.BaseStateEnum;
import com.snowdreams1006.securityplus.browser.base.enums.RoleSourceEnum;
import com.snowdreams1006.securityplus.browser.base.enums.VehicleTypeEnum;
import com.snowdreams1006.securityplus.browser.base.tools.LevelTools;
import com.snowdreams1006.securityplus.browser.base.tools.PageTools;
import com.snowdreams1006.securityplus.browser.base.tools.ShiroTools;
import com.snowdreams1006.securityplus.browser.base.tools.StringTools;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysUserDto;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysVehicleTypeDto;
import com.snowdreams1006.securityplus.browser.module.system.param.SysVehicleTypeParam;
import com.snowdreams1006.securityplus.core.enums.ResultEnum;
import com.snowdreams1006.securityplus.core.result.Result;
import com.snowdreams1006.securityplus.core.tools.JacksonTools;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.time.DateFormatUtils;
import org.apache.commons.lang3.time.DateUtils;
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runners.MethodSorters;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import java.util.Date;
import java.util.Objects;

import static org.hamcrest.Matchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

/**
 * 车辆类型管理 controllerTest
 *
 * @author snowdreams1006
 * @date 2018-05-93
 */
@Slf4j
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class SysVehicleTypeControllerTest extends BaseTest {

    /**
     * 车辆类型管理页面路径前缀
     */
    private static final String PREFIX = "module/system/vehicleType/";

    @Test
    public void testAIndexPageSuccess() throws Exception {
        //GET 请求跳转到车辆类型管理菜单页面(有权限控制)
        String result = mockMvc.perform(
                get("/system/vehicleType/index.page")
                        .contentType(MediaType.TEXT_HTML_VALUE)
        )
                .andExpect(status().isOk())
                .andExpect(view().name(PREFIX + "index.html"))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求跳转到车辆类型管理菜单页面(有权限控制)成功: {}", result);
    }

    @Test
    public void testBAddPageSuccess() throws Exception {
        //GET 请求跳转到新增车辆类型页面(有权限控制)
        String result = mockMvc.perform(
                get("/system/vehicleType/add.page")
                        .contentType(MediaType.TEXT_HTML_VALUE)
        )
                .andExpect(status().isOk())
                .andExpect(view().name(PREFIX + "add.html"))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求跳转到新增车辆类型页面(有权限控制)成功: {}", result);
    }

    @Test
    public void testCUpdatePageSuccess() throws Exception {
        //GET 请求跳转到更新车辆类型页面(有权限控制)
        String result = mockMvc.perform(
                get("/system/vehicleType/update.page/1")
                        .contentType(MediaType.TEXT_HTML_VALUE)
        )
                .andExpect(status().isOk())
                .andExpect(model().attributeExists("vehicleType"))
                .andExpect(model().attribute("vehicleType", hasProperty("id", equalTo(1L))))
                .andExpect(view().name(PREFIX + "update.html"))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求跳转到更新车辆类型页面(有权限控制)成功: {}", result);
    }

    @Test
    public void testCUpdatePageFailureBecauseofInvalidArgs() throws Exception {
        //GET 请求跳转到更新车辆类型页面(有权限控制)
        String result = mockMvc.perform(
                get("/system/vehicleType/update.page/" + Long.MAX_VALUE)
                        .contentType(MediaType.TEXT_HTML_VALUE)
        )
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.code", equalTo(HttpStatus.NOT_FOUND.value())))
                .andExpect(jsonPath("$.message", equalTo(String.format("待更新的车辆类型id[%s]不存在", Long.MAX_VALUE))))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求跳转到更新车辆类型页面(有权限控制)失败: {}", result);
    }

    @Test
    public void testCUpdatePageFailureBecauseofIllegalArgs() throws Exception {
        //GET 请求跳转到更新车辆类型页面(有权限控制)
        String result = mockMvc.perform(
                get("/system/vehicleType/update.page/" + StringTools.getRandomString(6))
                        .contentType(MediaType.TEXT_HTML_VALUE)
        )
                .andExpect(status().isInternalServerError())
                .andExpect(jsonPath("$.code", equalTo(HttpStatus.INTERNAL_SERVER_ERROR.value())))
                .andExpect(jsonPath("$.message", notNullValue()))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求跳转到更新车辆类型页面(有权限控制)失败: {}", result);
    }

    @Test
    public void testDSyncTreeJsonSuccessWithId() throws Exception {
        //GET 请求同步查询车辆类型树 (查询条件: id=0L)
        String result = mockMvc.perform(
                get("/system/vehicleType/syncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", LevelTools.ROOT_LEVEL)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.parentId", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.level", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.name", equalTo(LevelTools.ROOT_NAME)))
                .andExpect(jsonPath("$.data.isRoot", equalTo(true)))
                .andExpect(jsonPath("$.data.isParent", equalTo(true)))
                .andExpect(jsonPath("$.data.children", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].level", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.children[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].isRoot", everyItem(equalTo(false))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfChildren(result);

        log.info("GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testDSyncTreeJsonSuccessWithIdAndDate() throws Exception {
        //日期格式化
        String expectedEndDate = DateFormatUtils.format(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1),"yyyy-MM-dd");

        //GET 请求同步查询车辆类型树 (查询条件: id=0L,gmtCreate=[2018-06-04,2018-07-04])
        String result = mockMvc.perform(
                get("/system/vehicleType/syncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", LevelTools.ROOT_LEVEL)
                        .param("startDate", "2018-06-04")
                        .param("endDate", "2018-07-04")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.parentId", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.level", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.name", equalTo(LevelTools.ROOT_NAME)))
                .andExpect(jsonPath("$.data.isRoot", equalTo(true)))
                .andExpect(jsonPath("$.data.isParent", equalTo(true)))
                .andExpect(jsonPath("$.data.children", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].level", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.children[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].level", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data.children[*].gmtCreate", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].gmtCreate", everyItem(allOf(greaterThanOrEqualTo("2018-06-04"), lessThanOrEqualTo(expectedEndDate)))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfChildren(result);

        log.info("GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testDSyncTreeJsonSuccessWithIdAndType() throws Exception {
        //GET 请求同步查询车辆类型树 (查询条件: id=0L,type=[1])
        String result = mockMvc.perform(
                get("/system/vehicleType/syncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", LevelTools.ROOT_LEVEL)
                        .param("type", VehicleTypeEnum.SEMITRAILER.getValue().toString())
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.parentId", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.level", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.name", equalTo(LevelTools.ROOT_NAME)))
                .andExpect(jsonPath("$.data.isRoot", equalTo(true)))
                .andExpect(jsonPath("$.data.isParent", equalTo(true)))
                .andExpect(jsonPath("$.data.children", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].level", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.children[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].level", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data.children[*].type", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].type", everyItem(isIn(Lists.newArrayList(VehicleTypeEnum.SEMITRAILER.getValue())))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfChildren(result);

        log.info("GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testDSyncTreeJsonSuccessWithNullCondition() throws Exception {
        //GET 请求同步查询车辆类型树 (查询条件: null)
        String result = mockMvc.perform(
                get("/system/vehicleType/syncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.parentId", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.level", equalTo(LevelTools.ROOT_LEVEL)))
                .andExpect(jsonPath("$.data.name", equalTo(LevelTools.ROOT_NAME)))
                .andExpect(jsonPath("$.data.isRoot", equalTo(true)))
                .andExpect(jsonPath("$.data.isParent", equalTo(true)))
                .andExpect(jsonPath("$.data.children", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].level", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.children[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].level", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.children[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.children[*].isRoot", everyItem(equalTo(false))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfChildren(result);

        log.info("GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testDSyncTreeJsonFailureBecauseofNullParent() throws Exception {
        //GET 请求同步查询车辆类型树 (查询条件: id=Long.MAX_VALUE)
        String result = mockMvc.perform(
                get("/system/vehicleType/syncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", String.valueOf(Long.MAX_VALUE))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INEXISTENCE.getCode())))
                .andExpect(jsonPath("$.message", equalTo("同步查询的父车辆类型不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)失败: {}", result);
    }

    @Test
    public void testDSyncTreeJsonFailureBecauseofNullChildren() throws Exception {
        //GET 请求同步查询车辆类型树 (查询条件: id=93L)
        String result = mockMvc.perform(
                get("/system/vehicleType/syncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", "93")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INEXISTENCE.getCode())))
                .andExpect(jsonPath("$.message", equalTo("同步查询的子车辆类型不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求同步查询车辆类型树(有权限控制,并返回详细视图)失败: {}", result);
    }

    @Test
    public void testEAsyncTreeJsonSuccessWithId() throws Exception {
        //GET 请求异步查询车辆类型树 (查询条件: id=0L)
        String result = mockMvc.perform(
                get("/system/vehicleType/asyncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", LevelTools.ROOT_LEVEL)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data[*].level", empty()))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testEAsyncTreeJsonSuccessWithIdAndDate() throws Exception {
        //日期格式化
        String expectedEndDate = DateFormatUtils.format(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1),"yyyy-MM-dd");

        //GET 请求异步查询车辆类型树 (查询条件: id=0L,gmtCreate=[2018-06-04,2018-07-04])
        String result = mockMvc.perform(
                get("/system/vehicleType/asyncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", LevelTools.ROOT_LEVEL)
                        .param("startDate", "2018-06-04")
                        .param("endDate", "2018-07-04")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data[*].level", empty()))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andExpect(jsonPath("$.data[*].gmtCreate", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].gmtCreate", everyItem(allOf(greaterThanOrEqualTo("2018-06-04"), lessThanOrEqualTo(expectedEndDate)))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testEAsyncTreeJsonSuccessWithIdAndType() throws Exception {
        //GET 请求异步查询车辆类型树 (查询条件: id=0L,type=[1])
        String result = mockMvc.perform(
                get("/system/vehicleType/asyncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", LevelTools.ROOT_LEVEL)
                        .param("type", VehicleTypeEnum.SEMITRAILER.getValue().toString())
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data[*].level", empty()))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andExpect(jsonPath("$.data[*].type", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].type", everyItem(isIn(Lists.newArrayList(VehicleTypeEnum.SEMITRAILER.getValue())))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testEAsyncTreeJsonSuccessWithNullCondition() throws Exception {
        //GET 请求异步查询车辆类型树 (查询条件: null)
        String result = mockMvc.perform(
                get("/system/vehicleType/asyncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data[*].level", empty()))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testEAsyncTreeJsonFailureBecauseofNullParent() throws Exception {
        //GET 请求异步查询车辆类型树 (查询条件: id=Long.MAX_VALUE)
        String result = mockMvc.perform(
                get("/system/vehicleType/asyncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", String.valueOf(Long.MAX_VALUE))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INEXISTENCE.getCode())))
                .andExpect(jsonPath("$.message", equalTo("异步查询的父车辆类型不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)失败: {}", result);
    }

    @Test
    public void testEAsyncTreeJsonFailureBecauseofNullChildren() throws Exception {
        //GET 请求异步查询车辆类型树 (查询条件: id=93L)
        String result = mockMvc.perform(
                get("/system/vehicleType/asyncTree.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", "93")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INEXISTENCE.getCode())))
                .andExpect(jsonPath("$.message", equalTo("异步查询的子车辆类型不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求异步查询车辆类型树(有权限控制,并返回简单视图)失败: {}", result);
    }

    @Test
    public void testFListPageJsonSuccessWithId() throws Exception {
        //GET 请求分页查询车辆类型 (查询条件: id=0L)
        String result = mockMvc.perform(
                get("/system/vehicleType/listPage.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("offset", "0")
                        .param("limit", "3")
                        .param("sort", "num")
                        .param("order", "asc")
                        .param("id", LevelTools.ROOT_LEVEL)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.total", greaterThanOrEqualTo(0)))
                .andExpect(jsonPath("$.data.rows", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows", hasSize(lessThanOrEqualTo(3))))
                .andExpect(jsonPath("$.data.rows[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].level", everyItem(containsString(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data.rows[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfRows(result);

        log.info("GET 请求分页查询车辆类型(有权限控制,相当于全部视图)成功: {}", result);
    }

    @Test
    public void testFListPageJsonSuccessWithIdAndDate() throws Exception {
        //日期格式化
        String expectedEndDate = DateFormatUtils.format(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1),"yyyy-MM-dd");

        //GET 请求分页查询车辆类型 (查询条件: id=0L,gmtCreate=[2018-06-04,2018-07-04])
        String result = mockMvc.perform(
                get("/system/vehicleType/listPage.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("offset", "0")
                        .param("limit", "3")
                        .param("sort", "num")
                        .param("order", "asc")
                        .param("id", LevelTools.ROOT_LEVEL)
                        .param("startDate", "2018-06-04")
                        .param("endDate", "2018-07-04")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.total", greaterThanOrEqualTo(0)))
                .andExpect(jsonPath("$.data.rows", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows", hasSize(lessThanOrEqualTo(3))))
                .andExpect(jsonPath("$.data.rows[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].level", everyItem(containsString(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data.rows[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andExpect(jsonPath("$.data.rows[*].gmtCreate", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].gmtCreate", everyItem(allOf(greaterThanOrEqualTo("2018-06-04"), lessThanOrEqualTo(expectedEndDate)))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfRows(result);

        log.info("GET 请求分页查询车辆类型(有权限控制,相当于全部视图)成功: {}", result);
    }

    @Test
    public void testFListPageJsonSuccessWithIdAndType() throws Exception {
        //GET 请求分页查询车辆类型 (查询条件: id=0L,type=[1])
        String result = mockMvc.perform(
                get("/system/vehicleType/listPage.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("offset", "0")
                        .param("limit", "3")
                        .param("sort", "num")
                        .param("order", "asc")
                        .param("id", LevelTools.ROOT_LEVEL)
                        .param("type", VehicleTypeEnum.SEMITRAILER.getValue().toString())
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.total", greaterThanOrEqualTo(0)))
                .andExpect(jsonPath("$.data.rows", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows", hasSize(lessThanOrEqualTo(3))))
                .andExpect(jsonPath("$.data.rows[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].level", everyItem(containsString(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data.rows[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andExpect(jsonPath("$.data.rows[*].type", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].type", everyItem(isIn(Lists.newArrayList(VehicleTypeEnum.SEMITRAILER.getValue())))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfRows(result);

        log.info("GET 请求分页查询车辆类型(有权限控制,相当于全部视图)成功: {}", result);
    }

    @Test
    public void testFListPageJsonSuccessWithNullCondition() throws Exception {
        //GET 请求分页查询车辆类型 (查询条件: null)
        String result = mockMvc.perform(
                get("/system/vehicleType/listPage.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.total", greaterThanOrEqualTo(0)))
                .andExpect(jsonPath("$.data.rows", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows", hasSize(lessThanOrEqualTo(PageTools.DEFAULT_PAGE_SIZE))))
                .andExpect(jsonPath("$.data.rows[*].parentId", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].parentId", everyItem(equalTo(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].level", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].level", everyItem(containsString(LevelTools.ROOT_LEVEL))))
                .andExpect(jsonPath("$.data.rows[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data.rows[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data.rows[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantOfRows(result);

        log.info("GET 请求分页查询车辆类型(有权限控制,相当于全部视图)成功: {}", result);
    }

    @Test
    public void testFListPageJsonFailureBecauseofNullParent() throws Exception {
        //GET 请求分页查询车辆类型 (查询条件: id=Long.MAX_VALUE)
        String result = mockMvc.perform(
                get("/system/vehicleType/listPage.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("offset", "0")
                        .param("limit", "3")
                        .param("sort", "num")
                        .param("order", "asc")
                        .param("id", String.valueOf(Long.MAX_VALUE))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INEXISTENCE.getCode())))
                .andExpect(jsonPath("$.message", equalTo("分页查询的父车辆类型不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求分页查询车辆类型(有权限控制,相当于全部视图)失败: {}", result);
    }

    @Test
    public void testFListPageJsonFailureBecauseofNullChildren() throws Exception {
        //GET 请求分页查询车辆类型 (查询条件: id=93L)
        String result = mockMvc.perform(
                get("/system/vehicleType/listPage.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("offset", "0")
                        .param("limit", "3")
                        .param("sort", "num")
                        .param("order", "asc")
                        .param("id", "93")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INEXISTENCE.getCode())))
                .andExpect(jsonPath("$.message", equalTo("分页查询的子车辆类型不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求分页查询车辆类型(有权限控制,相当于全部视图)失败: {}", result);
    }

    @Test
    public void testGListConditionJsonSuccessWithName() throws Exception {
        //GET 请求条件查询车辆类型 (查询条件: name=半挂车)
        String result = mockMvc.perform(
                get("/system/vehicleType/listCondition.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("name", "半挂车")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].name", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].name", everyItem(containsString("半挂车"))))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求条件查询车辆类型(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testGListConditionJsonSuccessWithNameAndDate() throws Exception {
        //日期格式化
        String expectedEndDate = DateFormatUtils.format(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1),"yyyy-MM-dd");

        //GET 请求条件查询车辆类型 (查询条件: name=半挂车,gmtCreate=[2018-06-04,2018-07-04])
        String result = mockMvc.perform(
                get("/system/vehicleType/listCondition.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("name", "半挂车")
                        .param("startDate", "2018-06-04")
                        .param("endDate", "2018-07-04")
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].name", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].name", everyItem(containsString("半挂车"))))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andExpect(jsonPath("$.data[*].gmtCreate", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].gmtCreate", everyItem(allOf(greaterThanOrEqualTo("2018-06-04"), lessThanOrEqualTo(expectedEndDate)))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求条件查询车辆类型(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testGListConditionJsonSuccessWithNameAndType() throws Exception {
        //GET 请求条件查询车辆类型 (查询条件: name=半挂车,type=[1])
        String result = mockMvc.perform(
                get("/system/vehicleType/listCondition.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("name", "半挂车")
                        .param("type", VehicleTypeEnum.SEMITRAILER.getValue().toString())
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].name", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].name", everyItem(containsString("半挂车"))))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andExpect(jsonPath("$.data[*].type", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].type", everyItem(isIn(Lists.newArrayList(VehicleTypeEnum.SEMITRAILER.getValue())))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求条件查询车辆类型(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testGListConditionJsonSuccessWithNullCondition() throws Exception {
        //GET 请求条件查询车辆类型 (查询条件: null)
        String result = mockMvc.perform(
                get("/system/vehicleType/listCondition.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].state", everyItem(not(BaseStateEnum.DELETED.getValue()))))
                .andExpect(jsonPath("$.data[*].isRoot", hasSize(greaterThanOrEqualTo(0))))
                .andExpect(jsonPath("$.data[*].isRoot", everyItem(equalTo(false))))
                .andExpect(jsonPath("$.data[*].children", everyItem(emptyCollectionOf(SysVehicleTypeDto.class))))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForList(result);

        log.info("GET 请求条件查询车辆类型(有权限控制,并返回简单视图)成功: {}", result);
    }

    @Test
    public void testGRoleListConditionJsonFailureBecauseofNullResult() throws Exception {
        //GET 请求条件查询车辆类型 (查询条件: id=Long.MAX_VALUE)
        String result = mockMvc.perform(
                get("/system/vehicleType/listCondition.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .param("id", String.valueOf(Long.MAX_VALUE))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INEXISTENCE.getCode())))
                .andExpect(jsonPath("$.message", equalTo("条件查询的车辆类型不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("GET 请求条件查询车辆类型(有权限控制,并返回简单视图)失败: {}", result);
    }

    @Test
    public void testHAddJsonSuccessWithFullArgs() throws Exception {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long enterpriseId = currentUser.getOwnerEnterpriseId();
        String enterpriseName = currentUser.getOwnerEnterpriseName();
        Long userId = currentUser.getOwnerUserId();
        String userName = currentUser.getOwnerUserName();

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //默认新增完整参数
        SysVehicleTypeParam sysVehicleTypeParam = SysVehicleTypeParam.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("新增成功")
                .build();

        sysVehicleTypeParam.setRemark("新增成功:完整参数");
        sysVehicleTypeParam.setState(BaseStateEnum.OK.getValue());

        sysVehicleTypeParam.setOwnerEnterpriseId(enterpriseId);
        sysVehicleTypeParam.setOwnerEnterpriseName(enterpriseName);
        sysVehicleTypeParam.setOwnerUserId(userId);
        sysVehicleTypeParam.setOwnerUserName(userName);

        //租户过滤可能会重复添加:取决于是否存在租户约束
        if (ShiroTools.isAdmin()) {
            sysVehicleTypeParam.setTenantId(tenantId);
        }
        sysVehicleTypeParam.setTenantType(tenantType);
        sysVehicleTypeParam.setTenantName(tenantName);

        //POST 请求新增车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                post("/system/vehicleType/add.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", notNullValue()))
                .andExpect(jsonPath("$.data.parentId", equalTo(sysVehicleTypeParam.getParentId().toString())))
                .andExpect(jsonPath("$.data.level", containsString(sysVehicleTypeParam.getParentId().toString())))
                .andExpect(jsonPath("$.data.num", equalTo(sysVehicleTypeParam.getNum())))
                .andExpect(jsonPath("$.data.type", equalTo(sysVehicleTypeParam.getType())))
                .andExpect(jsonPath("$.data.name", equalTo(sysVehicleTypeParam.getName())))
                .andExpect(jsonPath("$.data.remark", equalTo(sysVehicleTypeParam.getRemark())))
                .andExpect(jsonPath("$.data.state", equalTo(sysVehicleTypeParam.getState())))
                .andExpect(jsonPath("$.data.ownerEnterpriseId", equalTo(sysVehicleTypeParam.getOwnerEnterpriseId().toString())))
                .andExpect(jsonPath("$.data.ownerEnterpriseName", equalTo(sysVehicleTypeParam.getOwnerEnterpriseName())))
                .andExpect(jsonPath("$.data.ownerUserId", equalTo(sysVehicleTypeParam.getOwnerUserId().toString())))
                .andExpect(jsonPath("$.data.ownerUserName", equalTo(sysVehicleTypeParam.getOwnerUserName())))
                .andExpect(jsonPath("$.data.createEnterpriseId", equalTo(currentUser.getCreateEnterpriseId().toString())))
                .andExpect(jsonPath("$.data.createEnterpriseName", equalTo(currentUser.getCreateEnterpriseName())))
                .andExpect(jsonPath("$.data.modifiedEnterpriseId", equalTo(currentUser.getModifiedEnterpriseId().toString())))
                .andExpect(jsonPath("$.data.modifiedEnterpriseName", equalTo(currentUser.getModifiedEnterpriseName())))
                .andExpect(jsonPath("$.data.createUserId", equalTo(currentUser.getCreateUserId().toString())))
                .andExpect(jsonPath("$.data.createUserName", equalTo(currentUser.getCreateUserName())))
                .andExpect(jsonPath("$.data.modifiedUserId", equalTo(currentUser.getModifiedUserId().toString())))
                .andExpect(jsonPath("$.data.modifiedUserName", equalTo(currentUser.getModifiedUserName())))
                .andExpect(jsonPath("$.data.tenantId", equalTo(sysVehicleTypeParam.getTenantId().toString())))
                .andExpect(jsonPath("$.data.tenantType", equalTo(sysVehicleTypeParam.getTenantType())))
                .andExpect(jsonPath("$.data.tenantName", equalTo(sysVehicleTypeParam.getTenantName())))
                .andExpect(jsonPath("$.data.gmtCreate", notNullValue()))
                .andExpect(jsonPath("$.data.gmtModified", notNullValue()))
                .andExpect(jsonPath("$.data.isRoot", equalTo(false)))
                .andExpect(jsonPath("$.data.isParent", equalTo(false)))
                .andExpect(jsonPath("$.data.children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("POST 请求新增车辆类型(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testHAddJsonSuccessWithRequiredArgs() throws Exception {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long enterpriseId = currentUser.getOwnerEnterpriseId();
        String enterpriseName = currentUser.getOwnerEnterpriseName();
        Long userId = currentUser.getOwnerUserId();
        String userName = currentUser.getOwnerUserName();

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        Integer source = ShiroTools.isAdmin() ? RoleSourceEnum.SYSTEM_ROLE.getValue() : RoleSourceEnum.NORMAL_ROLE.getValue();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //默认新增必须参数
        SysVehicleTypeParam sysVehicleTypeParam = SysVehicleTypeParam.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("新增成功")
                .build();

        sysVehicleTypeParam.setRemark("新增成功:必须参数");
        sysVehicleTypeParam.setState(BaseStateEnum.OK.getValue());

        //POST 请求新增车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                post("/system/vehicleType/add.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", notNullValue()))
                .andExpect(jsonPath("$.data.parentId", equalTo(sysVehicleTypeParam.getParentId().toString())))
                .andExpect(jsonPath("$.data.level", containsString(sysVehicleTypeParam.getParentId().toString())))
                .andExpect(jsonPath("$.data.num", equalTo(sysVehicleTypeParam.getNum())))
                .andExpect(jsonPath("$.data.type", equalTo(sysVehicleTypeParam.getType())))
                .andExpect(jsonPath("$.data.name", equalTo(sysVehicleTypeParam.getName())))
                .andExpect(jsonPath("$.data.remark", equalTo(sysVehicleTypeParam.getRemark())))
                .andExpect(jsonPath("$.data.state", equalTo(sysVehicleTypeParam.getState())))
                .andExpect(jsonPath("$.data.ownerEnterpriseId", equalTo(enterpriseId.toString())))
                .andExpect(jsonPath("$.data.ownerEnterpriseName", equalTo(enterpriseName)))
                .andExpect(jsonPath("$.data.ownerUserId", equalTo(userId.toString())))
                .andExpect(jsonPath("$.data.ownerUserName", equalTo(userName)))
                .andExpect(jsonPath("$.data.createEnterpriseId", equalTo(currentUser.getCreateEnterpriseId().toString())))
                .andExpect(jsonPath("$.data.createEnterpriseName", equalTo(currentUser.getCreateEnterpriseName())))
                .andExpect(jsonPath("$.data.modifiedEnterpriseId", equalTo(currentUser.getModifiedEnterpriseId().toString())))
                .andExpect(jsonPath("$.data.modifiedEnterpriseName", equalTo(currentUser.getModifiedEnterpriseName())))
                .andExpect(jsonPath("$.data.createUserId", equalTo(currentUser.getCreateUserId().toString())))
                .andExpect(jsonPath("$.data.createUserName", equalTo(currentUser.getCreateUserName())))
                .andExpect(jsonPath("$.data.modifiedUserId", equalTo(currentUser.getModifiedUserId().toString())))
                .andExpect(jsonPath("$.data.modifiedUserName", equalTo(currentUser.getModifiedUserName())))
                .andExpect(jsonPath("$.data.tenantId", equalTo(tenantId.toString())))
                .andExpect(jsonPath("$.data.tenantType", equalTo(tenantType)))
                .andExpect(jsonPath("$.data.tenantName", equalTo(tenantName)))
                .andExpect(jsonPath("$.data.gmtCreate", notNullValue()))
                .andExpect(jsonPath("$.data.gmtModified", notNullValue()))
                .andExpect(jsonPath("$.data.isRoot", equalTo(false)))
                .andExpect(jsonPath("$.data.isParent", equalTo(false)))
                .andExpect(jsonPath("$.data.children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("POST 请求新增车辆类型(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testHAddJsonFailureBecauseofInvalidArgs() throws Exception {
        //默认新增无效参数
        SysVehicleTypeParam sysVehicleTypeParam = SysVehicleTypeParam.builder()
//                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("新增失败")
                .build();

        sysVehicleTypeParam.setRemark("新增失败:无效参数");
        sysVehicleTypeParam.setState(BaseStateEnum.OK.getValue());

        //POST 请求新增车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                post("/system/vehicleType/add.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INVALID.getCode())))
                .andExpect(jsonPath("$.message", equalTo("parentId=上级车辆类型id不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("POST 请求新增车辆类型(有权限控制,并返回详细视图)失败: {}", result);
    }

    @Test
    public void testHAddJsonFailureBecauseofIllegalArgs() throws Exception {
        //默认新增非法参数
        SysVehicleTypeParam sysVehicleTypeParam = SysVehicleTypeParam.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("半挂车")
                .build();

        sysVehicleTypeParam.setRemark("新增失败:非法参数");
        sysVehicleTypeParam.setState(BaseStateEnum.OK.getValue());

        //POST 请求新增车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                post("/system/vehicleType/add.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.code", equalTo(HttpStatus.BAD_REQUEST.value())))
                .andExpect(jsonPath("$.message", equalTo(String.format("同一层级下的车辆类型[%s]已存在", "半挂车"))))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("POST 请求新增车辆类型(有权限控制,并返回详细视图)失败: {}", result);
    }

    @Test
    public void testIUpdateJsonSuccessWithAlone() throws Exception {
        //日期时间格式化
        String expectedModified = DateFormatUtils.format(DateUtils.addSeconds(new Date(), -10),"yyyy-MM-dd HH:mm:ss");

        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName("更新成功");

        sysVehicleTypeParam.setRemark("更新成功:单独更新");

        //PUT 请求更新车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                MockMvcRequestBuilders.put("/system/vehicleType/update.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", equalTo(sysVehicleTypeParam.getId().toString())))
                .andExpect(jsonPath("$.data.num", equalTo(sysVehicleTypeParam.getNum())))
                .andExpect(jsonPath("$.data.name", equalTo(sysVehicleTypeParam.getName())))
                .andExpect(jsonPath("$.data.remark", equalTo(sysVehicleTypeParam.getRemark())))
                .andExpect(jsonPath("$.data.gmtModified", greaterThanOrEqualTo(expectedModified)))
                .andExpect(jsonPath("$.data.isRoot", equalTo(prepare.getIsRoot())))
                .andExpect(jsonPath("$.data.isParent", equalTo(prepare.getIsParent())))
                .andExpect(jsonPath("$.data.children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForOne(result);

        log.info("PUT 请求更新车辆类型(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testIUpdateJsonSuccessWithDescendant() throws Exception {
        //日期时间格式化
        String expectedModified = DateFormatUtils.format(DateUtils.addSeconds(new Date(), -10),"yyyy-MM-dd HH:mm:ss");

        //测试数据
        SysVehicleTypeDto parentDto = prepareTestData(null);

        SysVehicleTypeParam childParam = SysVehicleTypeParam.builder()
                .parentId(parentDto.getId())
                .num(1)
                .type(parentDto.getType())
                .name(parentDto.getName() + ":子车辆类型")
                .build();

        childParam.setRemark(parentDto.getRemark() + ":子车辆类型");
        childParam.setState(parentDto.getState());

        prepareTestData(childParam);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(parentDto));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName("更新成功");

        sysVehicleTypeParam.setRemark("更新成功:后代更新");

        //PUT 请求更新车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                put("/system/vehicleType/update.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andExpect(jsonPath("$.data.id", equalTo(sysVehicleTypeParam.getId().toString())))
                .andExpect(jsonPath("$.data.num", equalTo(sysVehicleTypeParam.getNum())))
                .andExpect(jsonPath("$.data.name", equalTo(sysVehicleTypeParam.getName())))
                .andExpect(jsonPath("$.data.remark", equalTo(sysVehicleTypeParam.getRemark())))
                .andExpect(jsonPath("$.data.gmtModified", greaterThanOrEqualTo(expectedModified)))
                .andExpect(jsonPath("$.data.isRoot", equalTo(false)))
                .andExpect(jsonPath("$.data.isParent", equalTo(true)))
                .andExpect(jsonPath("$.data.children", emptyCollectionOf(SysVehicleTypeDto.class)))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        //验证租户信息
        validateTenantForOne(result);

        log.info("PUT 请求更新车辆类型(有权限控制,并返回详细视图)成功: {}", result);
    }

    @Test
    public void testIUpdateJsonFailureBecauseofInvalidArgs() throws Exception {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName("更新失败");
        sysVehicleTypeParam.setParentId(null);

        sysVehicleTypeParam.setRemark("更新失败:无效参数");

        //PUT 请求更新车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                put("/system/vehicleType/update.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(ResultEnum.INVALID.getCode()))
                .andExpect(jsonPath("$.message", equalTo("parentId=上级车辆类型id不存在")))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("PUT 请求更新车辆类型(有权限控制,并返回详细视图)失败: {}", result);
    }

    @Test
    public void testIUpdateJsonFailureBecauseofIllegalArgs() throws Exception {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeDto repeat = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName(repeat.getName());

        sysVehicleTypeParam.setRemark("更新失败:非法参数");

        //PUT 请求更新车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                put("/system/vehicleType/update.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.code", equalTo(HttpStatus.BAD_REQUEST.value())))
                .andExpect(jsonPath("$.message", equalTo(String.format("同一层级下的车辆类型[%s]已存在", sysVehicleTypeParam.getName()))))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("PUT 请求更新车辆类型(有权限控制,并返回详细视图)失败: {}", result);
    }

    @Test
    public void testIUpdateJsonFailureBecauseofNullIdArgs() throws Exception {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName("更新失败");
        sysVehicleTypeParam.setId(null);

        sysVehicleTypeParam.setRemark("更新失败:主键缺失");

        //PUT 请求更新车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                put("/system/vehicleType/update.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.INVALID.getCode())))
                .andExpect(jsonPath("$.message", equalTo(String.format("待更新的车辆类型id[%s]不存在", sysVehicleTypeParam.getName()))))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("PUT 请求更新车辆类型(有权限控制,并返回详细视图)失败: {}", result);
    }

    @Test
    public void testJDeleteJsonSuccess() throws Exception {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        //DELETE 请求删除车辆类型(有权限控制)
        String result = mockMvc.perform(
                delete("/system/vehicleType/delete.json/" + prepare.getId())
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("DELETE 请求删除车辆类型(有权限控制)成功: {}", result);
    }

    @Test
    public void testJDeleteJsonFailureBecauseofInvalidlArgs() throws Exception {
        //DELETE 请求删除车辆类型(有权限控制)
        String result = mockMvc.perform(
                delete("/system/vehicleType/delete.json/" + Long.MAX_VALUE)
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
        )
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.code", equalTo(HttpStatus.NOT_FOUND.value())))
                .andExpect(jsonPath("$.message", equalTo(String.format("待删除的车辆类型id[%s]不存在", Long.MAX_VALUE))))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("DELETE 请求删除车辆类型(有权限控制)失败: {}", result);
    }

    @Test
    public void testJDeleteJsonFailureBecauseofIllegalArgs() throws Exception {
        //测试数据
        SysVehicleTypeDto parentDto = prepareTestData(null);

        SysVehicleTypeParam childParam = SysVehicleTypeParam.builder()
                .parentId(parentDto.getId())
                .num(1)
                .type(parentDto.getType())
                .name(parentDto.getName() + ":子车辆类型")
                .build();

        childParam.setRemark(parentDto.getRemark() + ":子车辆类型");
        childParam.setState(parentDto.getState());

        prepareTestData(childParam);

        //DELETE 请求删除车辆类型(有权限控制)
        String result = mockMvc.perform(
                delete("/system/vehicleType/delete.json/" + parentDto.getId())
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
        )
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.code", equalTo(HttpStatus.BAD_REQUEST.value())))
                .andExpect(jsonPath("$.message", equalTo(String.format("待删除的车辆类型[%s]存在子车辆类型", parentDto.getName()))))
                .andExpect(jsonPath("$.data", nullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("DELETE 请求删除车辆类型(有权限控制)失败: {}", result);
    }

    /**
     * 准备测试数据
     *
     * @param sysVehicleTypeParam 新增单个车辆类型 param
     * @return 新增单个车辆类型 dto
     * @throws Exception 新增失败时抛出异常
     */
    private SysVehicleTypeDto prepareTestData(SysVehicleTypeParam sysVehicleTypeParam) throws Exception {
        //测试数据
        if (Objects.isNull(sysVehicleTypeParam)) {
            //当前登录用户信息
            SysUserDto currentUser = ShiroTools.getShiroUser();
            Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

            Long enterpriseId = currentUser.getOwnerEnterpriseId();
            String enterpriseName = currentUser.getOwnerEnterpriseName();
            Long userId = currentUser.getOwnerUserId();
            String userName = currentUser.getOwnerUserName();

            Long tenantId = currentUser.getTenantId();
            Integer tenantType = currentUser.getTenantType();
            String tenantName = currentUser.getTenantName();

            log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

            //默认测试数据
            sysVehicleTypeParam = SysVehicleTypeParam.builder()
                    .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                    .num(Integer.MAX_VALUE)
                    .type(VehicleTypeEnum.SEMITRAILER.getValue())
                    .name("测试数据:" + StringTools.getRandomString(6))
                    .build();

            sysVehicleTypeParam.setRemark("测试数据:" + StringTools.getRandomString(6));
            sysVehicleTypeParam.setState(BaseStateEnum.OK.getValue());

            sysVehicleTypeParam.setOwnerEnterpriseId(enterpriseId);
            sysVehicleTypeParam.setOwnerEnterpriseName(enterpriseName);
            sysVehicleTypeParam.setOwnerUserId(userId);
            sysVehicleTypeParam.setOwnerUserName(userName);

            //租户过滤可能会重复添加:取决于是否存在租户约束
            if (ShiroTools.isAdmin()) {
                sysVehicleTypeParam.setTenantId(tenantId);
            }
            sysVehicleTypeParam.setTenantType(tenantType);
            sysVehicleTypeParam.setTenantName(tenantName);
        }

        //POST 请求新增车辆类型(有权限控制,并返回详细视图)
        String result = mockMvc.perform(
                post("/system/vehicleType/add.json")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(JacksonTools.object2Json(sysVehicleTypeParam))
        )
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code", equalTo(ResultEnum.SUCCESS.getCode())))
                .andExpect(jsonPath("$.message", equalTo(ResultEnum.SUCCESS.getMessage())))
                .andExpect(jsonPath("$.data", notNullValue()))
                .andDo(print())
                .andReturn().getResponse().getContentAsString();

        log.info("POST 请求新增车辆类型(有权限控制,并返回详细视图)成功: {}", result);

        return JacksonTools.json2Object(result, new TypeReference<Result<SysVehicleTypeDto>>() {

        }).getData();
    }

}
```

[sysVehicleTypeControllerTest-split-1]: ../../../static/image/sysVehicleTypeControllerTest-split-1.png "sysVehicleTypeControllerTest-split-1"
[sysVehicleTypeControllerTest-split-2]: ../../../static/image/sysVehicleTypeControllerTest-split-2.png "sysVehicleTypeControllerTest-split-2"
[sysVehicleTypeControllerTest-page]: ../../../static/image/sysVehicleTypeControllerTest-page.png "sysVehicleTypeControllerTest-page"
[sysVehicleTypeControllerTest-sql]: ../../../static/image/sysVehicleTypeControllerTest-sql.png "sysVehicleTypeControllerTest-sql"
[sysVehicleTypeControllerTest-assign]: ../../../static/image/sysVehicleTypeControllerTest-assign.png "sysVehicleTypeControllerTest-assign"
[sysVehicleTypeControllerTest]: ../../../static/image/sysVehicleTypeControllerTest.png "sysVehicleTypeControllerTest"