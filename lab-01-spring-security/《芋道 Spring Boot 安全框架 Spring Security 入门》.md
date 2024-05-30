芋道源码 —— 纯源码解析博客
愿半生编码，如一生老友！
文章
知识星球
Github
微信公众号
工作内推
友链
大厂面试必备
Java 超神之路
芋道 Spring Boot 安全框架 Spring Security 入门
总阅读量:次
⭐⭐⭐ Spring Boot 项目实战	⭐⭐⭐ Spring Cloud 项目实战
《Dubbo 实现原理与源码解析 —— 精品合集》	《Netty 实现原理与源码解析 —— 精品合集》
《Spring 实现原理与源码解析 —— 精品合集》	《MyBatis 实现原理与源码解析 —— 精品合集》
《Spring MVC 实现原理与源码解析 —— 精品合集》	《数据库实体设计合集》
《Spring Boot 实现原理与源码解析 —— 精品合集》	《Java 面试题 + Java 学习指南》
摘要: 原创出处 http://www.iocoder.cn/Spring-Boot/Spring-Security/ 「芋道源码」欢迎转载，保留摘要，谢谢！

1. 概述
2. 快速入门
3. 进阶使用
4. 整合 Spring Session
5. 整合 OAuth2
6. 整合 JWT
7. 项目实战
666. 彩蛋
     本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs 的 对应 lab-01-spring-security 目录。

原创不易，给点个 Star 嘿，一起冲鸭！

1. 概述
   基本上，在所有的开发的系统中，都必须做认证(authentication)和授权(authorization)，以保证系统的安全性。😈 考虑到很多胖友对认证和授权有点分不清楚，艿艿这里引用一个网上有趣的例子：

FROM 《认证 (authentication) 和授权 (authorization) 的区别》

authentication [ɔ,θɛntɪ'keʃən] 认证
authorization [,ɔθərɪ'zeʃən] 授权
以打飞机举例子：

【认证】你要登机，你需要出示你的 passport 和 ticket，passport 是为了证明你张三确实是你张三，这就是 authentication。
【授权】而机票是为了证明你张三确实买了票可以上飞机，这就是 authorization。
以论坛举例子：

【认证】你要登录论坛，输入用户名张三，密码 1234，密码正确，证明你张三确实是张三，这就是 authentication。
【授权】再一 check 用户张三是个版主，所以有权限加精删别人帖，这就是 authorization 。
所以简单来说：认证解决“你是谁”的问题，授权解决“你能做什么”的问题。另外，在推荐阅读下《认证、授权、鉴权和权限控制》 文章，更加详细明确。

在 Java 生态中，目前有 Spring Security 和 Apache Shiro 两个安全框架，可以完成认证和授权的功能。本文，我们先来学习下 Spring Security 。其官方对自己介绍如下：

FROM 《Spring Security 官网》

Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
Spring Security 是一个功能强大且高度可定制的身份验证和访问控制框架。它是用于保护基于 Spring 的应用程序。

Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements
Spring Security 是一个框架，侧重于为 Java 应用程序提供身份验证和授权。与所有 Spring 项目一样，Spring 安全性的真正强大之处，在于它很容易扩展以满足定制需求。

2. 快速入门
   示例代码对应仓库：lab-01-springsecurity-demo 。

在本小节中，我们来快速入门下 Spring Security ，实现访问 API 接口时，需要首先进行登录，才能进行访问。

2.1 引入依赖
在 pom.xml 文件中，引入相关依赖。

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>2.1.10.RELEASE</version>
<relativePath/> <!-- lookup parent from repository -->
</parent>
<modelVersion>4.0.0</modelVersion>

    <artifactId>lab-01-springsecurity-demo</artifactId>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 实现对 Spring Security 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    </dependencies>

</project>
具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。

2.2 Application
创建 Application.java 类，配置 @SpringBootApplication 注解即可。代码如下：

// Application.java

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
2.3 配置文件
在 application.yml 中，添加 Spring Security 配置，如下：

spring:
# Spring Security 配置项，对应 SecurityProperties 配置类
security:
# 配置默认的 InMemoryUserDetailsManager 的用户账号与密码。
user:
name: user # 账号
password: user # 密码
roles: ADMIN # 拥有角色
在 spring.security 配置项，设置 Spring Security 的配置，对应 SecurityProperties 配置类。
默认情况下，Spring Boot UserDetailsServiceAutoConfiguration 自动化配置类，会创建一个内存级别的 InMemoryUserDetailsManager Bean 对象，提供认证的用户信息。
这里，我们添加了 spring.security.user 配置项，UserDetailsServiceAutoConfiguration 会基于配置的信息创建一个用户 User 在内存中。
如果，我们未添加 spring.security.user 配置项，UserDetailsServiceAutoConfiguration 会自动创建一个用户名为 "user" ，密码为 UUID 随机的用户 User 在内存中。
2.4 AdminController
在 cn.iocoder.springboot.lab01.springsecurity.controller 包路径下，创建 AdminController 类，提供管理员 API 接口。代码如下：

// AdminController.java

@RestController
@RequestMapping("/admin")
public class AdminController {

    @GetMapping("/demo")
    public String demo() {
        return "示例返回";
    }

}
这里，我们先提供一个 "/admin/demo" 接口，用于测试未登录时，会被拦截到登录界面。
2.5 简单测试
执行 Application#main(String[] args) 方法，运行项目。

项目启动成功后，浏览器访问 http://127.0.0.1:8080/admin/demo 接口。因为未登录，所以被 Spring Security 拦截到登录界面。如下图所示：默认登录界面

因为我们没有自定义登录界面，所以默认会使用 DefaultLoginPageGeneratingFilter 类，生成上述界面。

输入我们在「2.3 配置文件」中配置的「user/user」账号，进行登录。登录完成后，因为 Spring Security 会记录被拦截的访问地址，所以浏览器自动动跳转 http://127.0.0.1:8080/admin/demo 接口。访问结果如下图所示： 接口的结果

3. 进阶使用
   示例代码对应仓库：lab-01-springsecurity-demo-role 。

在「2. 快速入门」中，我们很快速的完成了 Spring Security 的入门。本小节，我们将会自定义 Spring Security 的配置，实现权限控制。

考虑到不污染上述的示例，我们新建一个 lab-01-springsecurity-demo-role 项目。

3.1 引入依赖
和 「2.1 引入依赖」 一致，见 pom.xml 文件。

3.2 示例一
在示例一中，我们会看看如何自定义 Spring Security 的配置，实现权限控制。

3.2.1 SecurityConfig
在 cn.iocoder.springboot.lab01.springsecurity.config 包下，创建 SecurityConfig 配置类，继承 WebSecurityConfigurerAdapter 抽象类，实现 Spring Security 在 Web 场景下的自定义配置。代码如下：

// SecurityConfig.java

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // ...

}
我们可以通过重写 WebSecurityConfigurerAdapter 的方法，实现自定义的 Spring Security 的配置。
首先，我们重写 #configure(AuthenticationManagerBuilder auth) 方法，实现 AuthenticationManager 认证管理器。代码如下：

// SecurityConfig.java

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
auth.
// <X> 使用内存中的 InMemoryUserDetailsManager
inMemoryAuthentication()
// <Y> 不使用 PasswordEncoder 密码编码器
.passwordEncoder(NoOpPasswordEncoder.getInstance())
// <Z> 配置 admin 用户
.withUser("admin").password("admin").roles("ADMIN")
// <Z> 配置 normal 用户
.and().withUser("normal").password("normal").roles("NORMAL");
}
<X> 处，调用 AuthenticationManagerBuilder#inMemoryAuthentication() 方法，使用内存级别的 InMemoryUserDetailsManager Bean 对象，提供认证的用户信息。
Spring 内置了两种 UserDetailsManager 实现：
InMemoryUserDetailsManager，和「2. 快速入门」是一样的。
JdbcUserDetailsManager ，基于 JDBC的 JdbcUserDetailsManager 。
实际项目中，我们更多采用调用 AuthenticationManagerBuilder#userDetailsService(userDetailsService) 方法，使用自定义实现的 UserDetailsService 实现类，更加灵活且自由的实现认证的用户信息的读取。
<Y> 处，调用 AbstractDaoAuthenticationConfigurer#passwordEncoder(passwordEncoder) 方法，设置 PasswordEncoder 密码编码器。
在这里，为了方便，我们使用 NoOpPasswordEncoder 。实际上，等于不使用 PasswordEncoder ，不配置的话会报错。
生产环境下，推荐使用 BCryptPasswordEncoder 。更多关于 PasswordEncoder 的内容，推荐阅读《该如何设计你的 PasswordEncoder?》文章。
<Z> 处，配置了「admin/admin」和「normal/normal」两个用户，分别对应 ADMIN 和 NORMAL 角色。相比「2. 快速入门」来说，可以配置更多的用户。
然后，我们重写 #configure(HttpSecurity http) 方法，主要配置 URL 的权限控制。代码如下：

// SecurityConfig.java

