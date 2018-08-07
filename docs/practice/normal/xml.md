# xml

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

[sysVehicleTypeMapper-split-xml-1]: ../../../static/image/sysVehicleTypeMapper-split-xml-1.png "sysVehicleTypeMapper-split-xml-1"
[sysVehicleTypeMapper-split-xml-2]: ../../../static/image/sysVehicleTypeMapper-split-xml-2.png "sysVehicleTypeMapper-split-xml-2"
[sysVehicleTypeMapper-xml]: ../../../static/image/sysVehicleTypeMapper-xml.png "sysVehicleTypeMapper-xml"