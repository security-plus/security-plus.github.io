# param

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

[sysVehicleTypeParam-split]: ../../../static/image/sysVehicleTypeParam-split.png "sysVehicleTypeParam-split"
[sysVehicleTypeParam]: ../../../static/image/sysVehicleTypeParam.png "sysVehicleTypeParam"