@Override
protected void configure(HttpSecurity http) throws Exception {
http
// <X> 配置请求地址的权限
.authorizeRequests()
.antMatchers("/test/echo").permitAll() // 所有用户可访问
.antMatchers("/test/admin").hasRole("ADMIN") // 需要 ADMIN 角色
.antMatchers("/test/normal").access("hasRole('ROLE_NORMAL')") // 需要 NORMAL 角色。
// 任何请求，访问的用户都需要经过认证
.anyRequest().authenticated()
.and()
// <Y> 设置 Form 表单登录
.formLogin()
//                    .loginPage("/login") // 登录 URL 地址
.permitAll() // 所有用户可访问
.and()
// 配置退出相关
.logout()
//                    .logoutUrl("/logout") // 退出 URL 地址
.permitAll(); // 所有用户可访问
}
<X> 处，调用 HttpSecurity#authorizeRequests() 方法，开始配置 URL 的权限控制。注意看艿艿配置的四个权限控制的配置。下面，是配置权限控制会使用到的方法：

#(String... antPatterns) 方法，配置匹配的 URL 地址，基于 Ant 风格路径表达式 ，可传入多个。
【常用】#permitAll() 方法，所有用户可访问。
【常用】#denyAll() 方法，所有用户不可访问。
【常用】#authenticated() 方法，登录用户可访问。
#anonymous() 方法，无需登录，即匿名用户可访问。
#rememberMe() 方法，通过 remember me 登录的用户可访问。
#fullyAuthenticated() 方法，非 remember me 登录的用户可访问。
#hasIpAddress(String ipaddressExpression) 方法，来自指定 IP 表达式的用户可访问。
【常用】#hasRole(String role) 方法， 拥有指定角色的用户可访问。
【常用】#hasAnyRole(String... roles) 方法，拥有指定任一角色的用户可访问。
【常用】#hasAuthority(String authority) 方法，拥有指定权限(authority)的用户可访问。
【常用】#hasAuthority(String... authorities) 方法，拥有指定任一权限(authority)的用户可访问。
【最牛】#access(String attribute) 方法，当 Spring EL 表达式的执行结果为 true 时，可以访问。
<Y> 处，调用 HttpSecurity#formLogin() 方法，设置 Form 表单登录。

如果胖友想要自定义登录页面，可以通过 #loginPage(String loginPage) 方法，来进行设置。不过这里我们希望像「2. 快速入门」一样，使用默认的登录界面，所以不进行设置。
<Z> 处，调用 HttpSecurity#logout() 方法，配置退出相关。

如果胖友想要自定义退出页面，可以通过 #logoutUrl(String logoutUrl) 方法，来进行设置。不过这里我们希望像「2. 快速入门」一样，使用默认的退出界面，所以不进行设置。
3.2.2 TestController
在 cn.iocoder.springboot.lab01.springsecurity.controller 包路径下，创建 TestController 类，提供测试 API 接口。代码如下：

// TestController.java

@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/echo")
    public String demo() {
        return "示例返回";
    }

    @GetMapping("/home")
    public String home() {
        return "我是首页";
    }

    @GetMapping("/admin")
    public String admin() {
        return "我是管理员";
    }

    @GetMapping("/normal")
    public String normal() {
        return "我是普通用户";
    }

}
对于 /test/echo 接口，直接访问，无需登录。
对于 /test/home 接口，无法直接访问，需要进行登录。
对于 /test/admin 接口，需要登录「admin/admin」用户，因为需要 ADMIN 角色。
对于 /test/normal 接口，需要登录「normal/normal」用户，因为需要 NORMAL 角色。
胖友可以按照如上的说明，进行各种测试。例如说，登录「normal/normal」用户后，去访问 /test/admin 接口，会返回 403 界面，无权限~

3.3 示例二
在示例二中，我们会看看如何使用 Spring Security 的注解，实现权限控制。

3.3.1 SecurityConfig
修改 SecurityConfig 配置类，增加 @EnableGlobalMethodSecurity 注解，开启对 Spring Security 注解的方法，进行权限验证。代码如下：

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter
3.3.2 DemoController
在 cn.iocoder.springboot.lab01.springsecurity.controller 包路径下，创建 DemoController 类，提供测试 API 接口。代码如下：

// DemoController.java

@RestController
@RequestMapping("/demo")
public class DemoController {

    @PermitAll
    @GetMapping("/echo")
    public String demo() {
        return "示例返回";
    }

    @GetMapping("/home")
    public String home() {
        return "我是首页";
    }

    @PreAuthorize("hasRole('ROLE_ADMIN')")
    @GetMapping("/admin")
    public String admin() {
        return "我是管理员";
    }

    @PreAuthorize("hasRole('ROLE_NORMAL')")
    @GetMapping("/normal")
    public String normal() {
        return "我是普通用户";
    }

}
每个 URL 的权限验证，和「3.2.2 TestController」是一一对应的。

@PermitAll 注解，等价于 #permitAll() 方法，所有用户可访问。

重要！！！因为在「3.2.1 SecurityConfig」中，配置了 .anyRequest().authenticated() ，任何请求，访问的用户都需要经过认证。所以这里 @PermitAll 注解实际是不生效的。

也就是说，Java Config 配置的权限，和注解配置的权限，两者是叠加的。

@PreAuthorize 注解，等价于 #access(String attribute) 方法，，当 Spring EL 表达式的执行结果为 true 时，可以访问。

Spring Security 还有其它注解，不过不太常用，可见《区别： @Secured(), @PreAuthorize() 及 @RolesAllowed()》文章。

胖友可以按照如上的说明，进行各种测试。例如说，登录「normal/normal」用户后，去访问 /test/admin 接口，会返回 403 界面，无权限~

4. 整合 Spring Session
   参见《芋道 Spring Boot 分布式 Session 入门》文章的「5. 整合 Spring Security」小节。

5. 整合 OAuth2
   参见《芋道 Spring Security OAuth2 入门》文章，详细到爆炸。

6. 整合 JWT
   参见《前后端分离 SpringBoot + SpringSecurity + JWT + RBAC 实现用户无状态请求验证》文章，写的很不错。

7. 项目实战
   在开源项目翻了一圈，找到一个相对合适项目 RuoYi-Vue 。主要以下几点原因：

基于 Spring Security 实现。
基于 RBAC 权限模型，并且支持动态的权限配置。
基于 Redis 服务，实现登录用户的信息缓存。
前后端分离。同时前端采用 Vue ，相对来说后端会 Vue 的比 React 的多。
考虑到方便自己添加注释，艿艿 Fork 出一个仓库， 地址是 https://github.com/YunaiV/RuoYi-Vue 。

强烈推荐，生产级 Spring Security 项目实践，支持管理后台 + 用户 App 两种平台！

项目地址：https://github.com/YunaiV/ruoyi-vue-pro

🔥 官方推荐 🔥 RuoYi-Vue 全新 Pro 版本，优化重构所有功能。基于 Spring Boot + MyBatis Plus + Vue & Element 实现的后台管理系统 + 微信小程序，支持 RBAC 动态权限、数据权限、SaaS 多租户、Activiti + Flowable 工作流、三方登录、支付、短信、商城等功能。你的 ⭐️ Star ⭐️，是作者生发的动力！

下面，来跟着艿艿一起走读下 RuoYi-Vue 的权限相关功能。

7.1 表结构
基于 RBAC 权限模型，一共有 5 个表。

对 RBAC 权限模型不了解的胖友，可以看看《到底什么是RBAC权限模型？！》

😈 嘻嘻，艿艿的大学毕业设计，做的就是统一认证中心，2011 年的时候，前后端分离。前端采用 ExtJS 框架，后端自己参考 Spring Security 造的权限框架的轮子，提供 SDK 接入统一认证中心，使用 HTTP 通信。

实体	表	说明
SysUser	sys_user	用户信息
SysRole	sys_role	用户信息
SysUserRole	sys_user_role	用户和角色关联
SysMenu	sys_menu	菜单权限
SysRoleMenu	sys_role_menu	角色和菜单关联
5 个表的关系比较简单：

一个 SysUse ，可以拥有多个 SysRole ，通过 SysUserRole 存储关联。
一个 SysRole ，可以拥有多个 SysMenu ，通过 SysRoleMenu 存储关联。
7.1.1 SysUser
SysUser ，用户实体类。代码如下：

// SysUser.java

public class SysUser extends BaseEntity {

    private static final long serialVersionUID = 1L;

    @Excel(name = "用户序号", cellType = ColumnType.NUMERIC, prompt = "用户编号")
    private Long userId;

    @Excel(name = "部门编号", type = Type.IMPORT)
    private Long deptId;

    @Excel(name = "登录名称")
    private String userName;

    @Excel(name = "用户名称")
    private String nickName;

    @Excel(name = "用户邮箱")
    private String email;
    
    @Excel(name = "手机号码")
    private String phonenumber;

    @Excel(name = "用户性别", readConverterExp = "0=男,1=女,2=未知")
    private String sex;

    /** 用户头像 */
    private String avatar;

    /** 密码 */
    private String password;

    /** 盐加密 */
    private String salt;

    @Excel(name = "帐号状态", readConverterExp = "0=正常,1=停用")
    private String status;

    /** 删除标志（0代表存在 2代表删除） */
    private String delFlag;

