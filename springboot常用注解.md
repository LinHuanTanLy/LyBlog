[toc]
#  备忘录
## Bean

### 生命周期
[Bean生命周期](https://www.cnblogs.com/xyang/p/12511388.html)
### Bean的创建方式
1. 注解方式
```java
@Configuration
public class PersonBean {

    @Bean
    public String sayHi() {
        return "hi,i am linHuanTan";
    }
}
```
2. @Bean定义方式
```java
@Component
public class PersonBean {

    public String sayHi() {
        return "hi,i am linHuanTan";
    }
}

```

## 注解

### 普通注解

1. **@SpringBootApplication**
申明让spring boot自动给程序进行必要的配置，这个配置等同于：
@Configuration ，@EnableAutoConfiguration 和 @ComponentScan 三个配置

```kotlin
@MapperScan("com.ly.learn01.mapper")
@SpringBootApplication
public class Learn01Application {
    public static void main(String[] args) {
        SpringApplication.run(Learn01Application.class, args);
    }
}
```
***
2. @ResponseBody
表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用，用于构建RESTful的api。在使用@RequestMapping后，返回值通常解析为跳转路径，加上@responsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如异步获取json数据，加上@Responsebody后，会直接返回json数据。该注解一般会配合@RequestMapping一起使用。
***
3. @Controller
用于定义控制器类，在spring项目中由控制器负责将用户发来的URL请求转发到对应的服务接口（service层），一般这个注解在类中，通常方法需要配合注解@RequestMapping。
***
4. @RestController
用于标注控制层组件(如struts中的action)，@ResponseBody和@Controller的合集。
```kotlin
@RestController
@RequestMapping("api/index")
@Api(value = "首页模块")
class IndexController {}
```
***
5. @RequestMapping
提供路由信息，负责URL到Controller中的具体函数的映射。
```kotlin
    @PostMapping("/")
    @ApiOperation("添加banner数据")
    fun addBanner(@RequestBody vo: BannerVo): CommResult<Banner> = CommResult.suc(indexService.addBanner(vo))
```
***
6. **@EnableAutoConfiguration**
SpringBoot自动配置（auto-configuration）：尝试根据你添加的jar依赖自动配置你的Spring应用。例如，如果你的classpath下存在HSQLDB，并且你没有手动配置任何数据库连接beans，那么我们将自动配置一个内存型（in-memory）数据库”。你可以将@EnableAutoConfiguration或者@SpringBootApplication注解添加到一个@Configuration类上来选择自动配置。如果发现应用了你不想要的特定自动配置类，你可以使用@EnableAutoConfiguration注解的排除属性来禁用它们。

 **可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。**
[相关博文](https://blog.csdn.net/zxc123e/article/details/80222967)
***
7. **@ComponentScan**
表示将该类自动发现扫描组件。个人理解相当于，如果扫描到有@Component、@Controller、@Service等这些注解的类，并注册为Bean，可以自动收集所有的Spring组件，包括@Configuration类。我们经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入。可以自动收集所有的Spring组件，包括@Configuration类。我们经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入。如果没有配置的话，Spring Boot会扫描启动类所在包下以及子包下的使用了@Service,@Repository等注解的类。

> Spring是一个依赖注入的框架，所有的内容都是基于bean的定义及其依赖关系。但是他不知道去哪里找到bean，所以 *ComponentScan做的事情就是告诉Spring从哪里找到bean*

1.  springBoot项目
	* 如果你的其他包都在使用了@SpringBootApplication注解的main app所在的包及其下级包，则你什么都不用做，SpringBoot会自动帮你把其他包都扫描了
	* 如果你有一些bean所在的包，不在main app的包及其下级包，那么你需要手动加上@ComponentScan注解并指定那个bean所在的包。

2.  非Spring Boot项目中，我们必须显式地使用@ComponentScan注解定义被扫描的包,可以通过XML文件在应用上下文中定义或在Java代码中对应用上下文定义。


***
8.  **@Configuration**  **@SpringBootConfiguration**
相当于传统的xml配置文件，如果有些第三方库需要用到xml文件，可以使用@ImportResource注解加载xml配置文件，里面包含了@Bean注解的配置项

	注意点：
		1. @Configuration不可以是final类型
		2. @Configuration不可以是匿名类；
		3. 嵌套的configuration必须是静态类。
```kotlin  
@Configuration
@EnableSwagger2
open class SwaggerConf {
    @Bean
    open fun createRestApi(): Docket = Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo())
            .select()            .apis(RequestHandlerSelectors.basePackage("com.ly.learn01.controller"))
            .paths(PathSelectors.any())
            .build()
    private fun apiInfo(): ApiInfo =
            ApiInfoBuilder()
                    .title("Ly接口平台")
                    .contact("Ly")
                    .version("1.0")
                    .build()
}
```
***
9. @Import
用来导入其他配置类


***
10. @ImportResource
用来加载xml配置文件

***
11. **@Autowired**
自动导入依赖的bean

步骤：
1. Spring先去容器中寻找NewsSevice类型的bean（先不扫描newsService字段）
2. 若找不到一个bean，会抛出异常
3. 若找到一个NewsSevice类型的bean，自动匹配，并把bean装配到newsService中
4. 若NewsService类型的bean有多个，则扫描后面newsService字段进行名字匹配，匹配成功后将bean装配到newsService中，配合*@Qualifier*
```kotlin
    @Autowired
    @Qualifier("authenticationSuccessHandler")
    private AuthenticationSuccessHandler successHandler;

	@Service("authenticationSuccessHandler")
	public class AuthenticationSuccessHandler extends SavedRequestAwareAuthenticationSuccessHandler {}
```
***
12. @Service
一般用于修饰service层的组件,可以跟上名字

13. @Repository
使用@Repository注解可以确保DAO或者repositories提供异常转译，这个注解修饰的DAO或者repositories类会被ComponetScan发现并配置，同时也不需要为它们提供XML配置项。

14. @Bean
用@Bean标注方法等价于XML中配置的bean。

15. @Value
注入Spring boot application.properties配置的属性的值。

16. @Inject
等价于默认的@Autowired，只是没有required属性；

17. @Component
泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注

18. @Qualifier
当有多个同一类型的Bean时，可以用@Qualifier(“name”)来指定。与@Autowired配合使用。@Qualifier限定描述符除了能根据名字进行注入，但能进行更细粒度的控制如何选择候选者

19. @Resource(name=”name”,type=”type”)
没有括号内内容的话，默认byName。与@Autowired干类似的事


20. **@Conditional**
这是 Spring 4.0 添加的新注解，用来标识一个 Spring Bean 或者 Configuration 配置文>件，当满足指定的条件才开启配置。


### jpa注解

1. @Entity：@Table(name=”“)
表明这是一个实体类。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

2. @MappedSuperClass
用在确定是父类的entity上。父类的属性子类可以继承

3. @NoRepositoryBean
一般用作父类的repository，有这个注解，spring不会去实例化该repository。

4. @Column
如果字段名与列名相同，则可以省略。

5. @Id
表示该属性为主键。

6. @GeneratedValue(strategy = GenerationType.SEQUENCE,generator = “repair_seq”)
表示主键生成策略是sequence（可以为Auto、IDENTITY、native等，Auto表示可在多个数据库间切换），指定sequence的名字是repair_seq。

7. @SequenceGeneretor(name = “repair_seq”, sequenceName = “seq_repair”, allocationSize = 1)
name为sequence的名称，以便使用，sequenceName为数据库的sequence名称，两个名称可以一致。

8. @Transient
表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。@Basic(fetch=FetchType.LAZY)：标记可以指定实体属性的加载方式

9. @JsonIgnore
作用是json序列化时将Java bean中的一些属性忽略掉,序列化和反序列化都受影响。

10. @JoinColumn
一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。

11. @OneToOne、@OneToMany、@ManyToOne：
对应hibernate配置文件中的一对一，一对多，多对一。



## 常用sql
### 查询
####  去重（DISTINCT ）
```sql
select  DISTINCT cid from article
```
####  模糊插叙
* %  替代 0 个或多个字符
* _   替代一个字符
* REGEXP [charList] char通配符
```sql
#包含Ly,前后无限字数
select title from article where title   like   '%Ly%'
#包含y，前面一个字符，后面无限
select title from article where title   like   '_y%'
#以L开头
select title from article where title REGEXP  '^[L]'
#不以L开头
select title from article where title not REGEXP  '^[L]'

@Select("select * from article where title like  CONCAT('%',#{title},'%')")
public List<Article> selectByTitle(String title);
```
#### UNION
UNION 操作符用于合并两个或多个 SELECT 语句的结果集。UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同；默认选取不同的值（与UNION ALL不一样）;查询结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名
```sql
select id from category union  select cid from article  
```
#### ORDER BY 
DESC 倒序
ASC 正序
```sql
select * from article ORDER BY id ASC
```
#### **连接**
#####  内连接
（组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分）
![enter image description here](https://img-blog.csdn.net/20181005173658980?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqdDk4MDQ1MjQ4Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```sql
inner  join   on
EG:
select boy.hid,boy.bname,girl.gname from boy inner JOIN girl on girl.hid=boy.hid
```
##### 左连接查询 left join
left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。 左(外)连接，左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。
![enter image description here](https://img-blog.csdn.net/20181005211357263?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqdDk4MDQ1MjQ4Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```sql
left join on / left outer join on 
eg:
select boy.hid,boy.bname,girl.gname from boy LEFT JOIN girl on girl.hid=boy.hid
```
##### 右连接查询 right join
right join是right outer join的简写，它的全称是右外连接，是外连接中的一种。与左(外)连接相反，右(外)连接，左表(a_table)只会显示符合搜索条件的记录，而右表(b_table)的记录将会全部表示出来。左表记录不足的地方均为NULL。
![enter image description here](https://img-blog.csdn.net/20181005213457811?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqdDk4MDQ1MjQ4Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```sql
right join on
EG:
select boy.hid,boy.bname,girl.gname from boy RIGHT JOIN girl on girl.hid=boy.hid
```

##### 全连接 union
1. 通过union连接的SQL它们分别单独取出的列数必须相同；
2. 不要求合并的表列名称相同时，以第一个sql 表列名为准
3. 使用union 时，完全相等的行，将会被合并，由于合并比较耗时，一般不直接使用 union 进行合并，而是通常采用union all 进行合并；
4. 被union 连接的sql 子句，单个子句中不用写order by ，因为不会有排序的效果。但可以对最终的结果集进行排序；
```sql
(select option ) union (select option)
(SELECT hid,BNAME FROM BOY  ) UNION  (SELECT HID,gname FROM GIRL )
```
##### 练习

数据库表文件
```sql
/*
 Navicat Premium Data Transfer

 Source Server         : Ly
 Source Server Type    : MySQL
 Source Server Version : 50647
 Source Host           : localhost:3306
 Source Schema         : join_test

 Target Server Type    : MySQL
 Target Server Version : 50647
 File Encoding         : 65001

 Date: 17/03/2020 15:20:12
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for match
-- ----------------------------
DROP TABLE IF EXISTS `match`;
CREATE TABLE `match`  (
  `matchDate` datetime(0) NOT NULL COMMENT '比赛时间\r\n',
  `matchResult` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '比赛结果',
  `hostTeamId` int(10) NOT NULL COMMENT '主队id\r\n',
  `gutstTeamId` int(10) NOT NULL COMMENT '客队id',
  `matchId` int(10) NOT NULL COMMENT '主键',
  PRIMARY KEY (`matchId`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of match
-- ----------------------------
INSERT INTO `match` VALUES ('2006-05-21 14:59:23', '2：0', 1, 2, 1);
INSERT INTO `match` VALUES ('2006-06-21 15:00:11', '1：2', 2, 3, 2);
INSERT INTO `match` VALUES ('2006-06-25 15:00:43', '2：5', 3, 1, 3);
INSERT INTO `match` VALUES ('2006-07-21 15:01:29', '3：2', 2, 1, 4);

-- ----------------------------
-- Table structure for team
-- ----------------------------
DROP TABLE IF EXISTS `team`;
CREATE TABLE `team`  (
  `teamName` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '队伍名',
  `teamId` int(11) NOT NULL,
  PRIMARY KEY (`teamId`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of team
-- ----------------------------
INSERT INTO `team` VALUES ('国安', 1);
INSERT INTO `team` VALUES ('申花', 2);
INSERT INTO `team` VALUES ('恒大', 3);

SET FOREIGN_KEY_CHECKS = 1;
```
1. 请查出 2006-6-1 到2006-7-1之间举行的所有比赛，并且用以下形式列出： 拜仁   2：0  不来梅  2006-6-21
```sql
-- m表左连接t表拿到主队的对战记录
select team.teamName,`match`.matchResult,`match`.matchDate from `match` LEFT JOIN team on team.teamId = `match`.hostTeamId
-- m表连接t表拿到客队的对战记录
select team.teamName,`match`.matchResult,`match`.matchDate from `match` LEFT JOIN team on team.teamId = `match`.gutstTeamId
-- 合并数据
select t1.teamName,t1.matchResult,t2.teamName,t1.matchDate from
(select `match`.matchId, team.teamName,`match`.matchResult,`match`.matchDate from `match` LEFT JOIN team on team.teamId = `match`.hostTeamId) as t1
LEFT JOIN
(select `match`.matchId, team.teamName,`match`.matchResult,`match`.matchDate from `match` LEFT JOIN team on team.teamId = `match`.gutstTeamId) as t2
on 
t1.matchId = t2.matchId 
where t1.matchDate
BETWEEN '2006-6-1' and '2006-7-1'
```

2. 
#### 更新
1. 简单的批量更新（between，in）
```sql
update article set state =2 where id BETWEEN 108 and 202

update article set state=1 where id in (108,109,110,101,102,103)
```




## AOP
> AOP（Aspect Oriented Programming），即面向切面编程，OP采用"横切"的技术，剖解开封装的对象内部，将影响了多个类的公共行为封装到一个可重用模块。将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。简单来说讲，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程

### aop切片log出接口请求前后
```java
@Aspect  //使之成为切面类
@Component //把切面类加入到IOC容器中
@Slf4j
public class BrokerAspect {

    /**
     * execution(* *(..))
     * 匹配所有方法
     * execution(public * com. savage.service.UserService.*(..))
     * 表示匹配com.savage.server.UserService中所有的公有方法
     * "execution(public * com.ly.learn01.controller..*.*(..)))"
     * 匹配controller包里面的所有公共方法
     */
    @Pointcut(value = "execution(public * com.ly.learn01.controller..*.*(..)))")
    public void BrokerAspect() {
    }


    /**
     * 执行之前执行
     */
    @Before("BrokerAspect()")
    public void onBefore() {
        log.info("------------------------before----------------------------------");
    }

    /**
     * 执行之后执行
     */
    @After("BrokerAspect()")
    public void onAfter() {
        log.info("------------------------after----------------------------------");
    }

    /**
     * 返回之后执行
     */
    @AfterReturning("BrokerAspect()")
    public void onAfterReturning() {
        log.info("------------------------afterReturn----------------------------------");
    }

    /**
     * 抛异常的时候执行
     */
    @AfterThrowing("BrokerAspect()")
    public void onAfterThrowing() {
        log.info("------------------------afterThrowing----------------------------------");
    }


//    @Around("BrokerAspect()")
//    public void doAround(ProceedingJoinPoint pjp) throws Throwable {
//        try {
//            log.info("-----doAround---before");
//            pjp.proceed();
//            log.info("-----doAround---after");
//        } catch (Throwable throwable) {
//            log.info("-----Exception---after" + pjp.getSourceLocation().getFileName());
//            throwable.printStackTrace();
//        }
//    }
}
```

