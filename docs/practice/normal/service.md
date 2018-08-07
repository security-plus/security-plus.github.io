# service

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

[ISysVehicleTypeService-split-1]: ../../../static/image/ISysVehicleTypeService-split-1.png "ISysVehicleTypeService-split-1"
[ISysVehicleTypeService-split-2]: ../../../static/image/ISysVehicleTypeService-split-2.png "ISysVehicleTypeService-split-2"
[ISysVehicleTypeService]: ../../../static/image/ISysVehicleTypeService.png "ISysVehicleTypeService"