    @Excel(name = "最后登录IP", type = Type.EXPORT)
    private String loginIp;

    @Excel(name = "最后登录时间", width = 30, dateFormat = "yyyy-MM-dd HH:mm:ss", type = Type.EXPORT)
    private Date loginDate;

    @Excels({
            @Excel(name = "部门名称", targetAttr = "deptName", type = Type.EXPORT),
            @Excel(name = "部门负责人", targetAttr = "leader", type = Type.EXPORT)
    })
    @Transient
    private SysDept dept;

    /** 角色对象 */
    @Transient
    private List<SysRole> roles;

    /** 角色组 */
    @Transient
    private Long[] roleIds;

    /** 岗位组 */
    @Transient
    private Long[] postIds;
    
    // ...省略 set/get 方法

}
添加 @Transient 注解的字段，非存储字段。后续的实体，补充重复赘述。
每个字段比较简单，胖友自己根据注释理解下即可。
对应表的创建 SQL 如下：

create table sys_user (
user_id           bigint(20)      not null auto_increment    comment '用户ID',
dept_id           bigint(20)      default null               comment '部门ID',
user_name         varchar(30)     not null                   comment '用户账号',
nick_name         varchar(30)     not null                   comment '用户昵称',
user_type         varchar(2)      default '00'               comment '用户类型（00系统用户）',
email             varchar(50)     default ''                 comment '用户邮箱',
phonenumber       varchar(11)     default ''                 comment '手机号码',
sex               char(1)         default '0'                comment '用户性别（0男 1女 2未知）',
avatar            varchar(100)    default ''                 comment '头像地址',
password          varchar(100)    default ''                 comment '密码',
status            char(1)         default '0'                comment '帐号状态（0正常 1停用）',
del_flag          char(1)         default '0'                comment '删除标志（0代表存在 2代表删除）',
login_ip          varchar(50)     default ''                 comment '最后登录IP',
login_date        datetime                                   comment '最后登录时间',
create_by         varchar(64)     default ''                 comment '创建者',
create_time       datetime                                   comment '创建时间',
update_by         varchar(64)     default ''                 comment '更新者',
update_time       datetime                                   comment '更新时间',
remark            varchar(500)    default null               comment '备注',
primary key (user_id)
) engine=innodb auto_increment=100 comment = '用户信息表';
7.1.2 SysRole
SysRole ，角色实体类。代码如下：

// SysRole.java

public class SysRole extends BaseEntity {

    private static final long serialVersionUID = 1L;

    @Excel(name = "角色序号", cellType = ColumnType.NUMERIC)
    private Long roleId;
    
    @Excel(name = "角色名称")
    private String roleName;

    @Excel(name = "角色权限")
    private String roleKey;

    @Excel(name = "角色排序")
    private String roleSort;

    @Excel(name = "数据范围", readConverterExp = "1=所有数据权限,2=自定义数据权限,3=本部门数据权限,4=本部门及以下数据权限")
    private String dataScope;

    @Excel(name = "角色状态", readConverterExp = "0=正常,1=停用")
    private String status;

    /** 删除标志（0代表存在 2代表删除） */
    private String delFlag;

    /** 用户是否存在此角色标识 默认不存在 */
    @Transient
    private boolean flag = false;

    /** 菜单组 */
    @Transient
    private Long[] menuIds;

    /** 部门组（数据权限） */
    @Transient
    private Long[] deptIds;
    
    // ...省略 set/get 方法

}
每个字段比较简单，胖友自己根据注释理解下即可。
对应表的创建 SQL 如下：

create table sys_role (
role_id           bigint(20)      not null auto_increment    comment '角色ID',
role_name         varchar(30)     not null                   comment '角色名称',
role_key          varchar(100)    not null                   comment '角色权限字符串',
role_sort         int(4)          not null                   comment '显示顺序',
data_scope        char(1)         default '1'                comment '数据范围（1：全部数据权限 2：自定数据权限 3：本部门数据权限 4：本部门及以下数据权限）',
status            char(1)         not null                   comment '角色状态（0正常 1停用）',
del_flag          char(1)         default '0'                comment '删除标志（0代表存在 2代表删除）',
create_by         varchar(64)     default ''                 comment '创建者',
create_time       datetime                                   comment '创建时间',
update_by         varchar(64)     default ''                 comment '更新者',
update_time       datetime                                   comment '更新时间',
remark            varchar(500)    default null               comment '备注',
primary key (role_id)
) engine=innodb auto_increment=100 comment = '角色信息表';
7.1.3 SysUserRole
SysUserRole ，用户和角色关联实体类。代码如下：

// SysUserRole.java

public class SysUserRole {

    /** 用户ID */
    private Long userId;

    /** 角色ID */
    private Long roleId;
    
    // ...省略 set/get 方法

}
每个字段比较简单，胖友自己根据注释理解下即可。
roleKey 属性，对应的角色标识字符串，可以对应多个角色标识，使用逗号分隔。例如说："admin,normal" 。
对应表的创建 SQL 如下：

create table sys_user_role (
user_id   bigint(20) not null comment '用户ID',
role_id   bigint(20) not null comment '角色ID',
primary key(user_id, role_id)
) engine=innodb comment = '用户和角色关联表';
7.1.4 SysMenu
SysMenu ，菜单权限实体类。代码如下：

// SysMenu.java

public class SysMenu extends BaseEntity {

    private static final long serialVersionUID = 1L;

    /** 菜单ID */
    private Long menuId;

    /** 菜单名称 */
    private String menuName;

    /** 父菜单名称 */
    private String parentName;

    /** 父菜单ID */
    private Long parentId;

    /** 显示顺序 */
    private String orderNum;

    /** 路由地址 */
    private String path;

    /** 组件路径 */
    private String component;

    /** 是否为外链（0是 1否） */
    private String isFrame;

    /** 类型（M目录 C菜单 F按钮） */
    private String menuType;

    /** 菜单状态:0显示,1隐藏 */
    private String visible;

    /** 权限字符串 */
    private String perms;

    /** 菜单图标 */
    private String icon;

    /** 子菜单 */
    @Transient
    private List<SysMenu> children = new ArrayList<SysMenu>();
    
    // ...省略 set/get 方法

}
😈 个人感觉，这个实体改成 SysResource 资源，更加合适，菜单仅仅是其中的一种。

每个字段比较简单，胖友自己根据资源理解下即可。我们来重点看几个字段。

menuType 属性，定义了三种类型。其中，F 代表按钮，是为了做页面中的功能级的权限。

perms 属性，对应的权限标识字符串。一般格式为 ${大模块}:${小模块}:{操作} 。示例如下：

用户查询：system:user:query
用户新增：system:user:add
用户修改：system:user:edit
用户删除：system:user:remove
用户导出：system:user:export
用户导入：system:user:import
重置密码：system:user:resetPwd
对于前端来说，每个按钮在展示时，可以判断用户是否有该按钮的权限。如果没有，则进行隐藏。当然，前端在首次进入系统的时候，会请求一次权限列表到本地进行缓存。
对于后端来说，每个接口上会添加 @PreAuthorize("@ss.hasPermi('system:user:list')") 注解。在请求接口时，会校验用户是否有该 URL 对应的权限。如果没有，则会抛出权限验证失败的异常。
一个 perms 属性，可以对应多个权限标识，使用逗号分隔。例如说："system:user:query,system:user:add" 。
对应表的创建 SQL 如下：

create table sys_menu (
menu_id           bigint(20)      not null auto_increment    comment '菜单ID',
menu_name         varchar(50)     not null                   comment '菜单名称',
parent_id         bigint(20)      default 0                  comment '父菜单ID',
order_num         int(4)          default 0                  comment '显示顺序',
path              varchar(200)    default ''                 comment '路由地址',
component         varchar(255)    default null               comment '组件路径',
is_frame          int(1)          default 1                  comment '是否为外链（0是 1否）',
menu_type         char(1)         default ''                 comment '菜单类型（M目录 C菜单 F按钮）',
visible           char(1)         default 0                  comment '菜单状态（0显示 1隐藏）',
perms             varchar(100)    default null               comment '权限标识',
icon              varchar(100)    default '#'                comment '菜单图标',
create_by         varchar(64)     default ''                 comment '创建者',
create_time       datetime                                   comment '创建时间',
update_by         varchar(64)     default ''                 comment '更新者',
update_time       datetime                                   comment '更新时间',
remark            varchar(500)    default ''                 comment '备注',
primary key (menu_id)
) engine=innodb auto_increment=2000 comment = '菜单权限表';
7.1.5 SysRoleMenu
SysRoleMenu ，菜单权限实体类。代码如下：

// SysRoleMenu.java

public class SysRoleMenu {

    /** 角色ID */
    private Long roleId;

    /** 菜单ID */
    private Long menuId;
    
    // ...省略 set/get 方法

}
每个字段比较简单，胖友自己根据注释理解下即可。
对应表的创建 SQL 如下：

