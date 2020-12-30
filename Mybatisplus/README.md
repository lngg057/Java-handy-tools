# MybatisPlus 辅助插件

> 一款基于Mybatis功能的扩展插件，可以实现类似于`JPA`模式的，通过对象操作数据库的形式来进行数据操作，而不用编写sql语句。

## 官网地址

https://baomidou.com

## 代码托管地址

**[Gitee](https://gitee.com/baomidou/mybatis-plus)**| **[Github](https://github.com/baomidou/mybatis-plus)**

## 插件简介

MybatisPlus是一款非常好用且便捷的Mybatis扩展插件，经常荣登Java插件使用热门排行榜TOP1。

MybatisPlus属于侵入度极低的Mybatis增强插件，引入后既不会产生过大的损耗，也不会改变现有的相关逻辑。也就是说从Maven中引入该项目后，基本不会对现有的项目框架、逻辑以及性能产生太大的影响。

MybatisPlus的基本使用规则是，通过对象调用的形式来进行增删改查等数据库操作。

同时插件内置已经定义好的Mapper和Service类对象，里面包含对应层级的常规单表操作，比方说：单表的插入、根据主键的更新、根据查询条件的更新、批量新增更新操作、根据查询条件的列表以及分页查询等。而以上的这些仅仅需要少量的配置即可实现。

MybatisPlus内置有条件查询器，并且支持Lamda的调用方式，可以非常方便的通多对象调用的方法，来构造想要的查询条件，而无需将查询参数从Controller层一路传到Mapper，也不需要担心写错或者拼错查询SQL字段。

插件支持自动生成主键，也支持数据库默认自增。MybatisPlus内置多种主键生成策略，配置后即可以在插入操作时自动生成主键ID，并更新到数据表中。自带的生成主键规则有雪花算法和UUID。自带的数据库主键策略有DB2KeyGenerator，PostgreKeyGenerator，OracleKeyGenerator，H2KeyGenerator等，如果内置支持不满足你的需求，也可通过调用实现IKeyGenerator接口来进行扩展。

## 特色功能

1. ### 自动分页插件

    MybatisPlus内置一款相对来说比较好用的分页插件` PaginationInterceptor`，可以通过 spring xml的形式或者springboot的 @Configuration形式来配置导入。

    该分页插件可以做些简单的参数配置，包括请求越界之后的操作，最大单页请求数量限制等。配置完毕后，可以直接在方法调用的参数中传入IPage对象即可实现自动分页的操作。

2. ### 自动填充功能

    在设计表结构时，通常会有一些表通用字段，比方说` createTime`， ` updateTime`  ，`createUser` ，`updateUser` 等。

    这些字段如果不进行通用处理的话，基本会零散分布在项目的各个代码中，并且会经常会缺失操作某几个字段，特别是在多个人编写代码，新增功能或者修复遗留问题的时候尤为明显。

    MybatisPlus内置提供**自动填充功能**，并且使用也很简单。

    1. 第一步需要自己编写@Component，并实现` MetaObjectHandler`接口的引用，并重写其对应的`insertFil`l和`updateFill`方法即可（这两个方法，一个是在你执行插入，一个是你在执行更新操作时，自定义的数据赋值操作）。

        ```java
        @Slf4j
        @Component
        public class MyMetaObjectHandler implements MetaObjectHandler {
        
            @Override
            public void insertFill(MetaObject metaObject) {
                log.info("start insert fill ....");
                this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now()); // 起始版本 3.3.0(推荐使用)
                // 或者
                this.strictInsertFill(metaObject, "createTime", () -> LocalDateTime.now(), LocalDateTime.class); // 起始版本 3.3.3(推荐)
                // 或者
                this.fillStrategy(metaObject, "createTime", LocalDateTime.now()); // 也可以使用(3.3.0 该方法有bug)
            }
        
            @Override
            public void updateFill(MetaObject metaObject) {
                log.info("start update fill ....");
                this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now()); // 起始版本 3.3.0(推荐)
                // 或者
                this.strictUpdateFill(metaObject, "updateTime", () -> LocalDateTime.now(), LocalDateTime.class); // 起始版本 3.3.3(推荐)
                // 或者
                this.fillStrategy(metaObject, "updateTime", LocalDateTime.now()); // 也可以使用(3.3.0 该方法有bug)
            }
        }
        ```

    2. 第二步需要将对应数据实体进行自动填充的对象添加注解` @TableField(.. fill = FieldFill.INSERT)`，具体的fill 类型，则需要根据该字段需要在什么时候被操作来决定。

        ```java
        public class User {
            // 注意！这里需要标记为填充字段
            @TableField(.. fill = FieldFill.INSERT)
            private String fillField;
            ....
        }
        ```

    3. 具体的实现方案可以参考官方文档和demo描述。

3. ### 逻辑删除功能

    对于某些系统来说，为了方便数据恢复和保护数据本身价值，在实际进行数据库删除操作的时候，并不会真正的对这些数据进行物理磁盘删除，而是通过特殊的指定字段，来进行逻辑删除。

    MybatisPlus可以支持全局配置对应的逻辑删除字段，来达到一处配置，处处使用的效果。配置方法如下

    1. 在对应的yml或者properties配置文件中添加` logic-delete-field`配置字段，来指定数据库中的哪个字段映射为逻辑删除标志字段，并根据` logic-delete-value` 和` logic-not-delete-value` 来指定他对应逻辑删除和不删除的值。
    2. 在实体类字段上加上`@TableLogic`注解。

    > **说明**:
    >
    > 逻辑删除只对**自动注入**的sql起效，换句话说，就是自己编写的SQL不会执行逻辑删除相关操作，这时候需要自行进行相关操作:
    >
    > - 插入: 不作限制
    > - 查找: 追加where条件过滤掉已删除数据,且使用 wrapper.entity 生成的where条件会忽略该字段
    > - 更新: 追加where条件防止更新到已删除数据,且使用 wrapper.entity 生成的where条件会忽略该字段
    > - 删除: 转变为 更新
    >
    > 例如:
    >
    > - 删除: `update user set deleted=1 where id = 1 and deleted=0`
    > - 查找: `select id,name,deleted from user where deleted=0`
    >
    > 字段类型支持说明:
    >
    > - 支持所有数据类型(推荐使用 `Integer`,`Boolean`,`LocalDateTime`)
    > - 如果数据库字段使用`datetime`,逻辑未删除值和已删除值支持配置为字符串`null`,另一个值支持配置为函数来获取值如`now()`

4. ### 动态表名插件

    插件的使用场景：在进行一些简单分区的表结构中（比方说业务上将用户表按照一定的规则拆分成多张表等），可以通过该插件，在不修改顶层通用的实现逻辑和对外表象的基础上，进行指定数据库表的增删改查操作。

    具体实现方案参考：👉 [mybatis-plus-sample-dynamic-tablename](https://gitee.com/baomidou/mybatis-plus-samples/tree/master/mybatis-plus-sample-dynamic-tablename)

    > **注意事项**：
    >
    > - 底层实现原理为解析替换设定表名为处理器的返回表名，表名建议可以定义复杂一些避免误替换
    > - 例如：真实表名为 user 设定为 mp_dt_user 处理器替换为 user_2019 等

5. ### 多租户插件

    具体实现方案参考：👉 [mybatis-plus-sample-tenant](https://gitee.com/baomidou/mybatis-plus-samples/tree/master/mybatis-plus-sample-tenant)

    租户概念是一个独立的概念，当系统需要对同一个模块的数据采用区域隔离的方式来访问时，就可以采用租户来实现对应功能。

    MybatisPlus自带多租户插件，启用多租户插件后所有执行的method的sql都会进行处理，来按照对应的租户ID进行数据过滤。

    下面提供一种可能性的配置多租户的方案。

    1. 添加相关依赖项。

        ```xml
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.0</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-extension</artifactId>
            <version>3.4.0</version>
        </dependency>
        ```

        

    2. 租户配置。

        ```java
        import lombok.Getter;
        import lombok.Setter;
        import org.springframework.boot.context.properties.ConfigurationProperties;
        import org.springframework.cloud.context.config.annotation.RefreshScope;
        
        import java.util.ArrayList;
        import java.util.Arrays;
        import java.util.List;
        
        /**
         * 租户属性
         */
        @Getter
        @Setter
        @RefreshScope
        @ConfigurationProperties(prefix = "mate.tenant")
        public class TenantProperties {
        
            /**
             * 是否开启租户模式
             */
            private Boolean enable = true;
        
            /**
             * 需要排除的多租户的表
             */
            private List<String> ignoreTables = Arrays.asList("mate_sys_user", "mate_sys_depart", "mate_sys_role", "mate_sys_tenant", "mate_sys_role_permission");
        
            /**
             * 多租户字段名称
             */
            private String column = "tenant_id";
        
            /**
             * 排除不进行租户隔离的sql
             */
            private List<String> ignoreSqls = new ArrayList<>();
        }
        ```
        
        
        
3. MybatisPlus配置。
    
    ```java
        import com.baomidou.mybatisplus.extension.plugins.handler.TenantLineHandler;
    import com.baomidou.mybatisplus.extension.plugins.inner.TenantLineInnerInterceptor;
        import lombok.AllArgsConstructor;
        import net.sf.jsqlparser.expression.Expression;
        import net.sf.jsqlparser.expression.NullValue;
        import net.sf.jsqlparser.expression.StringValue;
        import org.springframework.boot.autoconfigure.AutoConfigureBefore;
        import org.springframework.boot.context.properties.EnableConfigurationProperties;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        import vip.mate.core.common.context.TenantContextHolder;
        import vip.mate.core.database.props.TenantProperties;
        
        /**
         * 多租户配置中心
         */
        @Configuration
        @AllArgsConstructor
        @AutoConfigureBefore(MybatisPlusConfiguration.class)
        @EnableConfigurationProperties(TenantProperties.class)
        public class TenantConfiguration {
        
            private final TenantProperties tenantProperties;
        
            /**
             * 新多租户插件配置,一缓和二缓遵循mybatis的规则,
             * 需要设置 MybatisConfiguration#useDeprecatedExecutor = false
             * 避免缓存万一出现问题
             * @return
             */
            @Bean
            public TenantLineInnerInterceptor tenantLineInnerInterceptor(){
                return new TenantLineInnerInterceptor(new TenantLineHandler() {
                    /**
                     * 获取租户ID
                     * @return
                     */
                    @Override
                    public Expression getTenantId() {
                        String tenant = TenantContextHolder.getTenantId();
                        if (tenant != null) {
                            return new StringValue(TenantContextHolder.getTenantId());
                        }
                        return new NullValue();
                    }
        
                    /**
                     * 获取多租户的字段名
                     * @return String
                     */
                    @Override
                    public String getTenantIdColumn() {
                        return tenantProperties.getColumn();
                    }
        
                    /**
                     * 过滤不需要根据租户隔离的表
                     * 这是 default 方法,默认返回 false 表示所有表都需要拼多租户条件
                     * @param tableName 表名
                     */
                    @Override
                    public boolean ignoreTable(String tableName) {
                        return tenantProperties.getIgnoreTables().stream().anyMatch(
                                (t) -> t.equalsIgnoreCase(tableName)
                        );
                    }
                });
            }
        }
        ```
        
        
    
6. ### 乐观锁配置

    首先简单说明一下MybatisPlus关于乐观锁的实现，关于乐观锁的原理这里暂不赘述。

    MybatisPlus要实现乐观锁的前提是在数据库添加对应的版本` version` 字段，并在对应的Java实体对象属性字段上添加`@Version`注解。

    之后MybatisPlus在操作更新该表的时候就会进行乐观锁判断，从而避免级联更新的问题。

    具体的操作逻辑如下：

    - 取出记录时，获取当前数据库的version字段值
    - 更新时，储存该version值
    - 执行更新时， set version = newVersion where version = oldVersion
    - 如果在上一步执行更新操作的时候，version不对，则说明该条记录已经被其他进程更新过了，则该条更新失败

7. ### 性能分析插件

    MybatisPlus内置的性能分析插件，可在控制台输出实际执行的完整Sql语句以及该语句具体的执行时间。在测试时启用该功能，能有效解决慢查询。

    **注意**：该功能比较严重影响项目跑起来的性能，所以在线上环境发布的时候建议直接关闭该性能分析插件，只在测试环境使用。

    插件的使用依赖 `p6spy` 组件，使用前需要添加p6spy 依赖

    ```xml
    <dependency>
      <groupId>p6spy</groupId>
      <artifactId>p6spy</artifactId>
      <version>最新版本</version>
    </dependency>
    ```

    同时配置添加application.yml文件即可使用

    ```yaml
    spring:
      datasource:
        driver-class-name: com.p6spy.engine.spy.P6SpyDriver
        url: jdbc:p6spy:h2:mem:test
    ```



## 有瑕疵的地方

1. 大部分情况下只是针对单表查询的操作。在面对跨多张表进行级联查询，映射多个对象的时候，该插件就基本毫无发挥的余地，这时就只能回到最原始的Mybatis操作，进行Mapper的自定义SQL编写和多对象数据的绑定。

2. 分页插件不能很好地处理多个` left join` 操作。在自定义编写执行SQL时，如果SQL 中包含多级` left join` 操作，而没有按照要求的格式编写语句（比方说没有给查询关联的表或者字段设置别名等），则会出现**分页查询的结果和总数不匹配**的情况 。

    原因是因为MybatisPlus的分页插件在生成 countSql 时，会在 `left join` 的表不参与 `where` 条件的情况下,把 `left join` 优化掉。所以这里也是个坑。

3. 多租户插件也是有一样类似的问题，如果编写的sql涉及到级联多张表，特别是有`inner join`查询的时候，会发生数据查询不完整的情况发生。所以需要在编写sql的时候对每张关联的表设置别名， inner join 的要写标准的 inner join。

## 简单总结

实际用来很舒服的一款插件。

不管是Mapper还是Service层，只要引用了MybatisPlus即可以方便快速的写出直观漂亮的CRUD方法。

同时内置的很多插件也极大地提升了开发效率，能有效的推进项目进度开发。

实际推荐度：⭐️⭐️⭐️⭐️⭐️