# entityTest

测试目的在于检验 `mybaits-plus` 的 `ar` 模式是否可用,本质上是`mapper`的语法糖,因此只测试正常请求时表现如何,基本规则如下:

- `entityTest` 继承统一基类 `BaseTest` ,可根据实际情况决定是否重写某些方法;
- 测试方法名: `test+(大写字母)+(方法名)+SuccessWith+(条件)`;


1. 找到 `SysRoleTest` 角色 `entityTest` 测试类,并复制重命名为 `SysVehicleTypeTest`,同时打开两个窗口,方便编辑;
![sysVehicleTypeTest-split][sysVehicleTypeTest-split]

2. 根据实际情况,编写 `SysVehicleTypeTest` 文件;
![sysVehicleTypeTest][sysVehicleTypeTest]

SysVehicleTypeTest

```java
package com.snowdreams1006.securityplus.browser.module.system.entity;

import com.baomidou.mybatisplus.mapper.EntityWrapper;
import com.baomidou.mybatisplus.plugins.Page;
import com.google.common.base.Preconditions;
import com.snowdreams1006.securityplus.browser.base.BaseTest;
import com.snowdreams1006.securityplus.browser.base.enums.*;
import com.snowdreams1006.securityplus.browser.base.tools.LevelTools;
import com.snowdreams1006.securityplus.browser.base.tools.ShiroTools;
import com.snowdreams1006.securityplus.browser.base.tools.StringTools;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysUserDto;
import com.snowdreams1006.securityplus.core.tools.JacksonTools;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.time.DateUtils;
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runners.MethodSorters;

import java.util.Date;
import java.util.List;
import java.util.Objects;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;

/**
 * 系统级别车辆类型表 entityTest
 *
 * @author snowdreams1006
 * @date 2018-08-07
 */
@Slf4j
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class SysVehicleTypeTest extends BaseTest {

    @Test
    public void testAInsertSuccessWithFullField() {
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

        //初始化操作成功标识
        boolean result;

        //默认新增完整字段
        SysVehicleType sysVehicleType = SysVehicleType.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .level(LevelTools.ROOT_LEVEL)
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("新增成功")
                .build();

        sysVehicleType.setRemark("测试ActiveRecord模式下的insert:完整字段");
        sysVehicleType.setState(BaseStateEnum.OK.getValue());
        sysVehicleType.setDeleted(false);
        sysVehicleType.setVersion(1);
        sysVehicleType.setOwnerEnterpriseId(enterpriseId);
        sysVehicleType.setOwnerEnterpriseName(enterpriseName);
        sysVehicleType.setOwnerUserId(userId);
        sysVehicleType.setOwnerUserName(userName);
        sysVehicleType.setCreateEnterpriseId(enterpriseId);
        sysVehicleType.setCreateEnterpriseName(enterpriseName);
        sysVehicleType.setModifiedEnterpriseId(enterpriseId);
        sysVehicleType.setModifiedEnterpriseName(enterpriseName);
        sysVehicleType.setCreateUserId(userId);
        sysVehicleType.setCreateUserName(userName);
        sysVehicleType.setModifiedUserId(userId);
        sysVehicleType.setModifiedUserName(userName);
        //租户过滤可能会重复添加:取决于是否存在租户约束
        if (ShiroTools.isAdmin()) {
            sysVehicleType.setTenantId(tenantId);
        }
        sysVehicleType.setTenantType(tenantType);
        sysVehicleType.setTenantName(tenantName);
        sysVehicleType.setGmtCreate(new Date());
        sysVehicleType.setGmtModified(new Date());

        //新增
        result = sysVehicleType.insert();

        //再次查询最新属性
        sysVehicleType = sysVehicleType.selectById();

        assertThat(result, equalTo(true));
        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        assertThat(sysVehicleType.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(sysVehicleType.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(sysVehicleType.getNum(), equalTo(Integer.MAX_VALUE));
        assertThat(sysVehicleType.getType(), equalTo(VehicleTypeEnum.SEMITRAILER.getValue()));
        assertThat(sysVehicleType.getName(), equalTo("新增成功"));
        assertThat(sysVehicleType.getRemark(), equalTo("测试ActiveRecord模式下的insert:完整字段"));
        assertThat(sysVehicleType.getState(), equalTo(BaseStateEnum.OK.getValue()));
        assertThat(sysVehicleType.getDeleted(), equalTo(false));
        assertThat(sysVehicleType.getVersion(), equalTo(1));
        assertThat(sysVehicleType.getOwnerEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getOwnerEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getOwnerUserId(), equalTo(userId));
        assertThat(sysVehicleType.getOwnerUserName(), equalTo(userName));
        assertThat(sysVehicleType.getCreateEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getCreateEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getModifiedEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getModifiedEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getCreateUserId(), equalTo(userId));
        assertThat(sysVehicleType.getCreateUserName(), equalTo(userName));
        assertThat(sysVehicleType.getModifiedUserId(), equalTo(userId));
        assertThat(sysVehicleType.getModifiedUserName(), equalTo(userName));
        assertThat(sysVehicleType.getTenantId(), equalTo(tenantId));
        assertThat(sysVehicleType.getTenantType(), equalTo(tenantType));
        assertThat(sysVehicleType.getTenantName(), equalTo(tenantName));
        assertThat(sysVehicleType.getGmtCreate(), notNullValue());
        assertThat(sysVehicleType.getGmtModified(), notNullValue());

        log.info("新增成功: {}", JacksonTools.object2Json(sysVehicleType));
    }

    @Test
    public void testAInsertSuccessWithRequiredField() {
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

        //初始化操作成功标识
        boolean result;

        //默认新增必须字段
        SysVehicleType sysVehicleType = SysVehicleType.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .level(LevelTools.ROOT_LEVEL)
                .num(Integer.MAX_VALUE)
                .type(VehicleTypeEnum.SEMITRAILER.getValue())
                .name("新增成功")
                .build();

        sysVehicleType.setRemark("测试ActiveRecord模式下的insert:必须字段");
        sysVehicleType.setState(BaseStateEnum.OK.getValue());
        sysVehicleType.setDeleted(false);
        sysVehicleType.setVersion(1);
        sysVehicleType.setOwnerEnterpriseId(enterpriseId);
        sysVehicleType.setOwnerEnterpriseName(enterpriseName);
        sysVehicleType.setOwnerUserId(userId);
        sysVehicleType.setOwnerUserName(userName);
        //租户过滤可能会重复添加:取决于是否存在租户约束
        if (ShiroTools.isAdmin()) {
            sysVehicleType.setTenantId(tenantId);
        }
        sysVehicleType.setTenantType(tenantType);
        sysVehicleType.setTenantName(tenantName);

        //新增
        result = sysVehicleType.insert();

        //再次查询最新属性
        sysVehicleType = sysVehicleType.selectById();

        assertThat(result, equalTo(true));
        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        assertThat(sysVehicleType.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(sysVehicleType.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(sysVehicleType.getNum(), equalTo(Integer.MAX_VALUE));
        assertThat(sysVehicleType.getType(), equalTo(VehicleTypeEnum.SEMITRAILER.getValue()));
        assertThat(sysVehicleType.getName(), equalTo("新增成功"));
        assertThat(sysVehicleType.getRemark(), equalTo("测试ActiveRecord模式下的insert:必须字段"));
        assertThat(sysVehicleType.getState(), equalTo(BaseStateEnum.OK.getValue()));
        assertThat(sysVehicleType.getDeleted(), equalTo(false));
        assertThat(sysVehicleType.getVersion(), equalTo(1));
        assertThat(sysVehicleType.getOwnerEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getOwnerEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getOwnerUserId(), equalTo(userId));
        assertThat(sysVehicleType.getOwnerUserName(), equalTo(userName));
        assertThat(sysVehicleType.getCreateEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getCreateEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getModifiedEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getModifiedEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getCreateUserId(), equalTo(userId));
        assertThat(sysVehicleType.getCreateUserName(), equalTo(userName));
        assertThat(sysVehicleType.getModifiedUserId(), equalTo(userId));
        assertThat(sysVehicleType.getModifiedUserName(), equalTo(userName));
        assertThat(sysVehicleType.getTenantId(), equalTo(tenantId));
        assertThat(sysVehicleType.getTenantType(), equalTo(tenantType));
        assertThat(sysVehicleType.getTenantName(), equalTo(tenantName));
        assertThat(sysVehicleType.getGmtCreate(), notNullValue());
        assertThat(sysVehicleType.getGmtModified(), notNullValue());

        log.info("新增成功: {}", JacksonTools.object2Json(sysVehicleType));
    }

    @Test
    public void testBUpdateByIdSuccessWithSelectiveFiled() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long enterpriseId = currentUser.getOwnerEnterpriseId();
        String enterpriseName = currentUser.getOwnerEnterpriseName();
        Long userId = currentUser.getOwnerUserId();
        String userName = currentUser.getOwnerUserName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //初始化操作成功标识
        boolean result;

        //测试数据
        SysVehicleType sysVehicleType = prepareTestData(null);
        sysVehicleType.setNum(1);
        sysVehicleType.setName("更新成功");

        sysVehicleType.setRemark("测试ActiveRecord模式下的updateById:可选字段");

        //更新
        result = sysVehicleType.updateById();

        //再次查询最新属性
        sysVehicleType = sysVehicleType.selectById();

        assertThat(result, equalTo(true));
        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        assertThat(sysVehicleType.getNum(), equalTo(1));
        assertThat(sysVehicleType.getName(), equalTo("更新成功"));
        assertThat(sysVehicleType.getRemark(), equalTo("测试ActiveRecord模式下的updateById:可选字段"));
        assertThat(sysVehicleType.getModifiedEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getModifiedEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getModifiedUserId(), equalTo(userId));
        assertThat(sysVehicleType.getModifiedUserName(), equalTo(userName));
        assertThat(sysVehicleType.getGmtModified(), greaterThanOrEqualTo(DateUtils.addSeconds(new Date(), -10)));

        log.info("更新成功: {}", JacksonTools.object2Json(sysVehicleType));
    }

    @Test
    public void testCDeleteByIdSuccess() {
        //初始化操作成功标识
        boolean result;

        //测试数据
        SysVehicleType sysVehicleType = prepareTestData(null);

        //删除
        result = sysVehicleType.deleteById();

        //再次查询最新属性
        sysVehicleType = sysVehicleType.selectById();

        assertThat(result, equalTo(true));
        assertThat(sysVehicleType, nullValue());

        log.info("删除成功");
    }

    @Test
    public void testDSelectPageSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //分页查询
        SysVehicleType sysVehicleType = new SysVehicleType();
        Page<SysVehicleType> sysVehicleTypePage = sysVehicleType.selectPage(
                new Page<>(1, 3),
                new EntityWrapper<SysVehicleType>()
                        .eq("type", VehicleTypeEnum.SEMITRAILER.getValue())
        );

        assertThat(sysVehicleTypePage, notNullValue());
        assertThat(sysVehicleTypePage.getCurrent(), equalTo(1));
        assertThat(sysVehicleTypePage.getSize(), equalTo(3));
        assertThat(sysVehicleTypePage.getPages(), greaterThanOrEqualTo(0L));
        assertThat(sysVehicleTypePage.getTotal(), greaterThanOrEqualTo(0L));
        assertThat(sysVehicleTypePage.getRecords(), hasSize(greaterThanOrEqualTo(0)));
        assertThat(sysVehicleTypePage.getRecords(), everyItem(hasProperty("type", equalTo(VehicleTypeEnum.SEMITRAILER.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypePage.getRecords(), everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(sysVehicleTypePage.getRecords(), everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(sysVehicleTypePage.getRecords(), everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }
        log.info("分页查询成功: {}", JacksonTools.object2Json(sysVehicleTypePage));
    }

    @Test
    public void testESelectListSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询
        SysVehicleType sysVehicleType = new SysVehicleType();
        List<SysVehicleType> sysVehicleTypeList = sysVehicleType.selectList(new EntityWrapper<SysVehicleType>()
                .eq("type", VehicleTypeEnum.SEMITRAILER.getValue())
        );

        assertThat(sysVehicleTypeList, notNullValue());
        assertThat(sysVehicleTypeList, hasSize(greaterThanOrEqualTo(0)));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("type", equalTo(VehicleTypeEnum.SEMITRAILER.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }

        log.info("查询成功: {}", JacksonTools.object2Json(sysVehicleTypeList));
    }

    @Test
    public void testFSelectByIdSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //测试数据
        SysVehicleType sysVehicleType = prepareTestData(null);

        //查询
        sysVehicleType = sysVehicleType.selectById();

        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleType.getTenantId(), equalTo(tenantId));
            assertThat(sysVehicleType.getTenantType(), equalTo(tenantType));
            assertThat(sysVehicleType.getTenantName(), equalTo(tenantName));
        }

        log.info("查询成功: {}", JacksonTools.object2Json(sysVehicleType));
    }

    /**
     * 准备测试数据
     *
     * @param sysVehicleType 自定义的新增车辆类型对象,若 null, 则新增随机车辆类型对象,否则新增指定车辆类型对象
     * @return 新增成功的车辆类型对象
     */
    private SysVehicleType prepareTestData(SysVehicleType sysVehicleType) {
        //初始化操作成功标识
        boolean result;

        //初始化系统车辆类型实体对象 sysVehicleType
        if (Objects.isNull(sysVehicleType)) {
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

            log.info("准备测试登录用户: {}", JacksonTools.object2Json(currentUser));

            //默认测试数据
            sysVehicleType = SysVehicleType.builder()
                    .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                    .level(LevelTools.ROOT_LEVEL)
                    .num(Integer.MAX_VALUE)
                    .type(VehicleTypeEnum.SEMITRAILER.getValue())
                    .name("测试数据:" + StringTools.getRandomString(6))
                    .build();

            sysVehicleType.setRemark("测试ActiveRecord模式下的Insert:" + StringTools.getRandomString(6));
            sysVehicleType.setState(BaseStateEnum.OK.getValue());
            sysVehicleType.setDeleted(false);
            sysVehicleType.setVersion(1);
            sysVehicleType.setOwnerEnterpriseId(enterpriseId);
            sysVehicleType.setOwnerEnterpriseName(enterpriseName);
            sysVehicleType.setOwnerUserId(userId);
            sysVehicleType.setOwnerUserName(userName);
            sysVehicleType.setCreateEnterpriseId(enterpriseId);
            sysVehicleType.setCreateEnterpriseName(enterpriseName);
            sysVehicleType.setModifiedEnterpriseId(enterpriseId);
            sysVehicleType.setModifiedEnterpriseName(enterpriseName);
            sysVehicleType.setCreateUserId(userId);
            sysVehicleType.setCreateUserName(userName);
            sysVehicleType.setModifiedUserId(userId);
            sysVehicleType.setModifiedUserName(userName);
            //租户过滤可能会重复添加:取决于是否存在租户约束
            if (ShiroTools.isAdmin()) {
                sysVehicleType.setTenantId(tenantId);
            }
            sysVehicleType.setTenantType(tenantType);
            sysVehicleType.setTenantName(tenantName);
            sysVehicleType.setGmtCreate(new Date());
            sysVehicleType.setGmtModified(new Date());
        }

        //新增
        result = sysVehicleType.insert();

        //再次查询最新属性
        sysVehicleType = sysVehicleType.selectById();

        assertThat(result, equalTo(true));
        assertThat(sysVehicleType, notNullValue());

        log.info("测试数据: {}", JacksonTools.object2Json(sysVehicleType));

        return sysVehicleType;
    }

}
```

[sysVehicleTypeTest-split]: ../../../static/image/sysVehicleTypeTest-split.png "sysVehicleTypeTest-split"
[sysVehicleTypeTest]: ../../../static/image/sysVehicleTypeTest.png "sysVehicleTypeTest"