create table sys_role_menu (
role_id   bigint(20) not null comment '角色ID',
menu_id   bigint(20) not null comment '菜单ID',
primary key(role_id, menu_id)
) engine=innodb comment = '角色和菜单关联表';
7.2 SecurityConfig
在 SecurityConfig 配置类，继承 WebSecurityConfigurerAdapter 抽象类，实现 Spring Security 在 Web 场景下的自定义配置。代码如下：

// SecurityConfig.java

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // ...

}
涉及到的配置方法较多，我们逐个来看看。
重写 #configure(AuthenticationManagerBuilder auth) 方法，实现 AuthenticationManager 认证管理器。代码如下：

// SecurityConfig.java

/**
* 自定义用户认证逻辑
  */
  @Autowired
  private UserDetailsService userDetailsService;

/**
* 身份认证接口
  */
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth.userDetailsService(userDetailsService) // <X>
  .passwordEncoder(bCryptPasswordEncoder()); // <Y>
  }

/**
* 强散列哈希加密实现
  */
  @Bean
  public BCryptPasswordEncoder bCryptPasswordEncoder() {
  return new BCryptPasswordEncoder();
  }
  <X> 处，调用 AuthenticationManagerBuilder#userDetailsService(userDetailsService) 方法，使用自定义实现的 UserDetailsService 实现类，更加灵活且自由的实现认证的用户信息的读取。在「7.3.1 加载用户信息」中，我们会看到 RuoYi-Vue 对 UserDetailsService 的自定义实现类。
  <Y> 处，调用 AbstractDaoAuthenticationConfigurer#passwordEncoder(passwordEncoder) 方法，设置 PasswordEncoder 密码编码器。这里，就使用了 bCryptPasswordEncoder 强散列哈希加密实现。
  重写 #configure(HttpSecurity httpSecurity) 方法，主要配置 URL 的权限控制。代码如下：

// SecurityConfig.java

/**
* 认证失败处理类
  */
  @Autowired
  private AuthenticationEntryPointImpl unauthorizedHandler;

/**
* 退出处理类
  */
  @Autowired
  private LogoutSuccessHandlerImpl logoutSuccessHandler;

/**
* token 认证过滤器
  */
  @Autowired
  private JwtAuthenticationTokenFilter authenticationTokenFilter;

@Override
protected void configure(HttpSecurity httpSecurity) throws Exception {
httpSecurity
// CRSF禁用，因为不使用session
.csrf().disable()
// <X> 认证失败处理类
.exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
// 基于token，所以不需要session
.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
// 过滤请求
.authorizeRequests()
// <Y> 对于登录login 验证码captchaImage 允许匿名访问
.antMatchers("/login", "/captchaImage").anonymous()
.antMatchers(
HttpMethod.GET,
"/*.html",
"/**/*.html",
"/**/*.css",
"/**/*.js"
).permitAll()
.antMatchers("/profile/**").anonymous()
.antMatchers("/common/download**").anonymous()
.antMatchers("/swagger-ui.html").anonymous()
.antMatchers("/swagger-resources/**").anonymous()
.antMatchers("/webjars/**").anonymous()
.antMatchers("/*/api-docs").anonymous()
.antMatchers("/druid/**").anonymous()
// 除上面外的所有请求全部需要鉴权认证
.anyRequest().authenticated()
.and()
.headers().frameOptions().disable();
httpSecurity.logout().logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler); // <Z>
// <P> 添加 JWT filter
httpSecurity.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
}
比较长，我们选择重点的来看。
<X> 处，设置认证失败时的处理器为 unauthorizedHandler 。详细解析，见「7.6.1 AuthenticationEntryPointImpl」。
<Y> 处，设置用于登录的 /login 接口，允许匿名访问。这样，后续我们就可以使用自定义的登录接口。详细解析，见「7.3 登录 API 接口」。
<Z> 处，设置登出成功的处理器为 logoutSuccessHandler 。详细解析，见「7.6.3 LogoutSuccessHandlerImpl」。
<P> 处，添加 JWT 认证过滤器 authenticationTokenFilter ，用于用户使用用户名与密码登录完成后，后续请求基于 JWT 来认证。 详细解析，见「7.4 JwtAuthenticationTokenFilter」。
重写 #authenticationManagerBean 方法，解决无法直接注入 AuthenticationManager 的问题。代码如下：

// SecurityConfig.java

@Bean
@Override
public AuthenticationManager authenticationManagerBean() throws Exception {
return super.authenticationManagerBean();
}
在方法上，额外添加了 @Bean 注解，保证创建出 AuthenticationManager Bean 。
下面，我们详细的来看看，各个配置的 Bean 的逻辑。

7.3 登录 API 接口
SysLoginController#login(...)

在 SysLoginController 中，定义了 /login 接口，提供登录功能。代码如下：

// SysLoginController.java

@Autowired
private SysLoginService loginService;

/**
* 登录方法
*
* @param username 用户名
* @param password 密码
* @param code 验证码
* @param uuid 唯一标识
* @return 结果
  */
  @PostMapping("/login")
  public AjaxResult login(String username, String password, String code, String uuid) {
  AjaxResult ajax = AjaxResult.success();
  // 生成令牌
  String token = loginService.login(username, password, code, uuid);
  ajax.put(Constants.TOKEN, token);
  return ajax;
  }
  在内部，会调用 loginService#login(username, password, code, uuid) 方法，会进行基于用户名与密码的登录认证。认证通过后，返回身份 TOKEN 。

登录成功后，该接口响应示例如下

{
"msg": "操作成功",
"code": 200,
"token": "eyJhbGciOiJIUzUxMiJ9.eyJsb2dpbl91c2VyX2tleSI6ImJkN2Q4OTZiLTU2NTAtNGIyZS1iNjFjLTc0MjlkYmRkNzA1YyJ9.lkU8ot4GecLHs7VAcRAo1fLMOaFryd4W5Q_a2wzPwcOL0Kiwyd4enpnGd79A_aQczXC-JB8vELNcNn7BrtJn9A"
}
后续，前端在请求后端接口时，在请求头上带头该 token 值，作为用户身份标识。
SysLoginService#login(...)

在 SysLoginService 中，定义了 #login(username, password, code, uuid) 方法，进行基于用户名与密码的登录认证。认证通过后，返回身份 TOKEN 。代码如下：

// SysLoginService.java

@Autowired
private TokenService tokenService;

@Resource
private AuthenticationManager authenticationManager;

@Autowired
private RedisCache redisCache;

/**
* 登录验证
*
* @param username 用户名
* @param password 密码
* @param code     验证码
* @param uuid     唯一标识
* @return 结果
  */
  public String login(String username, String password, String code, String uuid) {
  // <1> 验证图片验证码的正确性
  String verifyKey = Constants.CAPTCHA_CODE_KEY + uuid; // uuid 的作用，是获得对应的图片验证码
  String captcha = redisCache.getCacheObject(verifyKey); // 从 Redis 中，获得图片验证码
  redisCache.deleteObject(verifyKey); // 从 Redis 中，删除图片验证码
  if (captcha == null) { // 图片验证码不存在
  AsyncManager.me().execute(AsyncFactory.recordLogininfor(username, Constants.LOGIN_FAIL, MessageUtils.message("user.jcaptcha.error")));
  throw new CaptchaExpireException();
  }
  if (!code.equalsIgnoreCase(captcha)) { // 图片验证码不正确
  AsyncManager.me().execute(AsyncFactory.recordLogininfor(username, Constants.LOGIN_FAIL, MessageUtils.message("user.jcaptcha.expire")));
  throw new CaptchaException();
  }
  // <2> 用户验证
  Authentication authentication;
  try {
  // 该方法会去调用 UserDetailsServiceImpl.loadUserByUsername
  authentication = authenticationManager
  .authenticate(new UsernamePasswordAuthenticationToken(username, password));
  } catch (Exception e) {
  // <2.1> 发生异常，说明验证不通过，记录相应的登录失败日志
  if (e instanceof BadCredentialsException) {
  AsyncManager.me().execute(AsyncFactory.recordLogininfor(username, Constants.LOGIN_FAIL, MessageUtils.message("user.password.not.match")));
  throw new UserPasswordNotMatchException();
  } else {
  AsyncManager.me().execute(AsyncFactory.recordLogininfor(username, Constants.LOGIN_FAIL, e.getMessage()));
  throw new CustomException(e.getMessage());
  }
  }
  // <2.2> 验证通过，记录相应的登录成功日志
  AsyncManager.me().execute(AsyncFactory.recordLogininfor(username, Constants.LOGIN_SUCCESS, MessageUtils.message("user.login.success")));
  // <3> 生成 Token
  LoginUser loginUser = (LoginUser) authentication.getPrincipal();
  return tokenService.createToken(loginUser);
  }
  <1> 处，验证图片验证码的正确性。该验证码会存储在 Redis 缓存中，通过 uuid 作为对应的标识。生成的逻辑，胖友自己看 CaptchaController 提供的 /captchaImage 接口。
  <2> 处，调用 Spring Security 的 AuthenticationManager 的 #authenticate(UsernamePasswordAuthenticationToken authentication) 方法，基于用户名与密码的登录认证。在其内部，会调用我们定义的 UserDetailsServiceImpl 的 #loadUserByUsername(String username) 方法，获得指定用户名对应的用户信息。详细解析，见「7.3.1 加载用户信息」。
  <2.1> 处，发生异常，说明认证不通过，记录相应的登录失败日志。
  <2.2> 处，未发生异常，说明认证通过，记录相应的登录成功日志。
  关于上述日志，我们在「7.7 登录日志」来讲。
  <3> 处，调用 TokenService 的 #createToken(LoginUser loginUser) 方法，给认证通过的用户，生成其对应的认证 TOKEN 。这样，该用户的后续请求，就使用该 TOKEN 作为身份标识进行认证。
  7.3.1 加载用户信息
  在 UserDetailsServiceImpl 中，实现 Spring Security UserDetailsService 接口，实现了该接口定义的 #loadUserByUsername(String username) 方法，获得指定用户名对应的用户信息。代码如下：

