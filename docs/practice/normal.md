# 常规开发

开发流程: 后端代码 -> 前端页面 -> 分配权限 -> 开发完成

下面以 `车辆类型` 为例,说明如何进行 `增删改查` 简单业务,效果如图

![最终效果预览][normal-preview]

根据业务特点分析,发现 `车辆类型` 功能和 `角色` 功能基本类似,所以打算采用复制粘贴的方式快速修改完成!

## 建立数据表

1. 作者以 `mysql workbench` 客户端为例,首先找到 `sys_role` 角色表,并查看表结构;
![角色表结构][sys_role_structure]

2. 按需选择 `sys_role` 数据表字段,右键 `copy` ,准备复制到 `sys_vehicle_type` 数据表;
![复制数据表][sys_role_copy]

3. 新建 `sys_vehilce_type` 数据表,并粘贴到该数据表中;
![粘贴数据表][sys_vehicle_type_paste]
![修改前数据表][sys_vehicle_type_unmodified]

4. 根据实际业务修改数据表结构;
![修改后数据表][sys_vehicle_type_modified]

sys_vehicle_type

```mysql
CREATE TABLE `sys_vehicle_type` (
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '车辆类型id',
  `parent_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '直接上级id,0表示无上级',
  `level` varchar(255) NOT NULL DEFAULT '' COMMENT '层级',
  `num` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '序号',
  `type` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '类型:1表示牵引车,2表示半挂车,3表示全挂车',
  `name` varchar(50) NOT NULL DEFAULT '' COMMENT '名称',
  `remark` varchar(255) NOT NULL DEFAULT '' COMMENT '备注',
  `state` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '状态,1启用中,2已冻结,3已删除',
  `is_deleted` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT '逻辑删除配置,1表示删除,0表示未删除',
  `version` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '乐观锁标志位',
  `owner_enterprise_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '所有者企业 id',
  `owner_enterprise_name` varchar(50) NOT NULL DEFAULT '' COMMENT '所有者企业名称',
  `owner_user_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '所有者用户 id',
  `owner_user_name` varchar(50) NOT NULL DEFAULT '' COMMENT '所有者用户名称',
  `create_enterprise_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '创建企业 id',
  `create_enterprise_name` varchar(50) NOT NULL DEFAULT '' COMMENT '创建企业名称',
  `modified_enterprise_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '更新企业 id',
  `modified_enterprise_name` varchar(50) NOT NULL DEFAULT '' COMMENT '更新企业名称',
  `create_user_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '创建用户 id',
  `create_user_name` varchar(50) NOT NULL DEFAULT '' COMMENT '创建用户名称',
  `modified_user_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '更新用户 id',
  `modified_user_name` varchar(50) NOT NULL DEFAULT '' COMMENT '更新企业名称',
  `tenant_id` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '租户 id\n',
  `tenant_type` bigint(11) unsigned NOT NULL DEFAULT '0' COMMENT '租户类型,1表示企业租户,2表示个人租户\n',
  `tenant_name` varchar(50) NOT NULL DEFAULT '' COMMENT '租户名称',
  `gmt_create` datetime DEFAULT NULL COMMENT '创建时间',
  `gmt_modified` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `id_UNIQUE` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=303 DEFAULT CHARSET=utf8 COMMENT='系统级别车辆类型表';
```

## 更新 `svn` 服务器

正式`coding`前,先保证当前工作空间中代码是最新状态,所以选中可能会更新的目录,右键 `subversion` -> `update Directory...`
![更新 svn 服务器][svn-update]
![正在更新 svn 服务器][svn-updating]

## entity 实体类

`entity` 实体类中各属性与数据表字段一一对应对应，基本规约如下: 

- `entity` 继承统一基类 `BaseEntity` ,可根据实际情况决定是否重写某些属性;
- `entity` 添加 `@Getter @Setter @Builder @NoArgsConstructor @TableName` 等注解简化实体类;
- 类名：`大驼峰命名` 规范,默认注释同数据表注释,并添加 `@author 和 @date` 注解说明;
- 属性名：`小驼峰命名`规范,默认注释同数据表字段,并添加 `@TableField @JsonView @ApiModelProperty ` 等注解说明;


1. 打开 `idea` 开发工具,找到 `SysRole` 角色实体类,右键复制并重命名 `SysVehicleType`;
![重命名sysVehicleType][sysVehicleType-unmodified]

2. 重命名成功后系统提示是否加入版本控制,确定,不然每一次新增文件都会提示!
![加入版本控制提示][svn-add-confirm]

3. 同时打开 `SysRole` 和 `SysVehicleType` 文件,选中 `SysVehicleType` 并右键 `Split-vertically` 分为左右两个窗口,方便复制粘贴;
![准备垂直分窗口][split-vertically-prepare]
![垂直分窗口完成][split-vertically-complete]

4. 根据数据库字段一一映射实体类的属性;
![基类属性][baseEntity]
![sysVehicleType][sysVehicleType]

SysVehicleType.java

```java
package com.snowdreams1006.securityplus.browser.module.system.entity;

import com.baomidou.mybatisplus.annotations.TableField;
import com.baomidou.mybatisplus.annotations.TableName;
import com.baomidou.mybatisplus.mapper.SqlCondition;
import com.fasterxml.jackson.annotation.JsonView;
import com.snowdreams1006.securityplus.browser.base.entity.BaseEntity;
import io.swagger.annotations.ApiModelProperty;
import lombok.*;

/**
 * 系统级别车辆类型表 entity
 *
 * @author snowdreams1006
 * @date 2018-08-07
 */
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@TableName("sys_vehicle_type")
public class SysVehicleType extends BaseEntity<SysVehicleType> {

    /**
     * 上级角色id,0表示无上级,简单视图
     */
    @TableField("parent_id")
    @JsonView(SimpleView.class)
    @ApiModelProperty(value = "上级角色id")
    private Long parentId;

    /**
     * 层级,详细视图
     */
    @TableField(value = "level", condition = SqlCondition.LIKE)
    @JsonView(DetailView.class)
    @ApiModelProperty(value = "层级")
    private String level;

    /**
     * 排序,简单视图
     */
    @TableField("num")
    @JsonView(SimpleView.class)
    @ApiModelProperty(value = "排序")
    private Integer num;

    /**
     * 类型,简单视图
     */
    @TableField("type")
    @JsonView(SimpleView.class)
    @ApiModelProperty(value = "类型")
    private Integer type;

    /**
     * 名称,简单视图
     */
    @TableField(value = "name", condition = SqlCondition.LIKE)
    @JsonView(SimpleView.class)
    @ApiModelProperty(value = "名称")
    private String name;

}
```

## param 参数类

参数类的设计初衷在于区分出实体类和参数类是不一样的!


例如创建时间,修改时间,操作人员等这些字段不能由前端传递给后端,而是应该后端自动判断生成,所以抽象出 `param` 模型,基本规则如下:

- `param` 继承统一基类 `BaseParam` ,可根据实际情况决定是否重写某些属性;
- `param` 添加 `@Getter @Setter @Builder @NoArgsConstructor @ApiModel` 等注解简化参数类;
- 类名：`大驼峰命名+Param` 规范,默认注释业务模型,并添加 `@author 和 @date` 注解说明;
- 属性名：`小驼峰命名`规范,默认注释业务模型,并添加 `@NotNull @Min @ApiModelProperty ` 等注解说明;


1. 找到 `SysRoleParam` 角色参数类,并复制重命名为 `SysVehicleTypeParam`,同时打开两个窗口,方便编辑;
![sysVehicleTypeParam-split][sysVehicleTypeParam-split]

2. 根据实际情况,编写 `SysVehicleTypeParam` 参数类;
![sysVehicleTypeParam][sysVehicleTypeParam]

SysVehicleTypeParam.java

```java
package com.snowdreams1006.securityplus.browser.module.system.param;

import com.snowdreams1006.securityplus.browser.base.param.BaseParam;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.*;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotBlank;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;

/**
 * 车辆类型 param
 *
 * @author snowdreams1006
 * @date 2018-08-07
 */
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@ApiModel(description = "车辆类型参数")
public class SysVehicleTypeParam extends BaseParam {

    /**
     * 直接上级id,0表示无上级
     */
    @NotNull(message = "上级车辆类型id不存在")
    @Min(value = 0, message = "上级车辆类型id必须大于零")
    @ApiModelProperty(value = "直接上级id", required = true)
    private Long parentId;

    /**
     * 排序
     */
    @NotNull(message = "展示顺序不存在")
    @Min(value = 0, message = "展示顺序必须大于零")
    @ApiModelProperty(value = "排序", required = true)
    private Integer num;

    /**
     * 类型
     */
    @NotNull(message = "类型不存在")
    @Min(value = 0, message = "类型必须大于零")
    @ApiModelProperty(value = "类型", required = true)
    private Integer type;

