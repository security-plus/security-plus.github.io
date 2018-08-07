# 常规开发

开发流程: 后端代码 -> 前端页面 -> 分配权限 -> 开发完成

下面以 `车辆类型` 为例,说明如何进行 `增删改查` 简单业务,效果如图

![最终效果预览][normal-preview]

### 建立数据表

![数据表结构][normal-mysql-1]

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


[normal-mysql-1]: ../../static/image/normal-mysql-1.png "normal-mysql-1"
[normal-mysql-2]: ../../static/image/normal-mysql-2.png "normal-mysql-2"
[normal-mysql-3]: ../../static/image/normal-mysql-3.png "normal-mysql-3"


[normal-preview]: ../../static/image/normal-preview.png "normal-preview"