// UserDetailsServiceImpl.java

private static final Logger log = LoggerFactory.getLogger(UserDetailsServiceImpl.class);

@Autowired
private ISysUserService userService;

@Autowired
private SysPermissionService permissionService;

@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
// <1> 查询指定用户名对应的 SysUser
SysUser user = userService.selectUserByUserName(username);
// <2> 各种校验
if (StringUtils.isNull(user)) {
log.info("登录用户：{} 不存在.", username);
throw new UsernameNotFoundException("登录用户：" + username + " 不存在");
} else if (UserStatus.DELETED.getCode().equals(user.getDelFlag())) {
log.info("登录用户：{} 已被删除.", username);
throw new BaseException("对不起，您的账号：" + username + " 已被删除");
} else if (UserStatus.DISABLE.getCode().equals(user.getStatus())) {
log.info("登录用户：{} 已被停用.", username);
throw new BaseException("对不起，您的账号：" + username + " 已停用");
}

    // <3> 创建 Spring Security UserDetails 用户明细
    return createLoginUser(user);
}

public UserDetails createLoginUser(SysUser user) {
return new LoginUser(user, permissionService.getMenuPermission(user));
}
<1> 处，调用 ISysUserService 的 #selectUserByUserName(String userName) 方法，查询指定用户名对应的 SysUser 。代码如下：

// SysUserServiceImpl.java
@Autowired
private SysUserMapper userMapper;

@Override
public SysUser selectUserByUserName(String userName) {
return userMapper.selectUserByUserName(userName);
}

// SysUserMapper.XML
<sql id="selectUserVo">
select u.user_id, u.dept_id, u.user_name, u.nick_name, u.email, u.avatar, u.phonenumber, u.password, u.sex, u.status, u.del_flag, u.login_ip, u.login_date, u.create_by, u.create_time, u.remark,
d.dept_id, d.parent_id, d.dept_name, d.order_num, d.leader, d.status as dept_status,
r.role_id, r.role_name, r.role_key, r.role_sort, r.data_scope, r.status as role_status
from sys_user u
left join sys_dept d on u.dept_id = d.dept_id
left join sys_user_role ur on u.user_id = ur.user_id
left join sys_role r on r.role_id = ur.role_id
</sql>

<select id="selectUserByUserName" parameterType="String" resultMap="SysUserResult">
    <include refid="selectUserVo"/>
	where u.user_name = #{userName}
</select>
通过查询 sys_user 表，同时连接 sys_dept、sys_user_role、sys_role 表，将 username 对应的 SysUser 相关信息都一次性查询出来。
返回结果 SysUserResult 的具体定义，点击 传送门 查看，实际就是 SysUser 实体类。
<2> 处，各种校验。如果校验不通过，抛出 UsernameNotFoundException 或 BaseException 异常。

<3> 处，调用 SysPermissionService 的 #getMenuPermission(SysUser user) 方法，获得用户的 SysRoleMenu 的权限标识字符串的集合。代码如下：

// SysPermissionService.java
@Autowired
private ISysMenuService menuService;

public Set<String> getMenuPermission(SysUser user) {
Set<String> roles = new HashSet<String>();
// 管理员拥有所有权限
if (user.isAdmin()) {
roles.add("*:*:*"); // 所有模块
} else {
// 读取
roles.addAll(menuService.selectMenuPermsByUserId(user.getUserId()));
}
return roles;
}

// SysMenuServiceImpl.java
@Autowired
private SysMenuMapper menuMapper;

@Override
public Set<String> selectMenuPermsByUserId(Long userId) {
// 读取 SysMenu 的权限标识数组
List<String> perms = menuMapper.selectMenuPermsByUserId(userId);
// 逐个，按照“逗号”分隔
Set<String> permsSet = new HashSet<>();
for (String perm : perms) {
if (StringUtils.isNotEmpty(perm)) {
permsSet.addAll(Arrays.asList(perm.trim().split(",")));
}
}
return permsSet;
}

// SysMenuMapper.xml
<select id="selectMenuPermsByUserId" parameterType="Long" resultType="String">
select distinct m.perms
from sys_menu m
left join sys_role_menu rm on m.menu_id = rm.menu_id
left join sys_user_role ur on rm.role_id = ur.role_id
where ur.user_id = #{userId}
</select>
虽然代码很长，但是核心的并不多。
首先，如果 SysUser 是超级管理员，则其权限标识集合就是 *:*:* ，标识可以所有模块的所有操作。
然后，查询 sys_menu 表，同时连接 sys_role_menu、sys_user_role 表，将 SysUser 拥有的 SysMenu 的权限标识数组，然后使用 "," 分隔每个 SysMenu 对应的权限标识。
这里，我们看到最终返回的是 LoginUser ，实现 Spring Security UserDetails 接口，自定义的用户明细。代码如下：

// LoginUser.java

public class LoginUser implements UserDetails {

    private static final long serialVersionUID = 1L;

    /** 用户唯一标识 */
    private String token;

    /** 登录时间 */
    private Long loginTime;

    /** 过期时间 */
    private Long expireTime;

    /** 登录IP地址 */
    private String ipaddr;

    /** 登录地点 */
    private String loginLocation;

    /** 浏览器类型 */
    private String browser;

    /** 操作系统 */
    private String os;

    /** 权限列表 */
    private Set<String> permissions;

    /** 用户信息 */
    private SysUser user;
    
    // ...省略 set/get 方法，以及各种实现方法

}
7.3.2 创建认证 Token
在 TokenService 中，定义了 #createToken(LoginUser loginUser) 方法，给认证通过的用户，生成其对应的认证 Token 。代码如下：

// TokenService.java

/**
* 创建令牌
*
* @param loginUser 用户信息
* @return 令牌
  */
  public String createToken(LoginUser loginUser) {
  // <1> 设置 LoginUser 的用户唯一标识。注意，这里虽然变量名叫 token ，其实不是身份认证的 Token
  String token = IdUtils.fastUUID();
  loginUser.setToken(token);
  // <2> 设置用户终端相关的信息，包括 IP、城市、浏览器、操作系统
  setUserAgent(loginUser);

  // <3> 记录缓存
  refreshToken(loginUser);

  // <4> 生成 JWT 的 Token
  Map<String, Object> claims = new HashMap<>();
  claims.put(Constants.LOGIN_USER_KEY, token);
  return createToken(claims);
  }
  注意，这个方法不仅仅会生成认证 Token ，还会缓存 LoginUser 到 Redis 缓存中。

<1> 处，设置 LoginUser 的用户唯一标识，即 LoginUser.token。注意，这里虽然变量名叫 token ，其实不是身份认证的 Token 。

<2> 处，调用 #setUserAgent(LoginUser loginUser) 方法，设置用户终端相关的信息，包括 IP、城市、浏览器、操作系统。代码如下：

// TokenService.java

public void setUserAgent(LoginUser loginUser) {
UserAgent userAgent = UserAgent.parseUserAgentString(ServletUtils.getRequest().getHeader("User-Agent"));
String ip = IpUtils.getIpAddr(ServletUtils.getRequest());
loginUser.setIpaddr(ip);
loginUser.setLoginLocation(AddressUtils.getRealAddressByIP(ip));
loginUser.setBrowser(userAgent.getBrowser().getName());
loginUser.setOs(userAgent.getOperatingSystem().getName());
}
<3> 处，调用 #refreshToken(LoginUser loginUser) 方法，缓存 LoginUser 到 Redis 缓存中。代码如下：

// application.yaml
# token配置
token:
# 令牌有效期（默认30分钟）
expireTime: 30

// Constants.java
/**
* 登录用户 redis key
  */
  public static final String LOGIN_TOKEN_KEY = "login_tokens:";

// TokenService.java
// 令牌有效期（默认30分钟）
@Value("${token.expireTime}")
private int expireTime;

@Autowired
private RedisCache redisCache;

public void refreshToken(LoginUser loginUser) {
loginUser.setLoginTime(System.currentTimeMillis());
loginUser.setExpireTime(loginUser.getLoginTime() + expireTime * MILLIS_MINUTE);
// 根据 uuid 将 loginUser 缓存
String userKey = getTokenKey(loginUser.getToken());
redisCache.setCacheObject(userKey, loginUser, expireTime, TimeUnit.MINUTES);
}

