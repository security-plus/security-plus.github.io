# serviceImplTest

基本规则如下:

- `serviceImplTest` 继承统一基类 `BaseTest` ,可根据实际情况决定是否重写某些方法;
- 类名：`大驼峰命名+ServiceImplTest` 规范,默认注释同数据表注释,并加上`@FixMethodOrder`等注解说明;
- 测试方法名: `test+(大写字母)+(方法名)+SuccessWith+(条件)` 或者 `test+(大写字母)+(方法名)+FailureBecauseof+(原因)`;


1. 找到 `SysRoleServiceImplTest` 角色 `serviceImplTest` 测试类,并复制重命名为 `SysVehicleTypeServiceImplTest`,同时打开两个窗口,方便编辑;
![sysVehicleTypeServiceImplTest-split-1][sysVehicleTypeServiceImplTest-split-1]
![sysVehicleTypeServiceImplTest-split-2][sysVehicleTypeServiceImplTest-split-2]
![sysVehicleTypeServiceImplTest-split-3][sysVehicleTypeServiceImplTest-split-3]

2. 根据实际情况,编写 `SysVehicleTypeServiceImplTest` 文件;
![sysVehicleTypeServiceImplTest][sysVehicleTypeServiceImplTest]

SysVehicleTypeServiceImplTest

