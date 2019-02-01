# 简介 | Introduce

`security-plus`（简称SP）框架是专门为`java`后台管理系统而诞生的完整系统,初见倾心,再见动情!

> **[success] 目标和愿景 ** 
> 
> 安全便捷,开箱即用,灵活方便可扩展.

# 优点 | Advantages
 
- **支持多企业管理**：不仅仅适用某一家企业内部管理,还可以拓展成多企业模式,轻松发展成XX平台;
- **支持无限制代理**：功能权限可自由分配,支持病毒式发展下级平台;
- **完整的权限体系**：基于 `rbac` 模式实现功能权限,基于 `多租户模式` 实现数据权限,灵活有效地控制权限;
- **递归分配权限**：天然支持递归权限树,功能权限和数据权限双重保证系统隐私安全;
- **强大代码生成器**：一键生成后端和前端全部代码,甚至于测试文件,此外也支持自定义模板逻辑;
- **清晰的目录结构**：基于 `maven` 多模块项目,构建标准的 `spring-boot` 目录结构,有利于降低学习成本;
- **良好的开发规范**：遵循[阿里巴巴java开发手册][Alibaba-Java-Coding-Guidelines]规范,完整的开发测试,提供高质量的软件产品服务;
- **友好的代码注释**: 细致但不繁琐的注释让您阅读源码以及二次开发不再吃力,开发何苦为难开发;
- **druid数据库监控**: 随时随地查看数据库连接信息,方便 `dba` 监控并调优数据库 `sql`;
- **swagger接口文档**: 自动生成 api 接口文档,方便调试开发,降低对接工作时的沟通成本;

# 快速预览 | Preview
![preview][preview]

# 技术选型 | Technology

- 基础框架: [spring-boot][spring-boot]
- ORM框架: [Mybatis-Plus][Mybatis-Plus] Born To Simplify Development
- 安全框架: [Apache Shiro™][Apache Shiro™] Simple. Java. Security.
- 模板框架: [Beetl][Beetl] 新一代Java模板引擎典范
- 前端框架: [hplus] 后台主题UI框架

# 项目结构 | Architecture

```
security-plus 雪之梦后台权限管理系统

├── security-plus-parent    父项目: 负责维护父子项目关系,本身无任何代码
├── security-plus-core      核心项目: 子项目,是基础模块,集成常用工具类
└── security-plus-browser   浏览器项目: 子项目,是后端管理系统项目,依赖于核心项目
    ├── log  
    ├── src/ 
    |   ├── main/ 
    |   |   ├── java/
    |   |   |   └── com.snowdreams1006.securityplus.browser/
    |   |   |       ├── base/              公共配置类以及常用工具类等
    |   |   |       ├── module/            具体业务模块
    |   |   |       └── SecurityPlusBrowserApplication  程序入口启动类
    |   |   └── resources/
    |   |       ├── sql                     代码生成器产出 sql 以及程序初始化 sql 
    |   |       ├── static/                 静态资源目录
    |   |       ├── templates/              模板页面
    |   |       ├── application.yml         总配置文件
    |   |       ├── application-dev.yml     开发环境配置文件
    |   |       ├── application-prod.yml    生产环境配置文件
    |   |       ├── application-test.yml    测试环境配置文件
    |   |       └── banner.txt              程序启动 banner
    |   └── test/ 
    |       └── java/
    |           └── com.snowdreams1006.securityplus.browser/
    |               ├── base/
    |               └── module/
    ├── target/ 
    └── pom.xml 

```

# 系统环境 | Environment

