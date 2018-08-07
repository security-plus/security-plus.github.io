# entity

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

[svn-update]: ../../../static/image/svn-update.png "svn-update"
[svn-updating]: ../../../static/image/svn-updating.png "svn-updating"
[sysVehicleType-unmodified]: ../../../static/image/sysVehicleType-unmodified.png "sysVehicleType-unmodified"
[svn-add-confirm]: ../../../static/image/svn-add-confirm.png "svn-add-confirm"
[split-vertically-prepare]: ../../../static/image/split-vertically-prepare.png "split-vertically-prepare"
[split-vertically-complete]: ../../../static/image/split-vertically-complete.png "split-vertically-complete"
[baseEntity]: ../../../static/image/baseEntity.png "baseEntity"
[sysVehicleType]: ../../../static/image/sysVehicleType.png "sysVehicleType"