private String getTokenKey(String uuid) {
return Constants.LOGIN_TOKEN_KEY + uuid;
}
缓存的 Redis Key 的统一前缀为 "login_tokens:" ，使用 Login 的用户唯一标识(LoginUser.token)作为后缀。
<4> 处，调用 #createToken(Map<String, Object> claims) 方法，生成 JWT 的 Token 。代码如下：

// application.yaml
# token配置
token:
# 令牌秘钥
secret: abcdefghijklmnopqrstuvwxyz

// TokenService.java
// 令牌秘钥
@Value("${token.secret}")
private String secret;

private String createToken(Map<String, Object> claims) {
return Jwts.builder()
.setClaims(claims)
.signWith(SignatureAlgorithm.HS512, secret).compact();
}
这里，我们采用了 jjwt 库。
对 JWT 不了解的胖友，可以阅读下《JSON Web Token - 在Web应用间安全地传递信息》和《八幅漫画理解使用 JSON Web Token 设计单点登录系统》文章。
注意，不要把这里生成的 JWT 的 Token ，和我们上面的 LoginUser.token 混淆在一起。
因为 LoginUser.token 添加到 claims 中，最终生成了 JWT 的 Token 。所以，后续我们可以通过解码该 JWT 的 Token ，从而获得 claims ，最终获得到对应的 LoginUser.token 。
在 JWT 的 Token 中，使用 LoginUser.token 而不是 userId 的好处，可以带来更好的安全性，避免万一 secret 秘钥泄露之后，黑客可以顺序生成 userId 从而生成对应的 JWT 的 Token ，后续直接可以操作该用户的数据。不过，这么做之后就不是纯粹的 JWT ，解析出来的 LoginUser.token 需要查询对应的 LoginUser 的 Redis 缓存。详细的，我们在「7.4 JwtAuthenticationTokenFilter」会看到这个过程。
至此，我们完成了基于用户名与密码的登录认证的整个过程。虽然整个过程的代码有小几百行，不过逻辑还是比较清晰明了的。如果不太理解的胖友，可以在反复看两遍。

7.4 JwtAuthenticationTokenFilter
在 JwtAuthenticationTokenFilter 中，继承 OncePerRequestFilter 过滤器，实现了基于 Token 的认证。代码如下：

// JwtAuthenticationTokenFilter.java

@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private TokenService tokenService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        // <1> 获得当前 LoginUser
        LoginUser loginUser = tokenService.getLoginUser(request);
        // 如果存在 LoginUser ，并且未认证过
        if (StringUtils.isNotNull(loginUser) && StringUtils.isNull(SecurityUtils.getAuthentication())) {
            // <2> 校验 Token 有效性
            tokenService.verifyToken(loginUser);
            // <3> 创建 UsernamePasswordAuthenticationToken 对象，设置到 SecurityContextHolder 中
            UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(loginUser, null, loginUser.getAuthorities());
            authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        }
        // <4> 继续过滤器
        chain.doFilter(request, response);
    }

}
<1> 处，调用 TokenService 的 #getLoginUser(request) 方法，获得当前 LoginUser 。代码如下：

// application.yaml
# token配置
token:
# 令牌自定义标识
header: Authorization

// TokenService.jav
// 令牌自定义标识
@Value("${token.header}")
private String header;

/**
* 获取用户身份信息
*
* @return 用户信息
  */
  public LoginUser getLoginUser(HttpServletRequest request) {
  // <1.1> 获取请求携带的令牌
  String token = getToken(request);
  if (StringUtils.isNotEmpty(token)) {
  // <1.2> 解析 JWT 的 Token
  Claims claims = parseToken(token);
  // <1.3> 从 Redis 缓存中，获得对应的 LoginUser
  String uuid = (String) claims.get(Constants.LOGIN_USER_KEY);
  String userKey = getTokenKey(uuid);
  return redisCache.getCacheObject(userKey);
  }
  return null;
  }

private String getToken(HttpServletRequest request) {
String token = request.getHeader(header);
if (StringUtils.isNotEmpty(token) && token.startsWith(Constants.TOKEN_PREFIX)) {
token = token.replace(Constants.TOKEN_PREFIX, "");
}
return token;
}

private Claims parseToken(String token) {
return Jwts.parser()
.setSigningKey(secret)
.parseClaimsJws(token)
.getBody();
}
<1.1> 处，调用 #getToken(request) 方法，从请求头 "Authorization" 中，获得身份认证的 Token 。
<1.2> 处，调用 #parseToken(token) 方法，解析 JWT 的 Token ，获得 Claims 对象，从而获得用户唯一标识(LoginUser.token)。
<1.3> 处，从 Redis 缓存中，获得对应的 LoginUser 。
<2> 处，调用 TokenService 的 #verifyToken(LoginUser loginUser) 方法，验证令牌有效期。代码如下：

// TokenService.java
protected static final long MILLIS_SECOND = 1000;
protected static final long MILLIS_MINUTE = 60 * MILLIS_SECOND;
private static final Long MILLIS_MINUTE_TEN = 20 * 60 * 1000L;

/**
* 验证令牌有效期，相差不足 20 分钟，自动刷新缓存
*
* @param loginUser 用户
  */
  public void verifyToken(LoginUser loginUser) {
  long expireTime = loginUser.getExpireTime();
  long currentTime = System.currentTimeMillis();
  // 相差不足 20 分钟，自动刷新缓存
  if (expireTime - currentTime <= MILLIS_MINUTE_TEN) {
  String token = loginUser.getToken();
  loginUser.setToken(token);
  refreshToken(loginUser);
  }
  }
  实际上，这个方法的目的不是验证 Token 的有效性，而是刷新对应的 LoginUser 的缓存的过期时间。
  考虑到避免每次请求都去刷新缓存的过期时间，所以过期时间不足 20 分钟，才会去刷新。
  <3> 处，手动创建 UsernamePasswordAuthenticationToken 对象，设置到 SecurityContextHolder 中。😈 因为，我们已经通过 Token 来完成认证了。

<4> 处，继续过滤器。

严格来说，RuoYi-Vue 并不是采用的无状态的 JWT ，而是使用 JWT 的 Token 的生成方式。

7.5 权限验证
在「3. 进阶使用」中，我们看到可以通过 Spring Security 提供的 @PreAuthorize 注解，实现基于 Spring EL 表达式的执行结果为 true 时，可以访问，从而实现灵活的权限校验。

在 RuoYi-Vue 中，通过 @PreAuthorize 注解的特性，使用其 PermissionService 提供的权限验证的方法。使用示例如下：

// SysDictDataController.java

@PreAuthorize("@ss.hasPermi('system:dict:list')")
@GetMapping("/list")
请求 /system/dict/data/list 接口，会调用 PermissionService 的 #hasPermi(String permission) 方法，校验用户是否有指定的权限。
为什么这里会有一个 @ss 呢？在 Spring EL 表达式中，调用指定 Bean 名字的方法时，使用 @ + Bean 的名字。在 RuoYi-Vue 中，声明 PermissionService 的 Bean 名字为 ss 。
7.5.1 判断是否有权限
在 PermissionService 中，定义了 #hasPermi(String permission) 方法，判断当前用户是否有指定的权限。代码如下：

// PermissionService.java

/**
* 所有权限标识
  */
  private static final String ALL_PERMISSION = "*:*:*";

@Autowired
private TokenService tokenService;

/**
* 验证用户是否具备某权限
*
* @param permission 权限字符串
* @return 用户是否具备某权限
  */
  public boolean hasPermi(String permission) {
  // 如果未设置需要的权限，强制不具备。
  if (StringUtils.isEmpty(permission)) {
  return false;
  }
  // 获得当前 LoginUser
  LoginUser loginUser = tokenService.getLoginUser(ServletUtils.getRequest());
  // 如果不存在，或者没有任何权限，说明权限验证不通过
  if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getPermissions())) {
  return false;
  }
  // 判断是否包含权限
  return hasPermissions(loginUser.getPermissions(), permission);
  }

/**
* 判断是否包含权限
*
* @param permissions 权限列表
* @param permission  权限字符串
* @return 用户是否具备某权限
  */
  private boolean hasPermissions(Set<String> permissions, String permission) {
  return permissions.contains(ALL_PERMISSION) || permissions.contains(StringUtils.trim(permission));
  }
  比较简单，胖友看看艿艿添加的代码注释，就能够明白。
  在 PermissionService 中，定义了 #lacksPermi(String permission) 方法，判断当前用户是否没有指定的权限。代码如下：

// PermissionService.java

/**
* 验证用户是否不具备某权限，与 hasPermi逻辑相反
*
* @param permission 权限字符串
* @return 用户是否不具备某权限
  */
  public boolean lacksPermi(String permission) {
  return !hasPermi(permission);
  }
  在 PermissionService 中，定义了 #hasAnyPermi(String permissions) 方法，判断当前用户是否有指定的任一权限。代码如下：

// PermissionService.java

private static final String PERMISSION_DELIMETER = ",";