- [maven 3.5.3+](http://maven.apache.org/download.cgi "apache-maven")
- [jdk 1.8.0_161+](http://www.oracle.com/technetwork/java/javase/downloads/index.html "jdk8+")
- [MySQL 5.7+](https://www.mysql.com/downloads/ "mysql5.7+")

# 下载地址 | Download

- [最物流][zui56]企业用户[点此访问][zui56-server] `svn` 服务器
- 其他用户[**后续访问**][security-plus-demo] `github` 服务器

>  `SP` 框架暂未开放下载,如有相关需求,欢迎随时[联系我][qq-513238368]!

# 本地部署 | Documentation

1. 下载 `security-plus` 源代码,并导入到 `Eclipse 或者 IDEA` 等 ide 开发工具;
2. 创建 `security-plus` 数据库,执行 `mysql.sql` 文件,初始化基本数据;
3. 修改 `application-*.yml` 配置文件，更新 `MySQL`连接信息以及其他配置;
4. 在 `security-plus-parent` 项目下，执行 `mvn clean install` 下载安装相关依赖;
5. 在 `security-plus-browser` 项目下,右键 `SecurityPlusBrowserApplication` 启动;
6. `security-plus-browser` 项目的默认访问路径：[http://localhost:8080/index.page](http://localhost:8080/index.page "index.page")

> `SP` 默认账号密码: admin/123456

# 文档 | Documentation

- [security-plus.pdf](./security-plus.pdf 'security-plus.pdf') `pdf` 文件
- [security-plus.mobi](./security-plus.mobi 'security-plus.mobi') `mobi` 文件
- [security-plus.epub](./security-plus.epub 'security-plus.epub') `ePub` 文件

> 如需获取完整版文档,欢迎随时[联系我][qq-513238368]!

# 版权 | License

`SP` 使用[Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0 "LICENSE-2.0")开源协议,请放心使用,如有顾虑[点击查看](./LICENSE)协议详情

# 交流反馈 | Feedback

- github 主页: [snowdreams1006][github-snowdreams1006]
- 交流提问区: [security-plus-issues][security-plus-issues]
- 我的QQ号: [513238368][qq-513238368]

# 捐赠 | Donate

![wechat-alipay][wechat-alipay]

> 您的支持是我创作的最大动力,感谢您的肯定!

# 致谢 | Thanks

- [Java实现权限管理（上）](https://www.imooc.com/view/584 "Java实现权限管理（上）")
- [Java实现权限管理（下）](https://www.imooc.com/learn/629 "Java实现权限管理（下）")
- [RBAC打造通用web管理权限](https://www.imooc.com/learn/799 "RBAC打造通用web管理权限")
- [Linux权限管理之基本权限](https://www.imooc.com/learn/481 "Linux权限管理之基本权限")
- [Linux权限管理之特殊权限](https://www.imooc.com/learn/481 "Linux权限管理之特殊权限")
- [Java开发企业级权限管理系统](https://coding.imooc.com/class/149.html "Java开发企业级权限管理系统")
- [Spring Security开发安全的REST服务](https://coding.imooc.com/class/134.html "Spring Security开发安全的REST服务")
- [2小时学会Spring Boot](https://www.imooc.com/learn/767 "2小时学会Spring Boot")
- [Spring Boot进阶之Web进阶](https://www.imooc.com/learn/810 "Spring Boot进阶之Web进阶")
- [SpringBoot开发常用技术整合](https://www.imooc.com/learn/956 "SpringBoot开发常用技术整合")
- [guns 后台管理系统](https://github.com/stylefeng/Guns "Guns") 
- [dp-BOOT](https://github.com/HeroBarry/dp-BOOT "dp-BOOT")
- [人人开源](https://github.com/renrenio/renren-security "renren-security") 

# 参与贡献 | Distribute

- 维护文档: [security-plus.github.io][security-plus.github.io],欢迎参与翻译和修订;
- 开发文档: [security-plus][security-plus],欢迎志同道合小伙伴加入;

> 欢迎各路好汉一起来参与完善 `security-plus`，我们期待你的PR！

[security-plus]: https://github.com/security-plus "security-plus"
[security-plus-issues]: https://github.com/security-plus/security-plus.github.io/issues "security-plus-issues"
[security-plus.github.io]: https://github.com/security-plus/security-plus.github.io "security-plus.github.io"
[github-snowdreams1006]: https://github.com/snowdreams1006 "github-snowdreams1006"
[spring-boot]: http://spring.io/projects/spring-boot "spring-boot"
[Mybatis-Plus]: http://mp.baomidou.com "Mybatis-Plus"
[Apache Shiro™]: http://shiro.apache.org/ "Apache Shiro™"
[Beetl]: http://ibeetl.com/ "Beetl"
[hplus]: http://www.zi-han.net/theme/hplus/ "hplus"
[Alibaba-Java-Coding-Guidelines]: https://github.com/alibaba/Alibaba-Java-Coding-Guidelines "Alibaba-Java-Coding-Guidelines" 
[zui56-server]: https://120.26.100.4:8443/svn/zui56-server "zui56-server"
[security-plus-demo]: https://github.com/security-plus/security-plus-demo "security-plus-demo"
[zui56]: https://www.zui56.net "zui56"
[qq-513238368]: http://wpa.qq.com/msgrd?v=3&uin=513238368&site=qq&menu=yes "qq-513238368"
[preview]: ./static/image/preview.png "preview"
[wechat]: ./static/image/wechat.jpg "wechat"
[alipay]: ./static/image/alipay.jpg "alipay"
[wechat-alipay]: ./static/image/wechat-alipay.jpg "wechat-alipay"




