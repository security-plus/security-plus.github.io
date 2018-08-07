# serviceImpl

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

[sysVehicleTypeServiceImpl-split]: ../../../static/image/sysVehicleTypeServiceImpl-split.png "sysVehicleTypeServiceImpl-split"
[sysVehicleTypeServiceImpl]: ../../../static/image/sysVehicleTypeServiceImpl.png "sysVehicleTypeServiceImpl"