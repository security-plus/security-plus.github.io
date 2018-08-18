# 开发规范


!> 假设我们已存在一张 User 表，且已有对应的实体类 User，实现 User 表的 CRUD 操作我们需要做什么呢？

?> AR 模式提供了一种更加便捷的方式实现 CRUD 操作，其本质还是调用的 Mybatis 对应的方法，类似于语法糖。


1.创建企业是默认所有者-用户


2.已创建的企业负责人所有者信息不对

```yaml
mybatis-plus:
    # 如果是放在src/main/java目录下 classpath:/com/yourpackage/*/mapper/*Mapper.xml
    # 如果是放在resource目录 classpath:/mapper/*Mapper.xml
    mapper-locations: classpath:/mapper/*Mapper.xml
    #实体扫描，多个package用逗号或者分号分隔
    typeAliasesPackage: com.yourpackage.*.entity
    global-config:
      #主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
      id-type: 3
      #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
      field-strategy: 2
      #驼峰下划线转换
      db-column-underline: true
      #mp2.3+ 全局表前缀 mp_
      #table-prefix: mp_
      #刷新mapper 调试神器
      #refresh-mapper: true
      #数据库大写下划线转换
      #capital-mode: true
      # Sequence序列接口实现类配置
      key-generator: com.baomidou.mybatisplus.incrementer.OracleKeyGenerator
      #逻辑删除配置（下面3个配置）
      logic-delete-value: 1
      logic-not-delete-value: 0
      sql-injector: com.baomidou.mybatisplus.mapper.LogicSqlInjector
      #自定义填充策略接口实现
      meta-object-handler: com.baomidou.springboot.MyMetaObjectHandler
    configuration:
      #配置返回数据库(column下划线命名&&返回java实体是驼峰命名)，自动匹配无需as（没开启这个，SQL需要写as： select user_id as userId） 
      map-underscore-to-camel-case: true
      cache-enabled: false
      #配置JdbcTypeForNull, oracle数据库必须配置
      jdbc-type-for-null: 'null' 
```
值                | 描述
---------------- | ---------------------
IdType.AUTO      | 数据库ID自增
IdType.INPUT     | 用户输入ID
IdType.ID_WORKER | 全局唯一ID，内容为空自动填充（默认配置）
IdType.UUID      | 全局唯一ID，内容为空自动填充



企业信息:

"企业负责人"角色默认为企业紧急联系人,有且只有一个企业负责人,账号是紧急联系人手机号;

角色信息:

"管理员"角色规则:以"admin_"开头,例如超级平台方: admin_super_platform

开发者平台方: admin_platform_developer

接口管理员: admin_platform_developer_api

开发管理员: admin_platform_developer_develop

基础管理员: admin_platform_developer_base

系统管理员: admin_platform_developer_system

系统基础管理员: admin_platform_developer_system_base

系统高级管理员: admin_platform_developer_system_advance

浙江省平台方: admin_platform_zhejiang

宁波市平台方: admin_platform_zhejiang_ningbo

宁波市危化品平台: admin_platform_zhejiang_ningbo_whp

宁波市危化品承运方: zhejiang_ningbo_whp_carrier

宁波市危化品货主: zhejiang_ningbo_whp_consignor

...

宁波市集卡平台: admin_platform_zhejiang_ningbo_jk

宁波市货代管理员: zhejiang_ningbo_jk_hd

宁波市货代操作: zhejiang_ningbo_jk_forwarder_operator

宁波市货代调度: zhejiang_ningbo_jk_forwarder_scheduling

宁波市车队管理员: zhejiang_ningbo_jk_motorcade

宁波市车队司机: zhejiang_ningbo_jk_motorcade_driver

...

企业内部角色不受此规则显示,由企业内部员工自定义,每个企业只有唯一一个负责人,拥有企业最高权限;