/**
* 验证用户是否具有以下任意一个权限
*
* @param permissions 以 PERMISSION_NAMES_DELIMETER 为分隔符的权限列表
* @return 用户是否具有以下任意一个权限
  */
  public boolean hasAnyPermi(String permissions) {
  // 如果未设置需要的权限，强制不具备。
  if (StringUtils.isEmpty(permissions)) {
  return false;
  }
  // 获得当前 LoginUser
  LoginUser loginUser = tokenService.getLoginUser(ServletUtils.getRequest());
  // 如果不存在，或者没有任何权限，说明权限验证不通过
  if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getPermissions())) {
  return false;
  }
  // 判断是否包含指定的任一权限
  Set<String> authorities = loginUser.getPermissions();
  for (String permission : permissions.split(PERMISSION_DELIMETER)) {
  if (permission != null && hasPermissions(authorities, permission)) {
  return true;
  }
  }
  return false;
  }
  7.5.2 判断是否有角色
  在 PermissionService 中，定义了 #hasRole(String role) 方法，判断当前用户是否有指定的角色。代码如下：

// PermissionService.java

/**
* 判断用户是否拥有某个角色
*
* @param role 角色字符串
* @return 用户是否具备某角色
  */
  public boolean hasRole(String role) {
  // 如果未设置需要的角色，强制不具备。
  if (StringUtils.isEmpty(role)) {
  return false;
  }
  // 获得当前 LoginUser
  LoginUser loginUser = tokenService.getLoginUser(ServletUtils.getRequest());
  // 如果不存在，或者没有任何角色，说明权限验证不通过
  if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getUser().getRoles())) {
  return false;
  }
  // 判断是否包含指定角色
  for (SysRole sysRole : loginUser.getUser().getRoles()) {
  String roleKey = sysRole.getRoleKey();
  if (SUPER_ADMIN.contains(roleKey) // 超级管理员的特殊处理
  || roleKey.contains(StringUtils.trim(role))) {
  return true;
  }
  }
  return false;
  }
  比较简单，胖友看看艿艿添加的代码注释，就能够明白。
  在 PermissionService 中，定义了 #lacksRole(String role) 方法，判断当前用户是否没有指定的角色。代码如下：

// PermissionService.java

/**
* 验证用户是否不具备某角色，与 isRole逻辑相反。
*
* @param role 角色名称
* @return 用户是否不具备某角色
  */
  public boolean lacksRole(String role) {
  return !hasRole(role);
  }
  在 PermissionService 中，定义了 #hasAnyRoles(String roles) 方法，判断当前用户是否有指定的任一角色。代码如下：

// PermissionService.java

private static final String ROLE_DELIMETER = ",";

/**
* 验证用户是否具有以下任意一个角色
*
* @param roles 以 ROLE_NAMES_DELIMETER 为分隔符的角色列表
* @return 用户是否具有以下任意一个角色
  */
  public boolean hasAnyRoles(String roles) {
  // 如果未设置需要的角色，强制不具备。
  if (StringUtils.isEmpty(roles)) {
  return false;
  }
  // 获得当前 LoginUser
  LoginUser loginUser = tokenService.getLoginUser(ServletUtils.getRequest());
  // 如果不存在，或者没有任何角色，说明权限验证不通过
  if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getUser().getRoles())) {
  return false;
  }
  // 判断是否包含指定的任一角色
  for (String role : roles.split(ROLE_DELIMETER)) {
  if (hasRole(role)) { // 这里实现有点问题，会循环调用 hasRole 方法，重复从 Redis 中读取当前 LoginUser
  return true;
  }
  }
  return false;
  }
  7.6 各种处理器
  在 Ruoyi-Vue 中，提供了各种处理器，处理各种情况，所以我们汇总在「7.6 各种处理器」 中，一起来瞅瞅。

7.6.1 AuthenticationEntryPointImpl
在 AuthenticationEntryPointImpl 中，实现 Spring Security AuthenticationEntryPoint 接口，处理认失败的 AuthenticationException 异常。代码如下：

// AuthenticationEntryPointImpl.java

// 认证失败处理类 返回未授权
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint, Serializable {

    private static final long serialVersionUID = -8970718410437077606L;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) {
        // 响应认证不通过
        int code = HttpStatus.UNAUTHORIZED;
        String msg = StringUtils.format("请求访问：{}，认证失败，无法访问系统资源", request.getRequestURI());
        ServletUtils.renderString(response, JSON.toJSONString(AjaxResult.error(code, msg)));
    }

}
响应认证不通过的 JSON 字符串。
7.6.2 GlobalExceptionHandler
在 GlobalExceptionHandler 中，定义了对 Spring Security 的异常处理。代码如下：

// GlobalExceptionHandler.java

@RestControllerAdvice
public class GlobalExceptionHandler {

@ExceptionHandler(AccessDeniedException.class) // 没有访问权限。使用 @PreAuthorize 校验权限不通过时，就会抛出 AccessDeniedException 异常
public AjaxResult handleAuthorizationException(AccessDeniedException e) {
log.error(e.getMessage());
return AjaxResult.error(HttpStatus.FORBIDDEN, "没有权限，请联系管理员授权");
}

    @ExceptionHandler(AccountExpiredException.class) // 账号已过期
    public AjaxResult handleAccountExpiredException(AccountExpiredException e) {
        log.error(e.getMessage(), e);
        return AjaxResult.error(e.getMessage());
    }

    @ExceptionHandler(UsernameNotFoundException.class) // 用户名不存在
    public AjaxResult handleUsernameNotFoundException(UsernameNotFoundException e) {
        log.error(e.getMessage(), e);
        return AjaxResult.error(e.getMessage());
    }

    // ... 省略对其它的异常类的处理的方法
}
基于 Spring MVC 提供的 @RestControllerAdvice + @ExceptionHandler 注解，实现全局异常的处理。不了解的胖友，可以看看《芋道 Spring Boot SpringMVC 入门》的「5. 全局异常处理」小节。
7.6.3 LogoutSuccessHandlerImpl
在 LogoutSuccessHandlerImpl 中，实现 Spring Security LogoutSuccessHandler 接口，自定义退出的处理，主动删除 LoginUser 在 Redis 中的缓存。代码如下：

// LogoutSuccessHandlerImpl.java

// 自定义退出处理类 返回成功
@Configuration
public class LogoutSuccessHandlerImpl implements LogoutSuccessHandler {

    @Autowired
    private TokenService tokenService;

    /**
     * 退出处理
     */
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        // <1> 获得当前 LoginUser
        LoginUser loginUser = tokenService.getLoginUser(request);
        // 如果有登录的情况下
        if (StringUtils.isNotNull(loginUser)) {
            String userName = loginUser.getUsername();
            // <2> 删除用户缓存记录
            tokenService.delLoginUser(loginUser.getToken());
            // <3> 记录用户退出日志
            AsyncManager.me().execute(AsyncFactory.recordLogininfor(userName, Constants.LOGOUT, "退出成功"));
        }
        // <4> 响应退出成功
        ServletUtils.renderString(response, JSON.toJSONString(AjaxResult.error(HttpStatus.SUCCESS, "退出成功")));
    }

}
<1> 处，调用 TokenService 的 #getLoginUser(request) 方法，获得当前 LoginUser 。

<2> 处，调用 TokenService 的 #delLoginUser(String token) 方法，删除 LoginUser 的 Redis 缓存。代码如下：

// TokenService.java

public void delLoginUser(String token) {
if (StringUtils.isNotEmpty(token)) {
String userKey = getTokenKey(token);
// 删除缓存
redisCache.deleteObject(userKey);
}
}
<3> 处，记录相应的退出成功日志。

<4> 处，响应退出成功的 JSON 字符串。

7.7 登录日志
SysLogininfor ，登录日志实体。代码如下：

// SysLogininfor.java

public class SysLogininfor extends BaseEntity  {

    private static final long serialVersionUID = 1L;

    @Excel(name = "序号", cellType = ColumnType.NUMERIC)
    private Long infoId;

    @Excel(name = "用户账号")
    private String userName;

    @Excel(name = "登录状态", readConverterExp = "0=成功,1=失败")
    private String status;

    @Excel(name = "登录地址")
    private String ipaddr;

    @Excel(name = "登录地点")
    private String loginLocation;

    @Excel(name = "浏览器")
    private String browser;

    @Excel(name = "操作系统")
    private String os;

    @Excel(name = "提示消息")
    private String msg;

    @Excel(name = "访问时间", width = 30, dateFormat = "yyyy-MM-dd HH:mm:ss")
    private Date loginTime;
    
    // ...省略 set/get 方法
}
每个字段比较简单，胖友自己根据注释理解下即可。
对应表的创建 SQL 如下：

create table sys_logininfor (
info_id        bigint(20)     not null auto_increment   comment '访问ID',
user_name      varchar(50)    default ''                comment '用户账号',
ipaddr         varchar(50)    default ''                comment '登录IP地址',
login_location varchar(255)   default ''                comment '登录地点',
browser        varchar(50)    default ''                comment '浏览器类型',
os             varchar(50)    default ''                comment '操作系统',
status         char(1)        default '0'               comment '登录状态（0成功 1失败）',
msg            varchar(255)   default ''                comment '提示消息',
login_time     datetime                                 comment '访问时间',
primary key (info_id)
) engine=innodb auto_increment=100 comment = '系统访问记录';
在 RuoYi-Vue 中，记录 SysLogininfor 的过程如下：