    /**
     * 名称
     */
    @NotBlank(message = "车辆类型名称不存在")
    @Length(min = 1, max = 50, message = "车辆类型名称长度需要在1-50个字符之间")
    @ApiModelProperty(value = "名称", required = true)
    private String name;

}
```

## query 查询类

顾名思义是对查询条件进行抽象而成的 `query` 模型,用于解决 `Map` 传递参数引发的诸多问题,基本规则如下:

- `query` 继承统一基类 `BaseQuery` ,可根据实际情况决定是否重写某些属性;
- `query` 添加 `@Getter @Setter @Builder @NoArgsConstructor` 等注解简化参数类;
- 类名：`大驼峰命名+Query` 规范,默认注释业务模型,并添加 `@author 和 @date` 注解说明;
- 属性名：`小驼峰命名`规范,默认注释业务模型;


1. 找到 `SysRoleQuery` 角色查询类,并复制重命名为 `SysVehicleTypeQuery`,同时打开两个窗口,方便编辑;
![sysVehicleTypeQuery-split][sysVehicleTypeQuery-split]

2. 根据实际情况,编写 `SysVehicleTypeQuery` 查询类;
![sysVehicleTypeQuery][sysVehicleTypeQuery]

SysVehicleTypeQuery.java

```java
package com.snowdreams1006.securityplus.browser.module.system.query;

import com.snowdreams1006.securityplus.browser.base.query.BaseQuery;
import lombok.*;

import java.util.List;

/**
 * 车辆类型 query
 *
 * @author snowdreams1006
 * @date 2018-08-07
 */
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class SysVehicleTypeQuery extends BaseQuery {

    /**
     * 名称
     */
    private String name;

    /**
     * 类型
     */
    private List<Integer> type;

}
```

## dto 传输类

参数类的设计初衷在于区分出实体类和传输类是不一样的!


例如 `ztree` 节点中某节点是否选中,是否是父节点等信息,这些都是动态信息,不适合永久存储数据库,因此抽象出 `dto` 模型动态解决此类需求,基本规则如下:

- `dto` 继承自己的实体类 `entity` ,根据实际情况扩展某些属性或新增属性;
- `dto` 添加 `@Getter @Setter @NoArgsConstructor @ApiModel` 等注解简化参数类;
- 类名：`大驼峰命名+Dto` 规范,默认注释业务模型,并添加 `@author 和 @date` 注解说明;
- 属性名：`小驼峰命名`规范,默认注释业务模型;


1. 找到 `SysRoleDto` 角色传输类,并复制重命名为 `SysVehicleTypeDto`,同时打开两个窗口,方便编辑;
![sysVehicleTypeDto-split][sysVehicleTypeDto-split]

2. 根据实际情况,编写 `SysVehicleTypeDto` 传输类;
![sysVehicleTypeDto][sysVehicleTypeDto]

SysVehicleTypeDto

```java
package com.snowdreams1006.securityplus.browser.module.system.dto;

import com.fasterxml.jackson.annotation.JsonView;
import com.google.common.collect.Lists;
import com.snowdreams1006.securityplus.browser.base.tools.LevelTools;
import com.snowdreams1006.securityplus.browser.module.system.entity.SysVehicleType;
import com.snowdreams1006.securityplus.browser.module.system.param.SysVehicleTypeParam;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.*;
import org.springframework.beans.BeanUtils;

import java.util.List;
import java.util.Objects;

/**
 * 车辆类型 dto
 *
 * @author snowdreams1006
 * @date 2018-05-17
 */
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@ApiModel(description = "车辆类型对象")
public class SysVehicleTypeDto extends SysVehicleType {

    /**
     * 是否为根节点,简单视图
     */
    @JsonView(SimpleView.class)
    @ApiModelProperty(value = "是否为根节点")
    private Boolean isRoot;

    /**
     * 是否为父节点,简单视图
     */
    @JsonView(SimpleView.class)
    @ApiModelProperty(value = "是否为父节点")
    private Boolean isParent;

    /**
     * 是否选中,详细视图
     */
    @JsonView(DetailView.class)
    @ApiModelProperty(value = "是否选中")
    private Boolean checked;

    /**
     * 是否展开,详细视图
     */
    @JsonView(DetailView.class)
    @ApiModelProperty(value = "是否展开")
    private Boolean open;

    /**
     * 子车辆类型,详细视图
     */
    @JsonView(DetailView.class)
    @ApiModelProperty(value = "子车辆类型")
    private List<SysVehicleTypeDto> children = Lists.newArrayList();

    /**
     * 默认顶级车辆类型
     */
    public static SysVehicleTypeDto createRoot() {
        SysVehicleTypeDto root = new SysVehicleTypeDto();

        root.setId(Long.valueOf(LevelTools.ROOT_LEVEL));
        root.setParentId(Long.valueOf(LevelTools.ROOT_LEVEL));
        root.setLevel(LevelTools.ROOT_LEVEL);
        root.setName(LevelTools.ROOT_NAME);
        root.setType(Integer.valueOf(LevelTools.ROOT_LEVEL));
        root.setNum(Integer.valueOf(LevelTools.ROOT_LEVEL));

        root.setIsRoot(true);
        root.setIsParent(true);
        root.setChecked(true);
        root.setOpen(true);

        return root;
    }

    /**
     * 将车辆类型entity适配成车辆类型dto
     *
     * @param sysVehicleType 车辆类型entity
     * @return 车辆类型dto
     */
    public static SysVehicleTypeDto entityAdaptDto(SysVehicleType sysVehicleType) {
        //若接收null,则返回 null, 则具体逻由调用者处理
        if (Objects.isNull(sysVehicleType)) {
            return null;
        }
        SysVehicleTypeDto sysVehicleTypeDto = new SysVehicleTypeDto();
        BeanUtils.copyProperties(sysVehicleType, sysVehicleTypeDto);
        return sysVehicleTypeDto;
    }

    /**
     * 将车辆类型entity适配成车辆类型param
     *
     * @param sysVehicleType 车辆类型entity
     * @return 车辆类型param
     */
    public static SysVehicleTypeParam entityAdaptParam(SysVehicleType sysVehicleType) {
        //若接收null,则返回 null, 则具体逻由调用者处理
        if (Objects.isNull(sysVehicleType)) {
            return null;
        }
        SysVehicleTypeParam sysVehicleTypeParam = new SysVehicleTypeParam();
        BeanUtils.copyProperties(sysVehicleType, sysVehicleTypeParam);
        return sysVehicleTypeParam;
    }

    /**
     * 将车辆类型dto适配成车辆类型entity
     *
     * @param sysVehicleTypeDto 车辆类型 dto
     * @return 车辆类型entity
     */
    public static SysVehicleType dtoAdaptEntity(SysVehicleTypeDto sysVehicleTypeDto) {
        //若 接收null,则返回 null, 则具体逻由调用者处理
        if (Objects.isNull(sysVehicleTypeDto)) {
            return null;
        }
        SysVehicleType sysVehicleType = new SysVehicleType();
        BeanUtils.copyProperties(sysVehicleTypeDto, sysVehicleType);
        return sysVehicleType;
    }

    /**
     *  将车辆类型dto适配成车辆类型param
     *
     * @param sysVehicleTypeDto 车辆类型 dto
     * @return 车辆类型entity
     */
    public static SysVehicleTypeParam dtoAdaptParam(SysVehicleTypeDto sysVehicleTypeDto) {
        //若 接收null,则返回 null, 则具体逻由调用者处理
        if (Objects.isNull(sysVehicleTypeDto)) {
            return null;
        }
        SysVehicleTypeParam sysVehicleTypeParam = new SysVehicleTypeParam();
        BeanUtils.copyProperties(sysVehicleTypeDto, sysVehicleTypeParam);
        return sysVehicleTypeParam;
    }

    /**
     * 将车辆类型param适配成车辆类型entity
     *
     * @param sysVehicleTypeParam 车辆类型param
     * @return 车辆类型entity
     */
    public static SysVehicleType paramAdaptEntity(SysVehicleTypeParam sysVehicleTypeParam) {
        //若 接收null,则返回 null, 则具体逻由调用者处理
        if (Objects.isNull(sysVehicleTypeParam)) {
            return null;
        }
        SysVehicleType sysVehicleType = new SysVehicleType();
        BeanUtils.copyProperties(sysVehicleTypeParam, sysVehicleType);
        return sysVehicleType;
    }

    /**
     * 将车辆类型param适配成车辆类型dto
     *
     * @param sysVehicleTypeParam 车辆类型param
     * @return 车辆类型dto
     */
    public static SysVehicleTypeDto paramAdaptDto(SysVehicleTypeParam sysVehicleTypeParam) {
        //若 接收null,则返回 null, 则具体逻由调用者处理
        if (Objects.isNull(sysVehicleTypeParam)) {
            return null;
        }
        SysVehicleTypeDto sysVehicleTypeDto = new SysVehicleTypeDto();
        BeanUtils.copyProperties(sysVehicleTypeParam, sysVehicleTypeDto);
        return sysVehicleTypeDto;
    }

}
```

## mapper.java

基本规则如下:

- `mapper` 继承自己的基类 `BaseMapper` ,根据实际情况扩展自定义`sql`语句;
- 类名：`大驼峰命名+Mapper` 规范,默认注释同数据表注释,并添加 `@author 和 @date` 注解说明;


1. 找到 `SysRoleMapper` 角色 `mapper` 类,并复制重命名为 `SysVehicleTypeMapper`,同时打开两个窗口,方便编辑;
![sysVehicleTypeMapper-split-java][sysVehicleTypeMapper-split-java]

2. 根据实际情况,编写 `SysVehicleTypeMapper` 接口;
![sysVehicleTypeMapper-java][sysVehicleTypeMapper-java]

SysVehicleTypeMapper.java

```java
package com.snowdreams1006.securityplus.browser.module.system.mapper;

