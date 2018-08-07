# query

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

[sysVehicleTypeQuery-split]: ../../../static/image/sysVehicleTypeQuery-split.png "sysVehicleTypeQuery-split.png"
[sysVehicleTypeQuery]: ../../../static/image/sysVehicleTypeQuery.png "sysVehicleTypeQuery"