首先，手动调用 AsyncFactory#recordLogininfor(username, status, message, args) 方法，创建一个 Java TimerTask 任务。
然后调用 AsyncManager#execute(TimerTask task) 方法，提交到定时任务的线程中，延迟 OPERATE_DELAY_TIME = 10 秒后，存储该记录到数据库中。
这样的好处，是可以实现异步存储日志到数据库中，提升 API 接口的性能。不过实际上，Spring 提供了 @Async 注解，方便的实现异步操作。不了解的胖友，可以看看《芋道 Spring Boot 异步任务入门》。

另外，在 RuoYi-Vue 中还定义了 SysOperLog ，操作日志实体类。感兴趣的胖友，自己去瞅瞅。

7.8 获得用户信息 API 接口
在 SysLoginController 中，定义了 /getInfo 接口，获取登录的用户信息。代码如下：

// SysLoginController.java

/**
* 获取用户信息
*
* @return 用户信息
  */
  @GetMapping("getInfo")
  public AjaxResult getInfo() {
  // <1> 获得当前 LoginUser
  LoginUser loginUser = tokenService.getLoginUser(ServletUtils.getRequest());
  SysUser user = loginUser.getUser();
  // <2> 角色标识的集合
  Set<String> roles = permissionService.getRolePermission(user);
  // <3> 权限集合
  Set<String> permissions = permissionService.getMenuPermission(user);
  // <4> 返回结果
  AjaxResult ajax = AjaxResult.success();
  ajax.put("user", user);
  ajax.put("roles", roles);
  ajax.put("permissions", permissions);
  return ajax;
  }
  <1> 处，调用 TokenService 的 #getLoginUser(request) 方法，获得当前 LoginUser 。

<2> 处，调用 PermissionService 的 #getRolePermission(SysUser user) 方法，获得 LoginUser 拥有的角色标识的集合。代码如下：

// SysPermissionService.java
@Autowired
private ISysRoleService roleService;

/**
* 获取角色数据权限
*
* @param user 用户信息
* @return 角色权限信息
  */
  public Set<String> getRolePermission(SysUser user) {
  Set<String> roles = new HashSet<String>();
  // 管理员拥有所有权限
  if (user.isAdmin()) { // 如果是管理员，强制添加 admin 角色
  roles.add("admin");
  } else { // 如果非管理员，进行查询
  roles.addAll(roleService.selectRolePermissionByUserId(user.getUserId()));
  }
  return roles;
  }

// SysRoleServiceImpl.java

@Autowired
private SysRoleMapper roleMapper;

/**
* 根据用户ID查询权限
*
* @param userId 用户ID
* @return 权限列表
  */
  @Override
  public Set<String> selectRolePermissionByUserId(Long userId) {
  // 获得 userId 拥有的 SysRole 数组
  List<SysRole> perms = roleMapper.selectRolePermissionByUserId(userId);
  // 遍历 SysRole 数组，生成角色标识数组
  Set<String> permsSet = new HashSet<>();
  for (SysRole perm : perms) {
  if (StringUtils.isNotNull(perm)) {
  permsSet.addAll(Arrays.asList(perm.getRoleKey().trim().split(",")));
  }
  }
  return permsSet;
  }

// SysRoleMapper.xml
<sql id="selectRoleVo">
select distinct r.role_id, r.role_name, r.role_key, r.role_sort, r.data_scope,
r.status, r.del_flag, r.create_time, r.remark
from sys_role r
left join sys_user_role ur on ur.role_id = r.role_id
left join sys_user u on u.user_id = ur.user_id
left join sys_dept d on u.dept_id = d.dept_id
</sql>

<select id="selectRolePermissionByUserId" parameterType="Long" resultMap="SysRoleResult">
	<include refid="selectRoleVo"/>
	WHERE r.del_flag = '0' and ur.user_id = #{userId}
</select>
通过查询 sys_role 表，同时连接 sys_user_role、sys_user、sys_dept 表，将 userId 对应的 SysRole 相关信息都一次性查询出来。
返回结果 SysRoleResult 的具体定义，点击 传送门 查看，实际就是 SysRole 实体类。
<3> 处，调用 SysPermissionService 的 #getMenuPermission(SysUser user) 方法，获得用户的 SysRoleMenu 的权限标识字符串的集合。

<4> 处，返回用户信息的 AjaxResult 结果。

通过调用该 /getInfo 接口，前端就可以根据角色标识、又或者权限标识，实现对页面级别的按钮实现权限控制，进行有权限时显示，无权限时隐藏。

7.9 获取路由信息
在 SysLoginController 中，定义了 /getRouters 接口，获取获取路由信息。代码如下：

// SysLoginController.java

@GetMapping("getRouters")
public AjaxResult getRouters() {
// 获得当前 LoginUser
LoginUser loginUser = tokenService.getLoginUser(ServletUtils.getRequest());
// 获得用户的 SysMenu 数组
SysUser user = loginUser.getUser();
List<SysMenu> menus = menuService.selectMenuTreeByUserId(user.getUserId());
// 构建路由 RouterVo 数组。可用于 Vue 构建管理后台的左边菜单
return AjaxResult.success(menuService.buildMenus(menus));
}
具体的代码，比较简单，胖友自己去阅读下，嘿嘿。
通过调用该 /getRouters 接口，前端就可以构建管理后台的左边菜单。

7.10 权限管理
如下的 Controller ，提供了 RuoYi-Vue 的权限管理功能，比较简单，胖友自己去瞅瞅即可。

用户管理 SysUserController ：用户是系统操作者，该功能主要完成系统用户配置。
角色管理 SysRoleController ：角色菜单权限分配、设置角色按机构进行数据范围权限划分。
菜单管理 SysMenuController ：配置系统菜单，操作权限，按钮权限标识等。
7.11 小小的建议
至此，我们完成了对 RuoYi-Vue 权限相关功能的源码进行解读，希望对胖友有一定的胖友。如果胖友项目中需要权限相关的功能，建议不要直接拷贝 RuoYi-Vue 的代码，而是按照自己的理解，一点点“重新”实现一遍。在这个过程中，我们会有更加深刻的理解，甚至会有自己的一些小创新。

666. 彩蛋
     相对还是比较良心的一篇文章，胖友你说是不是，嘿嘿。

这里额外在推荐一些 RabbitMQ 不错的内容：

《Spring Security 实现原理与源码解析系统 —— 精品合集》
《如何设计权限管理模块（附表结构）？》
不过艿艿实际项目中，并未采用 Spring Security 或是 Shiro 安全框架，而是自己团队开发了一个相对轻量级的组件。主要考虑，目前前后端分离之后，Spring Security 内置的很多功能，已经不太需要，在加上拓展一些功能不是非常方便，有点“曲折”，所以才选择自己开发。

Spring Boot
PREVIOUS:
Feign 官方文档翻译
NEXT:
芋道 Spring Security OAuth2 入门
博主【芋艿】在看的课程

【老牛逼了】Dubbo 源码解析系列
Netty 源码解析系列
Spring 源码解析系列
Spring MVC 源码解析系列
Spring Boot 源码解析系列
MyBatis 源码解析系列
数据库实体设计合集
【老牛逼了】Java 面试题
Spring Boot 学习路线
Spring Cloud 学习路线

微信公众号福利：芋道源码

0. 阅读源码葵花宝典
1. RocketMQ / MyCAT / Sharding-JDBC 详细中文注释源码
2. 您对于源码的疑问每条留言都将得到认真回复
3. 新的源码解析文章实时收到通知，每周六十点更新
4. 认真的源码交流微信群
   分类

APISIX1
ActiveMQ2
Apollo35
CAT1
Canal7
Elastic-Job-Cloud6
Elastic-Job-Lite16
Elasticsearch3
Eureka24
Fescar5
Guava13
HikariCP21
Hmily1
Hystrix12
IDEA4
JDK 源码5
JUC36
JVM1
Java 面试12
Jenkins1
Jetty6
Kafka27
Kong1
Maven1
MongoDB3
MyBatis7
MyCAT9
Nacos13
Netty11
Nginx3
NodeJS2
Onemall1
Prometheus1
RabbitMQ2
Redis3
Resilience4j12
Ribbon9
RocketMQ29
RxJava7
SOFA Mosn1
SOFA RPC4
Seata6
Sentinel17
Sentry1
Sharding Sphere6
Sharding-JDBC19
Shiro8
SkyWalking42
Solr1
Soul1
Spring24
Spring Boot99
Spring Cloud33
Spring Security34
Spring Session1
Spring Webflux8
Spring-Cloud-Config1
Spring-Cloud-Gateway26
Spring-MVC13
TCC-Transaction7
Tomcat18
XXL-JOB1
Yudao2
Zipkin11
Zookeeper2
Zuul8
书单33
工作内推4
性能测试9
数据结构与算法4
电商开源项目2
精进4791
芋道源码的周八20
设计模式25
鸡汤7

© 2023 芋道源码 && 总访客数 次 && 总访问量 次 && Hosted by Coding Pages
沪ICP备17037075号-1
