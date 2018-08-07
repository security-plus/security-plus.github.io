# mapper

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

[sysVehicleTypeMapper-split-java]: ../../../static/image/sysVehicleTypeMapper-split-java.png "sysVehicleTypeMapper-split-java"
[sysVehicleTypeMapper-java]: ../../../static/image/sysVehicleTypeMapper-java.png "sysVehicleTypeMapper-java"