```java
package com.snowdreams1006.securityplus.browser.module.system.service.impl;

import com.baomidou.mybatisplus.plugins.Page;
import com.google.common.base.Preconditions;
import com.google.common.collect.Lists;
import com.snowdreams1006.securityplus.browser.base.BaseTest;
import com.snowdreams1006.securityplus.browser.base.enums.BaseStateEnum;
import com.snowdreams1006.securityplus.browser.base.enums.RoleSourceEnum;
import com.snowdreams1006.securityplus.browser.base.enums.RoleTypeEnum;
import com.snowdreams1006.securityplus.browser.base.enums.VehicleTypeEnum;
import com.snowdreams1006.securityplus.browser.base.tools.LevelTools;
import com.snowdreams1006.securityplus.browser.base.tools.PageTools;
import com.snowdreams1006.securityplus.browser.base.tools.ShiroTools;
import com.snowdreams1006.securityplus.browser.base.tools.StringTools;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysUserDto;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysVehicleTypeDto;
import com.snowdreams1006.securityplus.browser.module.system.param.SysVehicleTypeParam;
import com.snowdreams1006.securityplus.browser.module.system.query.SysVehicleTypeQuery;
import com.snowdreams1006.securityplus.browser.module.system.service.ISysVehicleTypeService;
import com.snowdreams1006.securityplus.core.tools.JacksonTools;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.time.DateUtils;
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runners.MethodSorters;
import org.springframework.beans.factory.annotation.Autowired;

import javax.validation.ValidationException;
import java.text.ParseException;
import java.util.Date;
import java.util.List;
import java.util.Objects;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;

/**
 * 系统级别车辆类型表 ServiceImpl测试
 *
 * @author snowdreams1006
 * @date 2018-05-26
 */
@Slf4j
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class SysVehicleTypeServiceImplTest extends BaseTest {

    @Autowired
    private ISysVehicleTypeService sysVehicleTypeService;

    @Test
    public void testAGetSyncTreeSuccessWithId() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L
        SysVehicleTypeQuery descendantQuery = new SysVehicleTypeQuery();
        descendantQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));

        //同步查询
        SysVehicleTypeDto parent = sysVehicleTypeService.getSyncTree(descendantQuery);

        assertThat(parent, notNullValue());
        assertThat(parent.getId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(parent.getName(), equalTo(LevelTools.ROOT_NAME));
        assertThat(parent.getIsRoot(), equalTo(true));
        assertThat(parent.getIsParent(), equalTo(true));
        assertThat(parent.getChildren(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(parent.getChildren(), everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(parent.getChildren(), everyItem(hasProperty("level", equalTo(LevelTools.ROOT_LEVEL))));
        assertThat(parent.getChildren(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(parent.getChildren(), everyItem(hasProperty("isRoot", equalTo(false))));

        log.info("同步查询成功: {}", JacksonTools.object2Json(parent));
    }

    @Test
    public void testAGetSyncTreeSuccessWithIdAndDate() throws ParseException {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L,gmtCreate=[2018-06-04,2018-07-04]
        SysVehicleTypeQuery descendantQuery = new SysVehicleTypeQuery();
        descendantQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));
        descendantQuery.setStartDate(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd"));
        descendantQuery.setEndDate(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"));

        //同步查询
        SysVehicleTypeDto parent = sysVehicleTypeService.getSyncTree(descendantQuery);

        assertThat(parent, notNullValue());
        assertThat(parent.getId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(parent.getName(), equalTo(LevelTools.ROOT_NAME));
        assertThat(parent.getIsRoot(), equalTo(true));
        assertThat(parent.getIsParent(), equalTo(true));
        assertThat(parent.getChildren(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(parent.getChildren(), everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(parent.getChildren(), everyItem(hasProperty("level", equalTo(LevelTools.ROOT_LEVEL))));
        assertThat(parent.getChildren(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(parent.getChildren(), everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(parent.getChildren(), everyItem(hasProperty("gmtCreate", allOf(greaterThanOrEqualTo(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd")), lessThanOrEqualTo(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1))))));

        log.info("同步查询成功: {}", JacksonTools.object2Json(parent));
    }

    @Test
    public void testAGetSyncTreeSuccessWithIdAndType() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L,type=[1]
        List<Integer> type = Lists.newArrayList();
        type.add(RoleTypeEnum.ENTERPRISE.getValue());

        SysVehicleTypeQuery descendantQuery = new SysVehicleTypeQuery();
        descendantQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));
        descendantQuery.setType(type);

        //同步查询
        SysVehicleTypeDto parent = sysVehicleTypeService.getSyncTree(descendantQuery);

        assertThat(parent, notNullValue());
        assertThat(parent.getId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(parent.getName(), equalTo(LevelTools.ROOT_NAME));
        assertThat(parent.getIsRoot(), equalTo(true));
        assertThat(parent.getIsParent(), equalTo(true));
        assertThat(parent.getChildren(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(parent.getChildren(), everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(parent.getChildren(), everyItem(hasProperty("level", equalTo(LevelTools.ROOT_LEVEL))));
        assertThat(parent.getChildren(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(parent.getChildren(), everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(parent.getChildren(), everyItem(hasProperty("type", isIn(type))));

        log.info("同步查询成功: {}", JacksonTools.object2Json(parent));
    }

    @Test
    public void testAGetSyncTreeSuccessWithNullCondition() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //同步查询
        SysVehicleTypeDto parent = sysVehicleTypeService.getSyncTree(null);

        assertThat(parent, notNullValue());
        assertThat(parent.getId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(parent.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(parent.getName(), equalTo(LevelTools.ROOT_NAME));
        assertThat(parent.getIsRoot(), equalTo(true));
        assertThat(parent.getIsParent(), equalTo(true));
        assertThat(parent.getChildren(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(parent.getChildren(), everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(parent.getChildren(), everyItem(hasProperty("level", equalTo(LevelTools.ROOT_LEVEL))));
        assertThat(parent.getChildren(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(parent.getChildren(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(parent.getChildren(), everyItem(hasProperty("isRoot", equalTo(false))));

        log.info("同步查询成功: {}", JacksonTools.object2Json(parent));
    }

    @Test
    public void testAGetSyncTreeFailureBecauseofNullParent() {
        //查询条件: id=Long.MAX_VALUE
        SysVehicleTypeQuery descendantQuery = new SysVehicleTypeQuery();
        descendantQuery.setId(Long.MAX_VALUE);

        //同步查询
        SysVehicleTypeDto parent = sysVehicleTypeService.getSyncTree(descendantQuery);

        assertThat(parent, nullValue());

        log.info("同步查询失败: null");
    }

    @Test
    public void testAGetSyncTreeFailureBecauseofNullChildren() {
        //查询条件: id=93L
        SysVehicleTypeQuery descendantQuery = new SysVehicleTypeQuery();
        descendantQuery.setId(93L);

        //同步查询
        SysVehicleTypeDto parent = sysVehicleTypeService.getSyncTree(descendantQuery);

        assertThat(parent, notNullValue());
        assertThat(parent.getId(), equalTo(93L));
        assertThat(parent.getParentId(), equalTo(1L));
        assertThat(parent.getLevel(), equalTo("0.1"));
        assertThat(parent.getName(), equalTo("轻型普通半挂车"));
        assertThat(parent.getState(), not(BaseStateEnum.DELETED.getValue()));
        assertThat(parent.getIsRoot(), equalTo(false));
        assertThat(parent.getIsParent(), equalTo(false));
        assertThat(parent.getChildren(), emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("同步查询失败: {}", JacksonTools.object2Json(parent));
    }

    @Test
    public void testBListASyncTreeSuccessWithId() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L
        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));

        //异步查询
        List<SysVehicleTypeDto> children = sysVehicleTypeService.listAsyncTree(childrenQuery);

        assertThat(children, notNullValue());
        assertThat(children, hasSize(greaterThanOrEqualTo(0)));
        assertThat(children, everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(children, everyItem(hasProperty("level", containsString(childrenQuery.getId().toString()))));
        assertThat(children, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(children, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(children, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(children, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(children, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(children, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));

        log.info("异步查询成功: {}", JacksonTools.object2Json(children));
    }

    @Test
    public void testBListASyncTreeSuccessWithIdAndDate() throws ParseException {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L,gmtCreate=[2018-06-04,2018-07-04]
        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));
        childrenQuery.setStartDate(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd"));
        childrenQuery.setEndDate(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"));

        //异步查询
        List<SysVehicleTypeDto> children = sysVehicleTypeService.listAsyncTree(childrenQuery);

        assertThat(children, hasSize(greaterThanOrEqualTo(0)));
        assertThat(children, everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(children, everyItem(hasProperty("level", containsString(childrenQuery.getId().toString()))));
        assertThat(children, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(children, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(children, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(children, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(children, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(children, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));
        assertThat(children, everyItem(hasProperty("gmtCreate", allOf(greaterThanOrEqualTo(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd")), lessThanOrEqualTo(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1))))));

        log.info("异步查询成功: {}", JacksonTools.object2Json(children));
    }

    @Test
    public void testBListASyncTreeSuccessWithIdAndType() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L,type=[1]
        List<Integer> type = Lists.newArrayList();
        type.add(RoleTypeEnum.ENTERPRISE.getValue());

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));
        childrenQuery.setType(type);

        //异步查询
        List<SysVehicleTypeDto> children = sysVehicleTypeService.listAsyncTree(childrenQuery);

        assertThat(children, hasSize(greaterThanOrEqualTo(0)));
        assertThat(children, everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(children, everyItem(hasProperty("level", containsString(childrenQuery.getId().toString()))));
        assertThat(children, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(children, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(children, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(children, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(children, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(children, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));
        assertThat(children, everyItem(hasProperty("type", isIn(type))));

        log.info("异步查询成功: {}", JacksonTools.object2Json(children));
    }

    @Test
    public void testBListASyncTreeSuccessWithNullCondition() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //异步查询
        List<SysVehicleTypeDto> children = sysVehicleTypeService.listAsyncTree(null);

        assertThat(children, hasSize(greaterThanOrEqualTo(0)));
        assertThat(children, everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(children, everyItem(hasProperty("level", equalTo(LevelTools.ROOT_LEVEL))));
        assertThat(children, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(children, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(children, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(children, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(children, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(children, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));

        log.info("异步查询成功: {}", JacksonTools.object2Json(children));
    }

    @Test
    public void testBListAsyncTreeFailureBecauseofNullParent() {
        //查询条件: id=Long.MAX_VALUE
        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.MAX_VALUE);

        //异步查询
        List<SysVehicleTypeDto> children = sysVehicleTypeService.listAsyncTree(childrenQuery);

        assertThat(children, nullValue());

        log.info("异步查询失败: null");
    }

    @Test
    public void testBListASyncTreeFailureBecauseofNullChildren() {
        //查询条件: id=93L
        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(93L);

        //异步查询
        List<SysVehicleTypeDto> children = sysVehicleTypeService.listAsyncTree(childrenQuery);

        assertThat(children, notNullValue());
        assertThat(children, emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("异步查询失败: {}", JacksonTools.object2Json(children));
    }

    @Test
    public void testCListPageSuccessWithId() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L
        Page<SysVehicleTypeDto> page = new Page<>(1, 3, "num", true);

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));

        //分页查询
        page = sysVehicleTypeService.listPage(page, childrenQuery);

        assertThat(page, notNullValue());
        assertThat(page.getCurrent(), equalTo(1));
        assertThat(page.getSize(), equalTo(3));
        assertThat(page.getPages(), greaterThanOrEqualTo(0L));
        assertThat(page.getTotal(), greaterThanOrEqualTo(0L));
        assertThat(page.getRecords(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(page.getRecords(), hasSize(lessThanOrEqualTo(3)));
        assertThat(page.getRecords(), everyItem(hasProperty("parentId", equalTo(childrenQuery.getId()))));
        assertThat(page.getRecords(), everyItem(hasProperty("level", containsString(LevelTools.ROOT_LEVEL))));
        assertThat(page.getRecords(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(page.getRecords(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(page.getRecords(), everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(page.getRecords(), everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));

        log.info("分页查询成功: {}", JacksonTools.object2Json(page));
    }

    @Test
    public void testCListPageSuccessWithIdAndDate() throws ParseException {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L,gmtCreate=[2018-06-04,2018-07-04]
        Page<SysVehicleTypeDto> page = new Page<>(1, 3, "num", true);

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));
        childrenQuery.setStartDate(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd"));
        childrenQuery.setEndDate(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"));

        //分页查询
        page = sysVehicleTypeService.listPage(page, childrenQuery);

        assertThat(page, notNullValue());
        assertThat(page.getCurrent(), equalTo(1));
        assertThat(page.getSize(), equalTo(3));
        assertThat(page.getPages(), greaterThanOrEqualTo(0L));
        assertThat(page.getTotal(), greaterThanOrEqualTo(0L));
        assertThat(page.getRecords(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(page.getRecords(), hasSize(lessThanOrEqualTo(3)));
        assertThat(page.getRecords(), everyItem(hasProperty("parentId", equalTo(childrenQuery.getId()))));
        assertThat(page.getRecords(), everyItem(hasProperty("level", containsString(LevelTools.ROOT_LEVEL))));
        assertThat(page.getRecords(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(page.getRecords(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(page.getRecords(), everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(page.getRecords(), everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));
        assertThat(page.getRecords(), everyItem(hasProperty("gmtCreate", allOf(greaterThanOrEqualTo(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd")), lessThanOrEqualTo(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1))))));

        log.info("分页查询成功: {}", JacksonTools.object2Json(page));
    }

    @Test
    public void testCListPageSuccessWithIdAndType() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: id=0L,type=[1]
        Page<SysVehicleTypeDto> page = new Page<>(1, 3, "num", true);

        List<Integer> type = Lists.newArrayList();
        type.add(RoleTypeEnum.ENTERPRISE.getValue());

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.valueOf(LevelTools.ROOT_LEVEL));
        childrenQuery.setType(type);

        //分页查询
        page = sysVehicleTypeService.listPage(page, childrenQuery);

        assertThat(page, notNullValue());
        assertThat(page.getCurrent(), equalTo(1));
        assertThat(page.getSize(), equalTo(3));
        assertThat(page.getPages(), greaterThanOrEqualTo(0L));
        assertThat(page.getTotal(), greaterThanOrEqualTo(0L));
        assertThat(page.getRecords(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(page.getRecords(), hasSize(lessThanOrEqualTo(3)));
        assertThat(page.getRecords(), everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(page.getRecords(), everyItem(hasProperty("level", containsString(LevelTools.ROOT_LEVEL))));
        assertThat(page.getRecords(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(page.getRecords(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(page.getRecords(), everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(page.getRecords(), everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));
        assertThat(page.getRecords(), everyItem(hasProperty("type", isIn(type))));

        log.info("分页查询成功: {}", JacksonTools.object2Json(page));
    }

    @Test
    public void testCListPageSuccessWithNullCondition() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //分页查询
        Page<SysVehicleTypeDto> page = sysVehicleTypeService.listPage(null, null);

        assertThat(page, notNullValue());
        assertThat(page.getCurrent(), equalTo(PageTools.DEFAULT_PAGE_CURRENT));
        assertThat(page.getSize(), equalTo(PageTools.DEFAULT_PAGE_SIZE));
        assertThat(page.getPages(), greaterThanOrEqualTo(0L));
        assertThat(page.getTotal(), greaterThanOrEqualTo(0L));
        assertThat(page.getRecords(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(page.getRecords(), hasSize(lessThanOrEqualTo(PageTools.DEFAULT_PAGE_SIZE)));
        assertThat(page.getRecords(), everyItem(hasProperty("parentId", equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)))));
        assertThat(page.getRecords(), everyItem(hasProperty("level", equalTo(LevelTools.ROOT_LEVEL))));
        assertThat(page.getRecords(), everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(page.getRecords(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(page.getRecords(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(page.getRecords(), everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(page.getRecords(), everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));

        log.info("分页查询成功: {}", JacksonTools.object2Json(page));
    }

    @Test
    public void testCListPageFailureBecauseofNullParent() {
        //查询条件: id=Long.MAX_VALUE
        Page<SysVehicleTypeDto> page = new Page<>(1, 3, "num", true);

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(Long.MAX_VALUE);

        //分页查询
        page = sysVehicleTypeService.listPage(page, childrenQuery);

        assertThat(page, nullValue());

        log.info("分页查询失败: null");
    }

    @Test
    public void testCListPageFailureBecauseofNullChildren() {
        //查询条件: id=93L
        Page<SysVehicleTypeDto> page = new Page<>(1, 3, "num", true);

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setId(93L);

        //分页查询
        page = sysVehicleTypeService.listPage(page, childrenQuery);

        assertThat(page, notNullValue());
        assertThat(page.getCurrent(), equalTo(1));
        assertThat(page.getSize(), equalTo(3));
        assertThat(page.getPages(), equalTo(0L));
        assertThat(page.getTotal(), equalTo(0L));
        assertThat(page.getRecords(), emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("分页查询失败: {}", JacksonTools.object2Json(page));
    }

    @Test
    public void testDListConditionSuccessWithName() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: name=半挂车
        SysVehicleTypeQuery sysVehicleTypeQuery = new SysVehicleTypeQuery();
        sysVehicleTypeQuery.setName("半挂车");

        //条件查询
        List<SysVehicleTypeDto> results = sysVehicleTypeService.listCondition(sysVehicleTypeQuery);

        assertThat(results, notNullValue());
        assertThat(results, hasSize(greaterThanOrEqualTo(0)));
        assertThat(results, everyItem(hasProperty("name", containsString("半挂车"))));
        assertThat(results, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(results, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(results, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(results, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(results, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(results, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));

        log.info("条件查询成功: {}", JacksonTools.object2Json(results));
    }

    @Test
    public void testDListConditionSuccessWithNameAndDate() throws ParseException {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: name=半挂车,gmtCreate=[2018-06-04,2018-07-04]
        SysVehicleTypeQuery sysVehicleTypeQuery = new SysVehicleTypeQuery();
        sysVehicleTypeQuery.setName("半挂车");
        sysVehicleTypeQuery.setStartDate(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd"));
        sysVehicleTypeQuery.setEndDate(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"));

        //条件查询
        List<SysVehicleTypeDto> results = sysVehicleTypeService.listCondition(sysVehicleTypeQuery);

        assertThat(results, notNullValue());
        assertThat(results, hasSize(greaterThanOrEqualTo(0)));
        assertThat(results, everyItem(hasProperty("name", containsString("半挂车"))));
        assertThat(results, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(results, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(results, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(results, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(results, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(results, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));
        assertThat(results, everyItem(hasProperty("gmtCreate", allOf(greaterThanOrEqualTo(DateUtils.parseDate("2018-06-04", "yyyy-MM-dd")), lessThanOrEqualTo(DateUtils.addDays(DateUtils.parseDate("2018-07-04", "yyyy-MM-dd"), 1))))));

        log.info("条件查询成功: {}", JacksonTools.object2Json(results));
    }

    @Test
    public void testDListConditionSuccessWithNameAndType() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件: name=半挂车,type=[1]
        List<Integer> type = Lists.newArrayList();
        type.add(RoleTypeEnum.ENTERPRISE.getValue());

        SysVehicleTypeQuery sysVehicleTypeQuery = new SysVehicleTypeQuery();
        sysVehicleTypeQuery.setName("半挂车");
        sysVehicleTypeQuery.setType(type);

        //条件查询
        List<SysVehicleTypeDto> results = sysVehicleTypeService.listCondition(sysVehicleTypeQuery);

        assertThat(results, notNullValue());
        assertThat(results, hasSize(greaterThanOrEqualTo(0)));
        assertThat(results, everyItem(hasProperty("name", containsString("半挂车"))));
        assertThat(results, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(results, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(results, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(results, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(results, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(results, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));
        assertThat(results, everyItem(hasProperty("type", isIn(type))));

        log.info("条件查询成功: {}", JacksonTools.object2Json(results));
    }

    @Test
    public void testDListConditionSuccessWithNullCondition() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //条件查询
        List<SysVehicleTypeDto> results = sysVehicleTypeService.listCondition(null);

        assertThat(results, notNullValue());
        assertThat(results, hasSize(greaterThanOrEqualTo(0)));
        assertThat(results, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(results, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(results, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(results, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        assertThat(results, everyItem(hasProperty("isRoot", equalTo(false))));
        assertThat(results, everyItem(hasProperty("children", emptyCollectionOf(SysVehicleTypeDto.class))));

        log.info("条件查询成功: {}", JacksonTools.object2Json(results));
    }

    @Test
    public void testDListConditionFailureBecauseofNullResult() {
        //查询条件: id=Long.MAX_VALUE
        SysVehicleTypeQuery sysVehicleTypeQuery = new SysVehicleTypeQuery();
        sysVehicleTypeQuery.setId(Long.MAX_VALUE);

        //条件查询
        List<SysVehicleTypeDto> results = sysVehicleTypeService.listCondition(sysVehicleTypeQuery);

        assertThat(results, notNullValue());
        assertThat(results, emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("条件查询失败: {}", JacksonTools.object2Json(results));
    }

    @Test
    public void testEGetOneSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //精确查询: id=1L
        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.getOne(1L);

        assertThat(sysVehicleTypeDto, notNullValue());
        assertThat(sysVehicleTypeDto.getId(), equalTo(1L));
        assertThat(sysVehicleTypeDto.getState(), not(BaseStateEnum.DELETED.getValue()));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeDto.getTenantId(), equalTo(tenantId));
            assertThat(sysVehicleTypeDto.getTenantType(), equalTo(tenantType));
            assertThat(sysVehicleTypeDto.getTenantName(), equalTo(tenantName));
        }
        assertThat(sysVehicleTypeDto.getIsRoot(), equalTo(false));

        log.info("精确查询成功: {}", JacksonTools.object2Json(sysVehicleTypeDto));
    }

    @Test
    public void testEGetOneFailureBecauseofNullCondition() {
        try {
            //精确查询: id=null
            SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.getOne(null);

            assertThat(sysVehicleTypeDto, nullValue());
        } catch (NullPointerException e) {
            assertThat(e.getMessage(), equalTo("待查询的车辆类型id[null]不存在"));

            log.info("精确查询失败: {}", e);
        }
    }

    @Test
    public void testEGetOneFailureBecauseofNullResult() {
        //精确查询: id=Long.MAX_VALUE
        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.getOne(Long.MAX_VALUE);

        assertThat(sysVehicleTypeDto, nullValue());

        log.info("精确查询失败: null");
    }

    @Test
    public void testFAddOneSuccessWithFullArgs() {
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

        //新增
        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.addOne(sysVehicleTypeParam);

        assertThat(sysVehicleTypeDto, notNullValue());
        assertThat(sysVehicleTypeDto.getId(), notNullValue());
        assertThat(sysVehicleTypeDto.getParentId(), equalTo(sysVehicleTypeParam.getParentId()));
        assertThat(sysVehicleTypeDto.getLevel(), containsString(sysVehicleTypeParam.getParentId().toString()));
        assertThat(sysVehicleTypeDto.getNum(), equalTo(sysVehicleTypeParam.getNum()));
        assertThat(sysVehicleTypeDto.getType(), equalTo(sysVehicleTypeParam.getType()));
        assertThat(sysVehicleTypeDto.getName(), equalTo(sysVehicleTypeParam.getName()));
        assertThat(sysVehicleTypeDto.getRemark(), equalTo(sysVehicleTypeParam.getRemark()));
        assertThat(sysVehicleTypeDto.getState(), equalTo(sysVehicleTypeParam.getState()));
        assertThat(sysVehicleTypeDto.getOwnerEnterpriseId(), equalTo(sysVehicleTypeParam.getOwnerEnterpriseId()));
        assertThat(sysVehicleTypeDto.getOwnerEnterpriseName(), equalTo(sysVehicleTypeParam.getOwnerEnterpriseName()));
        assertThat(sysVehicleTypeDto.getOwnerUserId(), equalTo(sysVehicleTypeParam.getOwnerUserId()));
        assertThat(sysVehicleTypeDto.getOwnerUserName(), equalTo(sysVehicleTypeParam.getOwnerUserName()));
        assertThat(sysVehicleTypeDto.getCreateEnterpriseId(), equalTo(currentUser.getOwnerEnterpriseId()));
        assertThat(sysVehicleTypeDto.getCreateEnterpriseName(), equalTo(currentUser.getOwnerEnterpriseName()));
        assertThat(sysVehicleTypeDto.getModifiedEnterpriseId(), equalTo(currentUser.getOwnerEnterpriseId()));
        assertThat(sysVehicleTypeDto.getModifiedEnterpriseName(), equalTo(currentUser.getOwnerEnterpriseName()));
        assertThat(sysVehicleTypeDto.getCreateUserId(), equalTo(currentUser.getOwnerUserId()));
        assertThat(sysVehicleTypeDto.getCreateUserName(), equalTo(currentUser.getOwnerUserName()));
        assertThat(sysVehicleTypeDto.getModifiedUserId(), equalTo(currentUser.getOwnerUserId()));
        assertThat(sysVehicleTypeDto.getModifiedUserName(), equalTo(currentUser.getOwnerUserName()));
        assertThat(sysVehicleTypeDto.getTenantId(), equalTo(sysVehicleTypeParam.getTenantId()));
        assertThat(sysVehicleTypeDto.getTenantType(), equalTo(sysVehicleTypeParam.getTenantType()));
        assertThat(sysVehicleTypeDto.getTenantName(), equalTo(sysVehicleTypeParam.getTenantName()));
        assertThat(sysVehicleTypeDto.getGmtCreate(), notNullValue());
        assertThat(sysVehicleTypeDto.getGmtModified(), notNullValue());
        assertThat(sysVehicleTypeDto.getIsRoot(), equalTo(false));
        assertThat(sysVehicleTypeDto.getIsParent(), equalTo(false));
        assertThat(sysVehicleTypeDto.getChildren(), emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("新增成功: {}", JacksonTools.object2Json(sysVehicleTypeDto));
    }

    @Test
    public void testFAddOneSuccessWithRequiredArgs() {
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

        //新增
        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.addOne(sysVehicleTypeParam);

        assertThat(sysVehicleTypeDto, notNullValue());
        assertThat(sysVehicleTypeDto.getId(), notNullValue());
        assertThat(sysVehicleTypeDto.getParentId(), equalTo(sysVehicleTypeParam.getParentId()));
        assertThat(sysVehicleTypeDto.getLevel(), containsString(sysVehicleTypeParam.getParentId().toString()));
        assertThat(sysVehicleTypeDto.getNum(), equalTo(sysVehicleTypeParam.getNum()));
        assertThat(sysVehicleTypeDto.getType(), equalTo(sysVehicleTypeParam.getType()));
        assertThat(sysVehicleTypeDto.getName(), equalTo(sysVehicleTypeParam.getName()));
        assertThat(sysVehicleTypeDto.getRemark(), equalTo(sysVehicleTypeParam.getRemark()));
        assertThat(sysVehicleTypeDto.getState(), equalTo(sysVehicleTypeParam.getState()));
        assertThat(sysVehicleTypeDto.getOwnerEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleTypeDto.getOwnerEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleTypeDto.getOwnerUserId(), equalTo(userId));
        assertThat(sysVehicleTypeDto.getOwnerUserName(), equalTo(userName));
        assertThat(sysVehicleTypeDto.getCreateEnterpriseId(), equalTo(currentUser.getOwnerEnterpriseId()));
        assertThat(sysVehicleTypeDto.getCreateEnterpriseName(), equalTo(currentUser.getOwnerEnterpriseName()));
        assertThat(sysVehicleTypeDto.getModifiedEnterpriseId(), equalTo(currentUser.getOwnerEnterpriseId()));
        assertThat(sysVehicleTypeDto.getModifiedEnterpriseName(), equalTo(currentUser.getOwnerEnterpriseName()));
        assertThat(sysVehicleTypeDto.getCreateUserId(), equalTo(currentUser.getOwnerUserId()));
        assertThat(sysVehicleTypeDto.getCreateUserName(), equalTo(currentUser.getOwnerUserName()));
        assertThat(sysVehicleTypeDto.getModifiedUserId(), equalTo(currentUser.getOwnerUserId()));
        assertThat(sysVehicleTypeDto.getModifiedUserName(), equalTo(currentUser.getOwnerUserName()));
        assertThat(sysVehicleTypeDto.getTenantId(), equalTo(tenantId));
        assertThat(sysVehicleTypeDto.getTenantType(), equalTo(tenantType));
        assertThat(sysVehicleTypeDto.getTenantName(), equalTo(tenantName));
        assertThat(sysVehicleTypeDto.getGmtCreate(), notNullValue());
        assertThat(sysVehicleTypeDto.getGmtModified(), notNullValue());
        assertThat(sysVehicleTypeDto.getIsRoot(), equalTo(false));
        assertThat(sysVehicleTypeDto.getIsParent(), equalTo(false));
        assertThat(sysVehicleTypeDto.getChildren(), emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("新增成功: {}", JacksonTools.object2Json(sysVehicleTypeDto));
    }

    @Test
    public void testFAddOneFailureBecauseofInvalidArgs() {
        //默认新增无效参数
        SysVehicleTypeParam sysVehicleTypeParam = SysVehicleTypeParam.builder()
//                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("新增失败")
                .build();

        sysVehicleTypeParam.setRemark("新增失败:无效参数");
        sysVehicleTypeParam.setState(BaseStateEnum.OK.getValue());

        try {
            //新增
            SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.addOne(sysVehicleTypeParam);

            assertThat(sysVehicleTypeDto, nullValue());
        } catch (ValidationException e) {
            assertThat(e.getMessage(), notNullValue());
            assertThat(e.getMessage(), equalTo("parentId=上级车辆类型id不存在"));

            log.info("新增失败: {}", e);
        }
    }

    @Test
    public void testFAddOneFailureBecauseofIllegalArgs() {
        //默认新增非法参数
        SysVehicleTypeParam sysVehicleTypeParam = SysVehicleTypeParam.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("半挂车")
                .build();

        sysVehicleTypeParam.setRemark("新增失败:非法参数");
        sysVehicleTypeParam.setState(BaseStateEnum.OK.getValue());

        try {
            //新增
            SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.addOne(sysVehicleTypeParam);

            assertThat(sysVehicleTypeDto, nullValue());
        } catch (IllegalArgumentException e) {
            assertThat(e.getMessage(), notNullValue());
            assertThat(e.getMessage(), equalTo(String.format("同一层级下的车辆类型[%s]已存在", "半挂车")));

            log.info("新增失败: {}", e);
        }
    }

    @Test
    public void testGUpdateOneSuccessWithAlone() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName("更新成功");

        sysVehicleTypeParam.setRemark("更新成功:单独更新");

        //更新
        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.updateOne(sysVehicleTypeParam);

        assertThat(sysVehicleTypeDto, notNullValue());
        assertThat(sysVehicleTypeDto.getId(), equalTo(sysVehicleTypeParam.getId()));
        assertThat(sysVehicleTypeDto.getNum(), equalTo(sysVehicleTypeParam.getNum()));
        assertThat(sysVehicleTypeDto.getName(), equalTo(sysVehicleTypeParam.getName()));
        assertThat(sysVehicleTypeDto.getRemark(), equalTo(sysVehicleTypeParam.getRemark()));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeDto.getTenantId(), equalTo(tenantId));
            assertThat(sysVehicleTypeDto.getTenantType(), equalTo(tenantType));
            assertThat(sysVehicleTypeDto.getTenantName(), equalTo(tenantName));
        }
        assertThat(sysVehicleTypeDto.getGmtModified(), greaterThanOrEqualTo(DateUtils.addSeconds(new Date(), -10)));
        assertThat(sysVehicleTypeDto.getIsRoot(), equalTo(false));
        assertThat(sysVehicleTypeDto.getIsParent(), equalTo(false));
        assertThat(sysVehicleTypeDto.getChildren(), emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("更新成功: {}", JacksonTools.object2Json(sysVehicleTypeDto));
    }

    @Test
    public void testGUpdateOneSuccessWithDescendant() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

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

        //更新
        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.updateOne(sysVehicleTypeParam);

        assertThat(sysVehicleTypeDto, notNullValue());
        assertThat(sysVehicleTypeDto.getId(), equalTo(sysVehicleTypeParam.getId()));
        assertThat(sysVehicleTypeDto.getNum(), equalTo(sysVehicleTypeParam.getNum()));
        assertThat(sysVehicleTypeDto.getName(), equalTo(sysVehicleTypeParam.getName()));
        assertThat(sysVehicleTypeDto.getRemark(), equalTo(sysVehicleTypeParam.getRemark()));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeDto.getTenantId(), equalTo(tenantId));
            assertThat(sysVehicleTypeDto.getTenantType(), equalTo(tenantType));
            assertThat(sysVehicleTypeDto.getTenantName(), equalTo(tenantName));
        }
        assertThat(sysVehicleTypeDto.getGmtModified(), greaterThanOrEqualTo(DateUtils.addSeconds(new Date(), -10)));
        assertThat(sysVehicleTypeDto.getIsRoot(), equalTo(false));
        assertThat(sysVehicleTypeDto.getIsParent(), equalTo(true));
        assertThat(sysVehicleTypeDto.getChildren(), emptyCollectionOf(SysVehicleTypeDto.class));

        log.info("更新成功: {}", JacksonTools.object2Json(sysVehicleTypeDto));
    }

    @Test
    public void testGUpdateOneFailureBecauseofInvalidArgs() {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));

        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName("更新失败");
        sysVehicleTypeParam.setParentId(null);

        sysVehicleTypeParam.setRemark("更新失败:无效参数");

        try {
            //更新
            SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.updateOne(sysVehicleTypeParam);

            assertThat(sysVehicleTypeDto, nullValue());
        } catch (ValidationException e) {
            assertThat(e.getMessage(), notNullValue());
            assertThat(e.getMessage(), equalTo("parentId=上级车辆类型id不存在"));

            log.info("更新失败: {}", e);
        }
    }

    @Test
    public void testGUpdateOneFailureBecauseofIllegalArgs() {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeDto repeat = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName(repeat.getName());

        sysVehicleTypeParam.setRemark("更新失败:非法参数");

        try {
            //更新
            SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.updateOne(sysVehicleTypeParam);

            assertThat(sysVehicleTypeDto, nullValue());
        } catch (IllegalArgumentException e) {
            assertThat(e.getMessage(), notNullValue());
            assertThat(e.getMessage(), equalTo(String.format("同一层级下的车辆类型[%s]已存在", sysVehicleTypeParam.getName())));

            log.info("更新失败: {}", e);
        }
    }

    @Test
    public void testGUpdateOneFailureBecauseofNullIdArgs() {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        SysVehicleTypeParam sysVehicleTypeParam = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptParam(prepare));
        sysVehicleTypeParam.setNum(1);
        sysVehicleTypeParam.setName("更新失败");
        sysVehicleTypeParam.setId(null);

        sysVehicleTypeParam.setRemark("更新失败:主键缺失");

        try {
            //更新
            SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.updateOne(sysVehicleTypeParam);

            assertThat(sysVehicleTypeDto, nullValue());
        } catch (NullPointerException e) {
            assertThat(e.getMessage(), notNullValue());
            assertThat(e.getMessage(), equalTo(String.format("待更新的车辆类型id[%s]不存在", sysVehicleTypeParam.getName())));

            log.info("更新失败: {}", e);
        }
    }

    @Test
    public void testHDeleteOneSuccess() {
        //测试数据
        SysVehicleTypeDto prepare = prepareTestData(null);

        //删除
        sysVehicleTypeService.deleteOne(prepare.getId());

        log.info("删除成功");
    }

    @Test
    public void testHDeleteOneFailureBecauseofInvalidlArgs() {
        try {
            //删除
            sysVehicleTypeService.deleteOne(Long.MAX_VALUE);

            assertThat("正常情况下你看不到我,否则断言失败", true, equalTo(false));
        } catch (NullPointerException e) {
            assertThat(e.getMessage(), notNullValue());
            assertThat(e.getMessage(), equalTo(String.format("待删除的车辆类型id[%s]不存在", Long.MAX_VALUE)));

            log.info("删除失败: {}", e);
        }
    }

    @Test
    public void testHDeleteOneFailureBecauseofIllegalArgs() {
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

        try {
            //删除
            sysVehicleTypeService.deleteOne(parentDto.getId());

            assertThat("正常情况下你看不到我,否则断言失败", true, equalTo(false));
        } catch (IllegalArgumentException e) {
            assertThat(e.getMessage(), notNullValue());
            assertThat(e.getMessage(), equalTo(String.format("待删除的车辆类型[%s]存在子车辆类型", parentDto.getName())));

        }
    }

    /**
     * 准备测试数据
     *
     * @return 新增单个车辆类型
     */
    private SysVehicleTypeDto prepareTestData(SysVehicleTypeParam sysVehicleTypeParam) {
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

        //新增
        SysVehicleTypeDto sysVehicleTypeDto = sysVehicleTypeService.addOne(sysVehicleTypeParam);

        assertThat(sysVehicleTypeDto, notNullValue());
        assertThat(sysVehicleTypeDto.getId(), notNullValue());

        log.info("测试数据: {}", JacksonTools.object2Json(sysVehicleTypeDto));

        return sysVehicleTypeDto;
    }

}
```

[sysVehicleTypeServiceImplTest-split-1]: ../../../static/image/sysVehicleTypeServiceImplTest-split-1.png "sysVehicleTypeServiceImplTest-split-1"
[sysVehicleTypeServiceImplTest-split-2]: ../../../static/image/sysVehicleTypeServiceImplTest-split-2.png "sysVehicleTypeServiceImplTest-split-2"
[sysVehicleTypeServiceImplTest-split-3]: ../../../static/image/sysVehicleTypeServiceImplTest-split-3.png "sysVehicleTypeServiceImplTest-split-3"
[sysVehicleTypeServiceImplTest]: ../../../static/image/sysVehicleTypeServiceImplTest.png "sysVehicleTypeServiceImplTest"