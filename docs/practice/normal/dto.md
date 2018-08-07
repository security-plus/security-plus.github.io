# dto

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

[sysVehicleTypeDto-split]: ../../../static/image/sysVehicleTypeDto-split.png "sysVehicleTypeDto-split"
[sysVehicleTypeDto]: ../../../static/image/sysVehicleTypeDto.png "sysVehicleTypeDto"