import com.baomidou.mybatisplus.mapper.BaseMapper;
import com.baomidou.mybatisplus.plugins.Page;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysVehicleTypeDto;
import com.snowdreams1006.securityplus.browser.module.system.entity.SysVehicleType;
import com.snowdreams1006.securityplus.browser.module.system.query.SysVehicleTypeQuery;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * 系统级别车辆类型表 mapper
 *
 * @author snowdreams1006
 * @date 2018-08-07
 */
public interface SysVehicleTypeMapper extends BaseMapper<SysVehicleType> {

    /**
     * 查询当前车辆类型的后代车辆类型
     *
     * @param parent       当前车辆类型
     * @param sysVehicleTypeQuery 后代车辆类型过滤条件
     * @return 后代车辆类型
     */
    List<SysVehicleType> listDescendant(@Param("parent") SysVehicleTypeDto parent, @Param("sysVehicleTypeQuery") SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 查询当前车辆类型的子车辆类型
     *
     * @param parent       当前车辆类型
     * @param sysVehicleTypeQuery 子车辆类型过滤条件
     * @return 子车辆类型
     */
    List<SysVehicleType> listChildren(@Param("parent") SysVehicleTypeDto parent, @Param("sysVehicleTypeQuery") SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 查询当前车辆类型的子车辆类型统计值
     *
     * @param parent       当前车辆类型
     * @param sysVehicleTypeQuery 子车辆类型过滤条件
     * @return 子车辆类型统计值
     */
    Integer countChildren(@Param("parent") SysVehicleTypeDto parent, @Param("sysVehicleTypeQuery") SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 分页查询车辆类型
     *
     * @param page         分页条件
     * @param parent       当前车辆类型
     * @param sysVehicleTypeQuery 查询条件
     * @return 车辆类型列表
     */
    List<SysVehicleType> listPage(@Param("page") Page<SysVehicleTypeDto> page, @Param("parent") SysVehicleTypeDto parent, @Param("sysVehicleTypeQuery") SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 条件查询车辆类型
     *
     * @param sysVehicleTypeQuery 查询条件
     * @return 车辆类型列表
     */
    List<SysVehicleType> listCondition(@Param("sysVehicleTypeQuery") SysVehicleTypeQuery sysVehicleTypeQuery);

}
```

## mapper.xml

基本规则如下:

- `resultMap` 的 `id=entity+BaseResultMap`,例如 `id="sysVehicleTypeBaseResultMap"`;
- `sql` 的 `id=entityBaseColumns`,例如 `sysVehicleTypeBaseColumns`


1. 找到 `SysRoleMapper.xml` 角色 `mapper.xml` 文件,并复制重命名为 `SysVehicleTypeMapper.xml`,同时打开两个窗口,方便编辑;
![sysVehicleTypeMapper-split-xml-1][sysVehicleTypeMapper-split-xml-1]
![sysVehicleTypeMapper-split-xml-2][sysVehicleTypeMapper-split-xml-2]

2. 根据实际情况,编写 `SysVehicleTypeMapper.xml` 文件;
![sysVehicleTypeMapper-xml][sysVehicleTypeMapper-xml]

SysVehicleTypeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.snowdreams1006.securityplus.browser.module.system.mapper.SysVehicleTypeMapper">

    <!-- 通用查询结果列映射关系 -->
    <resultMap id="sysVehicleTypeBaseResultMap" type="com.snowdreams1006.securityplus.browser.module.system.entity.SysVehicleType">
        <id column="id" property="id"/>
        <result column="parent_id" property="parentId"/>
        <result column="level" property="level"/>
        <result column="num" property="num"/>
        <result column="type" property="type"/>
        <result column="name" property="name"/>
        <result column="remark" property="remark"/>
        <result column="state" property="state"/>
        <result column="is_deleted" property="deleted"/>
        <result column="version" property="version"/>
        <result column="owner_enterprise_id" property="ownerEnterpriseId"/>
        <result column="owner_enterprise_name" property="ownerEnterpriseName"/>
        <result column="owner_user_id" property="ownerUserId"/>
        <result column="owner_user_name" property="ownerUserName"/>
        <result column="create_enterprise_id" property="createEnterpriseId"/>
        <result column="create_enterprise_name" property="createEnterpriseName"/>
        <result column="modified_enterprise_id" property="modifiedEnterpriseId"/>
        <result column="modified_enterprise_name" property="modifiedEnterpriseName"/>
        <result column="create_user_id" property="createUserId"/>
        <result column="create_user_name" property="createUserName"/>
        <result column="modified_user_id" property="modifiedUserId"/>
        <result column="modified_user_name" property="modifiedUserName"/>
        <result column="tenant_id" property="tenantId"/>
        <result column="tenant_type" property="tenantType"/>
        <result column="tenant_name" property="tenantName"/>
        <result column="gmt_create" property="gmtCreate"/>
        <result column="gmt_modified" property="gmtModified"/>
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="sysVehicleTypeBaseColumns">
        id, parent_id, level, num, type, name,
        remark, state, is_deleted, version, owner_enterprise_id, owner_enterprise_name, owner_user_id, owner_user_name,
        create_enterprise_id, create_enterprise_name, modified_enterprise_id, modified_enterprise_name,
        create_user_id, create_user_name, modified_user_id, modified_user_name, tenant_id, tenant_type, tenant_name, gmt_create, gmt_modified
    </sql>

    <select id="listDescendant" resultMap="sysVehicleTypeBaseResultMap">
        select
        <include refid="sysVehicleTypeBaseColumns"/>
        from sys_vehicle_type
        <where>
            is_deleted = 0
            AND state != 3
            <if test="parent != null and parent.id != null and parent.id != 0">
                AND level like CONCAT("%",#{parent.level},".",#{parent.id},"%")
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.name != null and sysVehicleTypeQuery.name.trim() != ''">
                AND name like CONCAT("%",TRIM(#{sysVehicleTypeQuery.name}),"%")
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.type != null and sysVehicleTypeQuery.type.size() > 0">
                AND type IN
                <foreach collection="sysVehicleTypeQuery.type" item="typeItem" open="(" close=")" separator=",">
                    #{typeItem}
                </foreach>
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.startDate != null and sysVehicleTypeQuery.endDate != null ">
                AND gmt_create BETWEEN #{sysVehicleTypeQuery.startDate} AND #{sysVehicleTypeQuery.endDate}
            </if>
        </where>
        order by level ,num ASC
    </select>

    <select id="listChildren" resultMap="sysVehicleTypeBaseResultMap">
        select
        <include refid="sysVehicleTypeBaseColumns"/>
        from sys_vehicle_type
        <where>
            is_deleted = 0
            AND state != 3
            <if test="parent != null and parent.id != null">
                AND parent_id = #{parent.id}
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.name != null and sysVehicleTypeQuery.name.trim() != ''">
                AND name like CONCAT("%",TRIM(#{sysVehicleTypeQuery.name}),"%")
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.type != null and sysVehicleTypeQuery.type.size() > 0">
                AND type IN
                <foreach collection="sysVehicleTypeQuery.type" item="typeItem" open="(" close=")" separator=",">
                    #{typeItem}
                </foreach>
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.startDate != null and sysVehicleTypeQuery.endDate != null ">
                AND gmt_create BETWEEN #{sysVehicleTypeQuery.startDate} AND #{sysVehicleTypeQuery.endDate}
            </if>
        </where>
        order by level ,num ASC
    </select>

    <select id="countChildren" resultType="java.lang.Integer">
        select
        count(*)
        from sys_vehicle_type
        <where>
            is_deleted = 0
            AND state != 3
            <if test="parent != null and parent.id != null">
                AND parent_id = #{parent.id}
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.name != null and sysVehicleTypeQuery.name.trim() != ''">
                AND name like CONCAT("%",TRIM(#{sysVehicleTypeQuery.name}),"%")
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.type != null and sysVehicleTypeQuery.type.size() > 0">
                AND type IN
                <foreach collection="sysVehicleTypeQuery.type" item="typeItem" open="(" close=")" separator=",">
                    #{typeItem}
                </foreach>
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.startDate != null and sysVehicleTypeQuery.endDate != null ">
                AND gmt_create BETWEEN #{sysVehicleTypeQuery.startDate} AND #{sysVehicleTypeQuery.endDate}
            </if>
        </where>
    </select>

    <select id="listPage" resultMap="sysVehicleTypeBaseResultMap">
        select
        <include refid="sysVehicleTypeBaseColumns"/>
        from sys_vehicle_type
        <where>
            is_deleted = 0
            AND state != 3
            <if test="parent != null and parent.id != null">
                AND parent_id = #{parent.id}
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.name != null and sysVehicleTypeQuery.name.trim() != ''">
                AND name like CONCAT("%",TRIM(#{sysVehicleTypeQuery.name}),"%")
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.type != null and sysVehicleTypeQuery.type.size() > 0">
                AND type IN
                <foreach collection="sysVehicleTypeQuery.type" item="typeItem" open="(" close=")" separator=",">
                    #{typeItem}
                </foreach>
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.startDate != null and sysVehicleTypeQuery.endDate != null ">
                AND gmt_create BETWEEN #{sysVehicleTypeQuery.startDate} AND #{sysVehicleTypeQuery.endDate}
            </if>
        </where>
    </select>

    <select id="listCondition" resultMap="sysVehicleTypeBaseResultMap">
        select
        <include refid="sysVehicleTypeBaseColumns"/>
        from sys_vehicle_type
        <where>
            is_deleted = 0
            AND state != 3
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.id != null">
                AND id = #{sysVehicleTypeQuery.id}
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.name != null and sysVehicleTypeQuery.name.trim() != ''">
                AND name like CONCAT("%",TRIM(#{sysVehicleTypeQuery.name}),"%")
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.type != null and sysVehicleTypeQuery.type.size() > 0">
                AND type IN
                <foreach collection="sysVehicleTypeQuery.type" item="typeItem" open="(" close=")" separator=",">
                    #{typeItem}
                </foreach>
            </if>
            <if test="sysVehicleTypeQuery != null and sysVehicleTypeQuery.startDate != null and sysVehicleTypeQuery.endDate != null ">
                AND gmt_create BETWEEN #{sysVehicleTypeQuery.startDate} AND #{sysVehicleTypeQuery.endDate}
            </if>
        </where>
        order by level ,num ASC
    </select>

</mapper>
```

## mapperTest

测试目的在于检验 `mybaits-plus` 通用 `sql` 是否可用,以及自定义 `sql` 是否正常运行,因此只测试正常请求时表现如何,基本规则如下:

- `mapperTest` 继承统一基类 `BaseTest` ,可根据实际情况决定是否重写某些方法;
- 测试方法名: `test+(大写字母)+(方法名)+SuccessWith+(条件)`;


1. 找到 `SysRoleMapperTest` 角色 `mapperTest` 测试类,并复制重命名为 `SysVehicleTypeMapperTest`,同时打开两个窗口,方便编辑;
![sysVehicleTypeMapperTest-split-1][sysVehicleTypeMapperTest-split-1]
![sysVehicleTypeMapperTest-split-2][sysVehicleTypeMapperTest-split-2]

2. 根据实际情况,编写 `SysVehicleTypeMapperTest` 文件;
![sysVehicleTypeMapperTest][sysVehicleTypeMapperTest]

SysVehicleTypeMapperTest

```java
package com.snowdreams1006.securityplus.browser.module.system.mapper;

import com.baomidou.mybatisplus.mapper.EntityWrapper;
import com.baomidou.mybatisplus.plugins.Page;
import com.google.common.base.Preconditions;
import com.snowdreams1006.securityplus.browser.base.BaseTest;
import com.snowdreams1006.securityplus.browser.base.enums.BaseStateEnum;
import com.snowdreams1006.securityplus.browser.base.enums.RoleTypeEnum;
import com.snowdreams1006.securityplus.browser.base.enums.VehicleTypeEnum;
import com.snowdreams1006.securityplus.browser.base.tools.LevelTools;
import com.snowdreams1006.securityplus.browser.base.tools.ShiroTools;
import com.snowdreams1006.securityplus.browser.base.tools.StringTools;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysUserDto;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysVehicleTypeDto;
import com.snowdreams1006.securityplus.browser.module.system.entity.SysVehicleType;
import com.snowdreams1006.securityplus.browser.module.system.query.SysVehicleTypeQuery;
import com.snowdreams1006.securityplus.core.tools.JacksonTools;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.time.DateUtils;
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runners.MethodSorters;

import javax.annotation.Resource;
import java.util.Date;
import java.util.List;
import java.util.Objects;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;

/**
 * 系统级别车辆类型表 mapperTest
 *
 * @author snowdreams1006
 * @date 2018-07-10
 */
@Slf4j
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class SysVehicleTypeMapperTest extends BaseTest {

    @Resource
    private SysVehicleTypeMapper sysVehicleTypeMapper;

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

        //初始化影响行数
        int result;

        //默认新增完整字段
        SysVehicleType sysVehicleType = SysVehicleType.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .level(LevelTools.ROOT_LEVEL)
                .num(Integer.MAX_VALUE)
                .type(RoleTypeEnum.USER.getValue())
                .name("新增成功")
                .build();

        sysVehicleType.setRemark("测试普通mapper模式下的insert:完整字段");
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
        result = sysVehicleTypeMapper.insert(sysVehicleType);

        //再次查询最新属性
        sysVehicleType = sysVehicleTypeMapper.selectById(sysVehicleType.getId());

        assertThat(result, greaterThan(0));
        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        assertThat(sysVehicleType.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(sysVehicleType.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(sysVehicleType.getNum(), equalTo(Integer.MAX_VALUE));
        assertThat(sysVehicleType.getType(), equalTo(RoleTypeEnum.USER.getValue()));
        assertThat(sysVehicleType.getName(), equalTo("新增成功"));
        assertThat(sysVehicleType.getRemark(), equalTo("测试普通mapper模式下的insert:完整字段"));
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

        //初始化影响行数
        int result;

        //默认新增必须字段
        SysVehicleType sysVehicleType = SysVehicleType.builder()
                .parentId(Long.valueOf(LevelTools.ROOT_LEVEL))
                .level(LevelTools.ROOT_LEVEL)
                .num(Integer.MAX_VALUE)
                .type(RoleTypeEnum.USER.getValue())
                .name("新增成功")
                .build();

        sysVehicleType.setRemark("测试普通mapper模式下的insert:必须字段");
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
        result = sysVehicleTypeMapper.insert(sysVehicleType);

        //再次查询最新属性
        sysVehicleType = sysVehicleTypeMapper.selectById(sysVehicleType.getId());

        assertThat(result, greaterThan(0));
        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        assertThat(sysVehicleType.getParentId(), equalTo(Long.valueOf(LevelTools.ROOT_LEVEL)));
        assertThat(sysVehicleType.getLevel(), equalTo(LevelTools.ROOT_LEVEL));
        assertThat(sysVehicleType.getNum(), equalTo(Integer.MAX_VALUE));
        assertThat(sysVehicleType.getType(), equalTo(RoleTypeEnum.USER.getValue()));
        assertThat(sysVehicleType.getName(), equalTo("新增成功"));
        assertThat(sysVehicleType.getRemark(), equalTo("测试普通mapper模式下的insert:必须字段"));
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

        //初始化影响行数
        int result;

        //测试数据
        SysVehicleType sysVehicleType = prepareTestData(null);
        sysVehicleType.setNum(1);
        sysVehicleType.setName("更新成功");

        sysVehicleType.setRemark("测试普通mapper模式下的updateById:可选字段");

        //更新
        result = sysVehicleTypeMapper.updateById(sysVehicleType);

        //再次查询最新属性
        sysVehicleType = sysVehicleTypeMapper.selectById(sysVehicleType.getId());

        assertThat(result, greaterThan(0));
        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        assertThat(sysVehicleType.getNum(), equalTo(1));
        assertThat(sysVehicleType.getName(), equalTo("更新成功"));
        assertThat(sysVehicleType.getRemark(), equalTo("测试普通mapper模式下的updateById:可选字段"));
        assertThat(sysVehicleType.getModifiedEnterpriseId(), equalTo(enterpriseId));
        assertThat(sysVehicleType.getModifiedEnterpriseName(), equalTo(enterpriseName));
        assertThat(sysVehicleType.getModifiedUserId(), equalTo(userId));
        assertThat(sysVehicleType.getModifiedUserName(), equalTo(userName));
        assertThat(sysVehicleType.getGmtModified(), greaterThanOrEqualTo(DateUtils.addSeconds(new Date(), -10)));

        log.info("更新成功: {}", JacksonTools.object2Json(sysVehicleType));
    }

    @Test
    public void testCDeleteByIdSuccess() {
        //初始化影响行数
        int result;

        //测试数据
        SysVehicleType sysVehicleType = prepareTestData(null);

        //删除
        result = sysVehicleTypeMapper.deleteById(sysVehicleType.getId());

        //再次查询最新属性
        sysVehicleType = sysVehicleTypeMapper.selectById(sysVehicleType.getId());

        assertThat(result, greaterThan(0));
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
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.selectPage(
                new Page<>(1, 3),
                new EntityWrapper<SysVehicleType>()
                        .eq("type", VehicleTypeEnum.SEMITRAILER.getValue())
        );

        assertThat(sysVehicleTypeList, notNullValue());
        assertThat(sysVehicleTypeList, hasSize(greaterThanOrEqualTo(0)));
        assertThat(sysVehicleTypeList, hasSize(lessThanOrEqualTo(3)));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("type", equalTo(VehicleTypeEnum.SEMITRAILER.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }

        log.info("分页查询成功: {}", JacksonTools.object2Json(sysVehicleTypeList));
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
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.selectList(new EntityWrapper<SysVehicleType>()
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
        sysVehicleType = sysVehicleTypeMapper.selectById(sysVehicleType.getId());

        assertThat(sysVehicleType, notNullValue());
        assertThat(sysVehicleType.getId(), notNullValue());
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleType.getTenantId(), equalTo(tenantId));
            assertThat(sysVehicleType.getTenantType(), equalTo(tenantType));
            assertThat(sysVehicleType.getTenantName(), equalTo(tenantName));
        }

        log.info("查询成功: {}", JacksonTools.object2Json(sysVehicleType));
    }

    @Test
    public void testGListDescendantSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件
        SysVehicleTypeDto parent = SysVehicleTypeDto.createRoot();

        SysVehicleTypeQuery descendantQuery = new SysVehicleTypeQuery();
        descendantQuery.setName("管理员");

        //查询当前车辆类型的后代车辆类型
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listDescendant(parent, descendantQuery);

        assertThat(sysVehicleTypeList, notNullValue());
        assertThat(sysVehicleTypeList, hasSize(greaterThanOrEqualTo(0)));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("level", startsWith(parent.getLevel()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("name", containsString(descendantQuery.getName()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }

        log.info("查询成功: {}", JacksonTools.object2Json(sysVehicleTypeList));
    }

    @Test
    public void testHListChildrenSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件
        SysVehicleTypeDto parent = SysVehicleTypeDto.createRoot();

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setName("管理员");

        //查询当前车辆类型的子车辆类型
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listChildren(parent, childrenQuery);

        assertThat(sysVehicleTypeList, notNullValue());
        assertThat(sysVehicleTypeList, hasSize(greaterThanOrEqualTo(0)));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("level", startsWith(parent.getLevel()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("parentId", equalTo(parent.getId()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("name", containsString(childrenQuery.getName()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }

        log.info("查询成功: {}", JacksonTools.object2Json(sysVehicleTypeList));
    }

    @Test
    public void testICountChildrenSuccess() {
        //初始化影响行数
        int result;

        //查询条件
        SysVehicleTypeDto parent = SysVehicleTypeDto.createRoot();

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setName("管理员");

        //查询当前车辆类型的子车辆类型统计值
        result = sysVehicleTypeMapper.countChildren(parent, childrenQuery);

        assertThat(result, greaterThanOrEqualTo(0));

        log.info("查询成功: {}", JacksonTools.object2Json(result));
    }

    @Test
    public void testJListPageSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件
        Page<SysVehicleTypeDto> page = new Page<>(1, 3);

        SysVehicleTypeDto parent = SysVehicleTypeDto.createRoot();

        SysVehicleTypeQuery childrenQuery = new SysVehicleTypeQuery();
        childrenQuery.setName("管理员");

        //分页查询
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listPage(page, parent, childrenQuery);

        assertThat(sysVehicleTypeList, notNullValue());
        assertThat(sysVehicleTypeList, hasSize(greaterThanOrEqualTo(0)));
        assertThat(sysVehicleTypeList, hasSize(lessThanOrEqualTo(page.getSize())));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("level", startsWith(parent.getLevel()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("parentId", equalTo(parent.getId()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("name", containsString(childrenQuery.getName()))));
        assertThat(sysVehicleTypeList, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(sysVehicleTypeList, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }

        log.info("分页查询成功: {}", JacksonTools.object2Json(sysVehicleTypeList));
    }

    @Test
    public void testKListConditionSuccess() {
        //当前登录用户信息
        SysUserDto currentUser = ShiroTools.getShiroUser();
        Preconditions.checkNotNull(currentUser, "当前登录用户信息不存在");

        Long tenantId = currentUser.getTenantId();
        Integer tenantType = currentUser.getTenantType();
        String tenantName = currentUser.getTenantName();

        log.info("当前登录用户: {}", JacksonTools.object2Json(currentUser));

        //查询条件
        SysVehicleTypeQuery sysVehicleTypeQuery = new SysVehicleTypeQuery();
        sysVehicleTypeQuery.setName("管理员");

        //条件查询
        List<SysVehicleType> results = sysVehicleTypeMapper.listCondition(sysVehicleTypeQuery);

        assertThat(results, notNullValue());
        assertThat(results, hasSize(greaterThanOrEqualTo(0)));
        assertThat(results, everyItem(hasProperty("name", containsString(sysVehicleTypeQuery.getName()))));
        assertThat(results, everyItem(hasProperty("state", not(BaseStateEnum.DELETED.getValue()))));
        if (!ShiroTools.isAdmin()) {
            assertThat(results, everyItem(hasProperty("tenantId", equalTo(tenantId))));
            assertThat(results, everyItem(hasProperty("tenantType", equalTo(tenantType))));
            assertThat(results, everyItem(hasProperty("tenantName", equalTo(tenantName))));
        }

        log.info("条件查询成功: {}", JacksonTools.object2Json(results));
    }

    /**
     * 准备测试数据
     *
     * @param sysVehicleType 自定义的新增车辆类型对象,若 null, 则新增随机车辆类型对象,否则新增指定车辆类型对象
     * @return 新增成功的车辆类型对象
     */
    private SysVehicleType prepareTestData(SysVehicleType sysVehicleType) {
        //初始化影响行数
        int result;

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
                    .type(RoleTypeEnum.USER.getValue())
                    .name("测试数据:" + StringTools.getRandomString(6))
                    .build();

            sysVehicleType.setRemark("测试普通mapper模式下的insert:" + StringTools.getRandomString(6));
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
        result = sysVehicleTypeMapper.insert(sysVehicleType);

        //再次查询最新属性
        sysVehicleType = sysVehicleTypeMapper.selectById(sysVehicleType.getId());

        assertThat(result, greaterThan(0));
        assertThat(sysVehicleType, notNullValue());

        log.info("测试数据: {}", JacksonTools.object2Json(sysVehicleType));

        return sysVehicleType;
    }

}
```

## entityTest

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

## service

基本规则如下:

- `service` 继承统一基类 `IService` ,可根据实际情况决定是否重写某些方法;
- 类名：`I+(大驼峰命名)+Service` 规范,默认注释同数据表注释;
- 方法名: 查询多个结果以 `list` 开头,获取单个对象以 `get`开头,获取统计值以 `count`开头等;


1. 找到 `ISysRoleService` 角色 `service` 接口类,并复制重命名为 `ISysVehicleTypeService`,同时打开两个窗口,方便编辑;
![ISysVehicleTypeService-split-1][ISysVehicleTypeService-split-1]
![ISysVehicleTypeService-split-2][ISysVehicleTypeService-split-2]

2. 根据实际情况,编写 `ISysVehicleTypeService` 文件;
![ISysVehicleTypeService][ISysVehicleTypeService]

ISysVehicleTypeService

```java
package com.snowdreams1006.securityplus.browser.module.system.service;

import com.baomidou.mybatisplus.plugins.Page;
import com.baomidou.mybatisplus.service.IService;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysVehicleTypeDto;
import com.snowdreams1006.securityplus.browser.module.system.entity.SysVehicleType;
import com.snowdreams1006.securityplus.browser.module.system.param.SysVehicleTypeParam;
import com.snowdreams1006.securityplus.browser.module.system.query.SysVehicleTypeQuery;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

/**
 * 系统系别车辆类型表 service
 *
 * @author snowdreams1006
 * @date 2018-05-17
 */
public interface ISysVehicleTypeService extends IService<SysVehicleType> {

    /**
     * 同步查询车辆类型树
     *
     * @param sysVehicleTypeQuery 查询条件
     * @return 车辆类型树
     */
    SysVehicleTypeDto getSyncTree(SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 异步查询车辆类型树
     *
     * @param sysVehicleTypeQuery 查询条件
     * @return 车辆类型列表
     */
    List<SysVehicleTypeDto> listAsyncTree(SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 分页查询车辆类型
     *
     * @param page         分页条件
     * @param sysVehicleTypeQuery 查询条件
     * @return 分页对象
     */
    Page<SysVehicleTypeDto> listPage(Page<SysVehicleTypeDto> page, SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 条件查询车辆类型
     *
     * @param sysVehicleTypeQuery 查询条件
     * @return 车辆类型列表
     */
    List<SysVehicleTypeDto> listCondition(SysVehicleTypeQuery sysVehicleTypeQuery);

    /**
     * 精确查询某个车辆类型
     *
     * @param id 车辆类型id
     * @return 车辆类型对象
     */
    SysVehicleTypeDto getOne(Long id);

    /**
     * 新增某个车辆类型
     *
     * @param sysVehicleTypeParam 新增参数
     * @return 新增车辆类型对象
     */
    @Transactional(rollbackFor = Exception.class)
    SysVehicleTypeDto addOne(SysVehicleTypeParam sysVehicleTypeParam);

    /**
     * 更新某个车辆类型
     *
     * @param sysVehicleTypeParam 更新参数
     * @return 更新车辆类型对象
     */
    @Transactional(rollbackFor = Exception.class)
    SysVehicleTypeDto updateOne(SysVehicleTypeParam sysVehicleTypeParam);

    /**
     * 删除某个车辆类型
     *
     * @param id 车辆类型id
     */
    @Transactional(rollbackFor = Exception.class)
    void deleteOne(Long id);

}
```

## serviceImpl

- `serviceImpl` 继承统一基类 `ServiceImpl` ,可根据实际情况决定是否重写某些方法;
- 类名：`I+(大驼峰命名)+Service` 规范,默认注释同数据表注释,并加上`@Service`等注解说明;
- 方法名: 接口的实现类对外暴露的方法 `public`,否则 `private`,尽量不要 `protected`;


1. 找到 `SysRoleServiceImpl` 角色 `serviceImpl` 实现类,并复制重命名为 `SysVehicleTypeServiceImpl`,同时打开两个窗口,方便编辑;
![sysVehicleTypeServiceImpl-split][sysVehicleTypeServiceImpl-split]

2. 根据实际情况,编写 `SysVehicleTypeServiceImpl` 文件;
![sysVehicleTypeServiceImpl][sysVehicleTypeServiceImpl]

SysVehicleTypeServiceImpl

```
package com.snowdreams1006.securityplus.browser.module.system.service.impl;

import com.baomidou.mybatisplus.mapper.EntityWrapper;
import com.baomidou.mybatisplus.mapper.Wrapper;
import com.baomidou.mybatisplus.plugins.Page;
import com.baomidou.mybatisplus.service.impl.ServiceImpl;
import com.google.common.base.Preconditions;
import com.google.common.collect.ArrayListMultimap;
import com.google.common.collect.Lists;
import com.google.common.collect.Multimap;
import com.google.common.collect.Sets;
import com.snowdreams1006.securityplus.browser.base.tools.LevelTools;
import com.snowdreams1006.securityplus.browser.base.tools.PageTools;
import com.snowdreams1006.securityplus.browser.base.tools.ShiroTools;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysUserDto;
import com.snowdreams1006.securityplus.browser.module.system.dto.SysVehicleTypeDto;
import com.snowdreams1006.securityplus.browser.module.system.entity.SysVehicleType;
import com.snowdreams1006.securityplus.browser.module.system.mapper.SysVehicleTypeMapper;
import com.snowdreams1006.securityplus.browser.module.system.param.SysVehicleTypeParam;
import com.snowdreams1006.securityplus.browser.module.system.query.SysVehicleTypeQuery;
import com.snowdreams1006.securityplus.browser.module.system.service.ISysVehicleTypeService;
import com.snowdreams1006.securityplus.core.tools.ValidatorTools;
import org.apache.commons.collections.CollectionUtils;
import org.apache.commons.lang3.BooleanUtils;
import org.apache.commons.lang3.time.DateUtils;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.*;
import java.util.stream.Collectors;

/**
 * 系统系别车辆类型表 serviceImpl
 *
 * @author snowdreams1006
 * @date 2018-05-17
 */
@Service
public class SysVehicleTypeServiceImpl extends ServiceImpl<SysVehicleTypeMapper, SysVehicleType> implements ISysVehicleTypeService {

    @Resource
    private SysVehicleTypeMapper sysVehicleTypeMapper;

    @Override
    public SysVehicleTypeDto getSyncTree(SysVehicleTypeQuery sysVehicleTypeQuery) {
        //处理公共查询条件
        this.handleCommonQuery(sysVehicleTypeQuery);

        //根据查询条件获取上级车辆类型
        SysVehicleTypeDto parent = this.getParentByQuery(sysVehicleTypeQuery);
        if (Objects.isNull(parent)) {
            return null;
        }

        //上级车辆类型的子车辆类型
        List<SysVehicleTypeDto> children = Lists.newArrayList();

        //查询上级车辆类型的子孙车辆类型(平行列表)
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listDescendant(parent, sysVehicleTypeQuery);
        if (CollectionUtils.isNotEmpty(sysVehicleTypeList)) {
            //entity列表 适配成 dto列表
            List<SysVehicleTypeDto> descendant = this.entityListAdaptDtoList(sysVehicleTypeList, sysVehicleTypeQuery);

            //平行列表 适配成 递归列表
            children = this.parallelList2recursiveList(parent, descendant);
        }

        //建立父子节点关联
        parent.setChildren(children);

        return parent;
    }

    @Override
    public List<SysVehicleTypeDto> listAsyncTree(SysVehicleTypeQuery sysVehicleTypeQuery) {
        //处理公共查询条件
        this.handleCommonQuery(sysVehicleTypeQuery);

        //根据查询条件获取上级车辆类型
        SysVehicleTypeDto parent = this.getParentByQuery(sysVehicleTypeQuery);
        if (Objects.isNull(parent)) {
            return null;
        }

        //上级车辆类型的子车辆类型
        List<SysVehicleTypeDto> children = Lists.newArrayList();

        //查询上级车辆类型的子车辆类型
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listChildren(parent, sysVehicleTypeQuery);
        if (CollectionUtils.isNotEmpty(sysVehicleTypeList)) {
            //entity列表 适配成 dto列表
            children = this.entityListAdaptDtoList(sysVehicleTypeList, sysVehicleTypeQuery);
        }

        return children;
    }

    @Override
    public Page<SysVehicleTypeDto> listPage(Page<SysVehicleTypeDto> page, SysVehicleTypeQuery sysVehicleTypeQuery) {
        //默认分页
        if (Objects.isNull(page)) {
            page = new PageTools<SysVehicleTypeDto>().defaultPage();
        }

        //处理公共查询条件
        this.handleCommonQuery(sysVehicleTypeQuery);

        //根据查询条件获取上级车辆类型
        SysVehicleTypeDto parent = this.getParentByQuery(sysVehicleTypeQuery);
        if (Objects.isNull(parent)) {
            return null;
        }

        //上级车辆类型的子车辆类型
        List<SysVehicleTypeDto> records = Lists.newArrayList();

        //分页查询车辆类型
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listPage(page, parent, sysVehicleTypeQuery);
        if (CollectionUtils.isNotEmpty(sysVehicleTypeList)) {
            //entity列表 适配成 dto列表
            records = this.entityListAdaptDtoList(sysVehicleTypeList, sysVehicleTypeQuery);
        }

        //构造分页信息
        page.setRecords(records);

        return page;
    }

    @Override
    public List<SysVehicleTypeDto> listCondition(SysVehicleTypeQuery sysVehicleTypeQuery) {
        //处理公共查询条件
        this.handleCommonQuery(sysVehicleTypeQuery);

        //查询结果列表
        List<SysVehicleTypeDto> results = Lists.newArrayList();

        //条件查询车辆类型
        List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listCondition(sysVehicleTypeQuery);
        if (CollectionUtils.isNotEmpty(sysVehicleTypeList)) {
            //entity列表 适配成 dto列表
            results = this.entityListAdaptDtoList(sysVehicleTypeList, sysVehicleTypeQuery);
        }

        return results;
    }

    @Override
    public SysVehicleTypeDto getOne(Long id) {
        //基本数据验证
        Preconditions.checkNotNull(id, "待查询的车辆类型id[null]不存在");

        //精确查询
        SysVehicleType sysVehicleType = sysVehicleTypeMapper.selectById(id);

        //数据不存在,返回 null, 具体逻辑由调用者处理
        if (Objects.isNull(sysVehicleType)) {
            return null;
        }

        return this.entityAdaptDto(sysVehicleType);
    }

    @Override
    public SysVehicleTypeDto addOne(SysVehicleTypeParam sysVehicleTypeParam) {
        //基本数据验证
        ValidatorTools.check(sysVehicleTypeParam);

        //调整参数
        adjustParam(sysVehicleTypeParam);

        //唯一性校验
        if (checkExist(sysVehicleTypeParam.getParentId(), sysVehicleTypeParam.getId(), sysVehicleTypeParam.getName())) {
            throw new IllegalArgumentException(String.format("同一层级下的车辆类型[%s]已存在", sysVehicleTypeParam.getName()));
        }

        //param 适配成 entity
        SysVehicleType sysVehicleType = Preconditions.checkNotNull(SysVehicleTypeDto.paramAdaptEntity(sysVehicleTypeParam));
        sysVehicleType.setLevel(LevelTools.calculateLevel(this.getLevel(sysVehicleType.getParentId()), sysVehicleType.getParentId()));

        //新增
        sysVehicleTypeMapper.insert(sysVehicleType);

        return this.getOne(sysVehicleType.getId());
    }

    @Override
    public SysVehicleTypeDto updateOne(SysVehicleTypeParam sysVehicleTypeParam) {
        //基本数据验证
        ValidatorTools.check(sysVehicleTypeParam);
        Preconditions.checkNotNull(sysVehicleTypeParam.getId(), String.format("待更新的车辆类型id[%s]不存在", sysVehicleTypeParam.getName()));

        //业务逻辑验证
        SysVehicleTypeDto beforeDto = Preconditions.checkNotNull(this.getOne(sysVehicleTypeParam.getId()));
        Preconditions.checkNotNull(beforeDto, String.format("待更新的车辆类型[%s]不存在", beforeDto.getName()));

        //dto 适配成 entity
        SysVehicleType before = Preconditions.checkNotNull(SysVehicleTypeDto.dtoAdaptEntity(beforeDto));

        //调整更新参数
        adjustParam(sysVehicleTypeParam);

        //唯一性校验
        if (checkExist(sysVehicleTypeParam.getParentId(), sysVehicleTypeParam.getId(), sysVehicleTypeParam.getName())) {
            throw new IllegalArgumentException(String.format("同一层级下的车辆类型[%s]已存在", sysVehicleTypeParam.getName()));
        }

        //param 适配成 entity
        SysVehicleType after = Preconditions.checkNotNull(SysVehicleTypeDto.paramAdaptEntity(sysVehicleTypeParam));
        after.setLevel(LevelTools.calculateLevel(this.getLevel(after.getParentId()), after.getParentId()));

        //同步自动新增字段
        syncInsertFillFiled(before, after);

        //更新子孙车辆类型
        this.updateDescendant(before, after);

        return this.getOne(after.getId());
    }

    @Override
    public void deleteOne(Long id) {
        //基本数据验证
        Preconditions.checkNotNull(id, "待删除的车辆类型id[null]不存在");

        //业务逻辑验证
        SysVehicleTypeDto sysVehicleTypeDto = this.getOne(id);
        Preconditions.checkNotNull(sysVehicleTypeDto, String.format("待删除的车辆类型id[%s]不存在", id));

        //当前车辆类型存在子车辆类型
        if (sysVehicleTypeMapper.countChildren(sysVehicleTypeDto, null) > 0) {
            throw new IllegalArgumentException(String.format("待删除的车辆类型[%s]存在子车辆类型", sysVehicleTypeDto.getName()));
        }

        sysVehicleTypeMapper.deleteById(id);
    }

    /**
     * 处理公共查询条件
     *
     * @param sysVehicleTypeQuery 车辆类型查询条件
     */
    private void handleCommonQuery(SysVehicleTypeQuery sysVehicleTypeQuery) {
        //修正查询日期: yyyy-MM-dd 解析成 Date 类型是当日零点,因此结束日期手动推迟一天
        if (Objects.nonNull(sysVehicleTypeQuery)) {
            Date startDate = sysVehicleTypeQuery.getStartDate();
            Date endDate = sysVehicleTypeQuery.getEndDate();

            //不存在开始日期,则日期查询无效,若不存在结束日期,则默认当前日期
            if (Objects.nonNull(startDate)) {
                if (Objects.nonNull(endDate)) {
                    endDate = DateUtils.addDays(endDate, 1);
                } else {
                    endDate = new Date();
                }
                sysVehicleTypeQuery.setEndDate(endDate);
            }
        }
    }

    /**
     * 根据车辆类型查询条件获取上级车辆类型
     *
     * @param sysVehicleTypeQuery 车辆类型查询条件
     * @return 上级车辆类型
     */
    private SysVehicleTypeDto getParentByQuery(SysVehicleTypeQuery sysVehicleTypeQuery) {
        //默认上级车辆类型是根节点,level = "0"
        SysVehicleTypeDto parent = SysVehicleTypeDto.createRoot();

        //上级车辆类型是否是普通节点
        if (Objects.nonNull(sysVehicleTypeQuery)) {
            Long id = sysVehicleTypeQuery.getId();
            boolean normalParentFlag = Objects.nonNull(id) && !Objects.equals(Long.valueOf(LevelTools.ROOT_LEVEL), id);
            if (normalParentFlag) {
                parent = this.getOne(id);
            }
        }

        return parent;
    }

    /**
     * 车辆类型entity适配成车辆类型dto
     *
     * @param sysVehicleType 车辆类型entity
     * @return 车辆类型dto
     */
    private SysVehicleTypeDto entityAdaptDto(SysVehicleType sysVehicleType) {
        //参数 null, 则返回 null, 具体逻辑由调用者处理
        if (Objects.isNull(sysVehicleType)) {
            return null;
        }

        //entity静态数据浅拷贝
        SysVehicleTypeDto sysVehicleTypeDto = Preconditions.checkNotNull(SysVehicleTypeDto.entityAdaptDto(sysVehicleType));

        //dto动态数据在线生成
        sysVehicleTypeDto.setIsRoot(false);
        Boolean isParent = sysVehicleTypeMapper.countChildren(sysVehicleTypeDto, null) > 0;
        sysVehicleTypeDto.setIsParent(isParent);

        return sysVehicleTypeDto;
    }

    /**
     * 车辆类型entity列表适配成车辆类型dto列表
     *
     * @param sysVehicleTypeList 车辆类型entity列表
     * @return 车辆类型dto列表
     */
    private List<SysVehicleTypeDto> entityListAdaptDtoList(List<SysVehicleType> sysVehicleTypeList, SysVehicleTypeQuery sysVehicleTypeQuery) {
        //最终输出dto结果列表
        List<SysVehicleTypeDto> sysVehicleTypeDtoList = Lists.newArrayList();

        //集合为空,则返回空集合
        if (CollectionUtils.isEmpty(sysVehicleTypeList)) {
            return sysVehicleTypeDtoList;
        }

        //将系统权车辆类型适配成车辆类型dto
        for (SysVehicleType sysVehicleType : sysVehicleTypeList) {
            //entity静态数据浅拷贝
            SysVehicleTypeDto sysVehicleTypeDto = Preconditions.checkNotNull(SysVehicleTypeDto.entityAdaptDto(sysVehicleType));

            //dto动态数据在线生成
            sysVehicleTypeDto.setIsRoot(false);

            //查询子车辆类型统计值
            Integer allChildCount = sysVehicleTypeMapper.countChildren(sysVehicleTypeDto, sysVehicleTypeQuery);
            sysVehicleTypeDto.setIsParent(allChildCount > 0);

            //entity 转成 dto
            sysVehicleTypeDtoList.add(sysVehicleTypeDto);
        }

        return sysVehicleTypeDtoList;
    }

    /**
     * 根据上级车辆类型及其子孙车辆类型渲染成车辆类型树列表
     *
     * @param parent         上级车辆类型
     * @param sysVehicleTypeDtoList 上级车辆类型的子孙车辆类型(平行结构)
     * @return 车辆类型树列表(递归结构)
     */
    private List<SysVehicleTypeDto> parallelList2recursiveList(SysVehicleTypeDto parent, List<SysVehicleTypeDto> sysVehicleTypeDtoList) {
        //父节点的直接子节点dto列表
        List<SysVehicleTypeDto> children = Lists.newArrayList();

        //逐级遍历,不支持跨级递归
        if (CollectionUtils.isEmpty(sysVehicleTypeDtoList)) {
            return children;
        }

        //全部子节点车辆类型dto对象map
        Multimap<String, SysVehicleTypeDto> allDtoMap = ArrayListMultimap.create();

        //父节点是根节点还是普通节点
        String parentLevel = parent.getLevel();
        if (!parent.getIsRoot()) {
            parentLevel = LevelTools.calculateLevel(parentLevel, parent.getId());
        }

        //将子孙节点组装到指定集合
        for (SysVehicleTypeDto sysVehicleTypeDto : sysVehicleTypeDtoList) {
            //按照 level 层级区分子孙节点
            allDtoMap.put(sysVehicleTypeDto.getLevel(), sysVehicleTypeDto);

            //组装父节点的一级子节点
            if (Objects.equals(parentLevel, sysVehicleTypeDto.getLevel())) {
                children.add(sysVehicleTypeDto);
            }
        }

        //对一级子节点dto列表进行排序
        children.sort(sysVehicleTypeDtoComparator);

        //转变成树形结构
        this.recursive2Tree(children, parentLevel, allDtoMap);

        return children;
    }

    /**
     * 调整参数
     *
     * @param sysVehicleTypeParam 原始参数
     */
    private void adjustParam(SysVehicleTypeParam sysVehicleTypeParam) {
        //新增时,无所有者以及租户信息,则自动设置为当前用户参数(注:注意租户 tenantId 属性)
        boolean autoInsertFlag = Objects.isNull(sysVehicleTypeParam.getId());
        if (autoInsertFlag) {
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

            //所有者信息:可能无"管理员"车辆类型(无权设置),也可能有"管理员"车辆类型(忽略设置)
            if (Objects.isNull(sysVehicleTypeParam.getOwnerEnterpriseId())) {
                sysVehicleTypeParam.setOwnerEnterpriseId(enterpriseId);
            }
            if (Objects.isNull(sysVehicleTypeParam.getOwnerEnterpriseName())) {
                sysVehicleTypeParam.setOwnerEnterpriseName(enterpriseName);
            }
            if (Objects.isNull(sysVehicleTypeParam.getOwnerUserId())) {
                sysVehicleTypeParam.setOwnerUserId(userId);
            }
            if (Objects.isNull(sysVehicleTypeParam.getOwnerUserName())) {
                sysVehicleTypeParam.setOwnerUserName(userName);
            }

            //租户 tenantId:无"管理员"车辆类型(无权设置,不能设置),有"管理员"车辆类型(忽略设置)
            if (ShiroTools.isAdmin() && Objects.isNull(sysVehicleTypeParam.getTenantId())) {
                sysVehicleTypeParam.setTenantId(tenantId);
            }

            //租户信息:可能无"管理员"车辆类型(无权设置),也可能有"管理员"车辆类型(忽略设置)
            if (Objects.isNull(sysVehicleTypeParam.getTenantType())) {
                sysVehicleTypeParam.setTenantType(tenantType);
            }
            if (Objects.isNull(sysVehicleTypeParam.getTenantName())) {
                sysVehicleTypeParam.setTenantName(tenantName);
            }
        }
    }

    /**
     * 检查车辆类型是否已存在
     *
     * @param parentId 上级车辆类型 id
     * @param id       车辆类型 id, 新增时为空, 修改时非空
     * @param name     车辆类型名称
     * @return 车辆类型是否存在
     */
    private boolean checkExist(Long parentId, Long id, String name) {
        //新增/更新判断是否已存在
        Wrapper<SysVehicleType> sysVehicleTypeWrapper = new EntityWrapper<>();
        sysVehicleTypeWrapper.eq("parent_id", parentId)
                .eq("name", name);
        if (Objects.nonNull(id)) {
            sysVehicleTypeWrapper.ne("id", id);
        }

        return sysVehicleTypeMapper.selectCount(sysVehicleTypeWrapper) > 0;
    }

    /**
     * 查询车辆类型 level 值
     *
     * @param id 车辆类型 id,
     * @return 车辆类型level, 存在返回 level,不存在时返回""
     */
    private String getLevel(Long id) {
        SysVehicleType sysVehicleType = sysVehicleTypeMapper.selectById(id);
        if (sysVehicleType == null) {
            return "";
        }
        return sysVehicleType.getLevel();
    }

    /**
     * 同步新增填充字段
     *
     * @param before 同步数据源
     * @param after  待同步数据
     */
    private void syncInsertFillFiled(SysVehicleType before, SysVehicleType after) {
        //获取新增自动填充字段
        Long createEnterpriseId = before.getCreateEnterpriseId();
        String createEnterpriseName = before.getCreateEnterpriseName();
        Long createUserId = before.getCreateUserId();
        String createUserName = before.getCreateUserName();
        Date gmtCreate = before.getGmtCreate();

        //同步更新新增自动填充字段
        after.setCreateEnterpriseId(createEnterpriseId);
        after.setCreateEnterpriseName(createEnterpriseName);
        after.setCreateUserId(createUserId);
        after.setCreateUserName(createUserName);
        after.setGmtCreate(gmtCreate);
    }

    /**
     * 更新子孙车辆类型
     *
     * @param before 更新前车辆类型对象
     * @param after  更新后车辆类型对象
     */
    private void updateDescendant(SysVehicleType before, SysVehicleType after) {
        //更新前后车辆类型level
        String beforeLevel = before.getLevel();
        String afterLevel = after.getLevel();

        //更改父节点,则同步更新子孙节点 level 冗余字段
        if (!Objects.equals(beforeLevel, afterLevel)) {
            //处理当前车辆类型的后代
            SysVehicleTypeDto sysVehicleTypeDto = SysVehicleTypeDto.entityAdaptDto(before);
            List<SysVehicleType> sysVehicleTypeList = sysVehicleTypeMapper.listDescendant(sysVehicleTypeDto, null);

            //更新子代的 level 冗余字段
            if (CollectionUtils.isNotEmpty(sysVehicleTypeList)) {
                for (SysVehicleType sysVehicleType : sysVehicleTypeList) {
                    String level = sysVehicleType.getLevel();

                    //更新子孙车辆类型level
                    if (level.startsWith(beforeLevel)) {
                        //更新根节点的前缀
                        level = afterLevel + level.substring(beforeLevel.length());
                        sysVehicleType.setLevel(level);
                    }

                    //更新子孙车辆类型
                    sysVehicleTypeMapper.updateById(sysVehicleType);
                }
            }
        }

        //更新车辆类型
        sysVehicleTypeMapper.updateById(after);
    }

    /**
     * 根据选中车辆类型列表来处理完整车辆类型列表
     *
     * @param allSysVehicleTypeDtoList      完整车辆类型列表
     * @param selectedSysVehicleTypeDtoList 选中车辆类型列表
     * @return 带有选中状态的完整车辆类型列表
     */
    private List<SysVehicleTypeDto> handleSelectRoleList(List<SysVehicleTypeDto> allSysVehicleTypeDtoList, List<SysVehicleTypeDto> selectedSysVehicleTypeDtoList) {
        //完整车辆类型列表非空校验
        if (CollectionUtils.isEmpty(allSysVehicleTypeDtoList)) {
            return Lists.newArrayList();
        }

        //完整车辆类型列表 list 转 set
        Set<Long> allSysVehicleTypeDtoIdSet = allSysVehicleTypeDtoList.stream().map(SysVehicleTypeDto::getId).collect(Collectors.toSet());

        //选中车辆类型列表 list 转 set
        Set<Long> selectedSysVehicleTypeDtoIdSet = selectedSysVehicleTypeDtoList.stream().map(SysVehicleTypeDto::getId).collect(Collectors.toSet());

        //完整权限与选中权限交集
        Sets.SetView<Long> selected = Sets.intersection(allSysVehicleTypeDtoIdSet, selectedSysVehicleTypeDtoIdSet);

        //处理完整权限列表
        for (SysVehicleTypeDto sysVehicleTypeDto : allSysVehicleTypeDtoList) {
            //选中权限节点
            if (selected.contains(sysVehicleTypeDto.getId())) {
                sysVehicleTypeDto.setChecked(true);
            } else {
                sysVehicleTypeDto.setChecked(false);
            }
            //父节点则展开
            if (BooleanUtils.toBooleanDefaultIfNull(sysVehicleTypeDto.getIsParent(), false)) {
                sysVehicleTypeDto.setOpen(true);
            } else {
                sysVehicleTypeDto.setOpen(false);
            }
        }

        return allSysVehicleTypeDtoList;
    }

    /**
     * 以currentList为基础,递归渲染车辆类型树
     *
     * @param currentList  正在遍历的List
     * @param currentLevel 正在遍历的level
     * @param allDtoMap    全部节点map
     */
    private void recursive2Tree(List<SysVehicleTypeDto> currentList, String currentLevel, Multimap<String, SysVehicleTypeDto> allDtoMap) {
        //遍历当前节点并和子孙节点建立关系
        for (SysVehicleTypeDto sysVehicleTypeDto : currentList) {
            //下次遍历的层级level
            String childrenLevel = LevelTools.calculateLevel(currentLevel, sysVehicleTypeDto.getId());

            //下次遍历的dtoList
            List<SysVehicleTypeDto> children = (List<SysVehicleTypeDto>) allDtoMap.get(childrenLevel);

            //不支持跨级递归
            if (CollectionUtils.isNotEmpty(children)) {
                //对下次遍历的list排序
                children.sort(sysVehicleTypeDtoComparator);

                //将下次遍历的list作为当前节点的children
                sysVehicleTypeDto.setChildren(children);

                //继续迭代
                recursive2Tree(children, childrenLevel, allDtoMap);
            }
        }
    }

    /**
     * 车辆类型dto 比较器
     */
    private Comparator<SysVehicleTypeDto> sysVehicleTypeDtoComparator = Comparator.comparingInt(SysVehicleType::getNum);

}
```


[sys_role_structure]: ../../static/image/sys_role_structure.png "sys_role_structure"
[sys_role_copy]: ../../static/image/sys_role_copy.png "sys_role_copy"
[sys_vehicle_type_paste]: ../../static/image/sys_vehicle_type_paste.png "sys_vehicle_type_paste"
[sys_vehicle_type_unmodified]: ../../static/image/sys_vehicle_type_unmodified.png "sys_vehicle_type_unmodified"
[sys_vehicle_type_modified]: ../../static/image/sys_vehicle_type_modified.png "sys_vehicle_type_modified"
[svn-update]: ../../static/image/svn-update.png "svn-update"
[svn-updating]: ../../static/image/svn-updating.png "svn-updating"
[sysVehicleType-unmodified]: ../../static/image/sysVehicleType-unmodified.png "sysVehicleType-unmodified"
[svn-add-confirm]: ../../static/image/svn-add-confirm.png "svn-add-confirm"
[split-vertically-prepare]: ../../static/image/split-vertically-prepare.png "split-vertically-prepare"
[split-vertically-complete]: ../../static/image/split-vertically-complete.png "split-vertically-complete"
[baseEntity]: ../../static/image/baseEntity.png "baseEntity"
[sysVehicleType]: ../../static/image/sysVehicleType.png "sysVehicleType"
[sysVehicleTypeParam-split]: ../../static/image/sysVehicleTypeParam-split.png "sysVehicleTypeParam-split"
[sysVehicleTypeParam]: ../../static/image/sysVehicleTypeParam.png "sysVehicleTypeParam"
[sysVehicleTypeQuery-split]: ../../static/image/sysVehicleTypeQuery-split.png "sysVehicleTypeQuery-split"
[sysVehicleTypeQuery]: ../../static/image/sysVehicleTypeQuery.png "sysVehicleTypeQuery"
[sysVehicleTypeDto-split]: ../../static/image/sysVehicleTypeDto-split.png "sysVehicleTypeDto-split"
[sysVehicleTypeDto]: ../../static/image/sysVehicleTypeDto.png "sysVehicleTypeDto"
[sysVehicleTypeMapper-split-java]: ../../static/image/sysVehicleTypeMapper-split-java.png "sysVehicleTypeMapper-split-java"
[sysVehicleTypeMapper-java]: ../../static/image/sysVehicleTypeMapper-java.png "sysVehicleTypeMapper-java"
[sysVehicleTypeMapper-split-xml-1]: ../../static/image/sysVehicleTypeMapper-split-xml-1.png "sysVehicleTypeMapper-split-xml-1"
[sysVehicleTypeMapper-split-xml-2]: ../../static/image/sysVehicleTypeMapper-split-xml-2.png "sysVehicleTypeMapper-split-xml-2"
[sysVehicleTypeMapper-xml]: ../../static/image/sysVehicleTypeMapper-xml.png "sysVehicleTypeMapper-xml"
[sysVehicleTypeMapperTest-split-1]: ../../static/image/sysVehicleTypeMapperTest-split-1.png "sysVehicleTypeMapperTest-split-1"
[sysVehicleTypeMapperTest-split-2]: ../../static/image/sysVehicleTypeMapperTest-split-2.png "sysVehicleTypeMapperTest-split-2"
[sysVehicleTypeMapperTest]: ../../static/image/sysVehicleTypeMapperTest.png "sysVehicleTypeMapperTest"
[sysVehicleTypeTest-split]: ../../static/image/sysVehicleTypeTest-split.png "sysVehicleTypeTest-split"
[sysVehicleTypeTest]: ../../static/image/sysVehicleTypeTest.png "sysVehicleTypeTest"
[ISysVehicleTypeService-split-1]: ../../static/image/ISysVehicleTypeService-split-1.png "ISysVehicleTypeService-split-1"
[ISysVehicleTypeService-split-2]: ../../static/image/ISysVehicleTypeService-split-2.png "ISysVehicleTypeService-split-2"
[ISysVehicleTypeService]: ../../static/image/ISysVehicleTypeService.png "ISysVehicleTypeService"
[sysVehicleTypeServiceImpl-split]: ../../static/image/sysVehicleTypeServiceImpl-split.png "sysVehicleTypeServiceImpl-split"
[sysVehicleTypeServiceImpl]: ../../static/image/sysVehicleTypeServiceImpl.png "sysVehicleTypeServiceImpl"