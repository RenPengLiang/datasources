## 多读写库切换使用说明及范例：

    本文档着重描述如何在一个系统中访问多个读写数据库,支持随机，循环，权重轮询操作数据库

------

###### 后端范例

* 在子系统pom.xml中添加公共模块的datasource组件pom依赖

  ```java
     <dependency>
       <groupId>com.cjkj</groupId>
       <artifactId>cjkj-datasource-spring-boot-starter</artifactId>
       <version>1.5.0</version>
     </dependency>
  ```
  
* 在子系统配置文件中配置数据库信息包括只读库列表和写入库列表
  * 数据库基本信息包括，jdbcUrl：数据库连接地址，username：用户名，password: 密码, driverClassName: 连接数据库驱动, weight: 轮询权重, toDataSourceKey: 切换数据源key，定义：read1,read2... write1,write2...
    
  ```java
    # 多数据来源配置
    # 轮询方式： pollingWays【CYCLE：循环 WEIGHT：权重 RANDOM：随机】
    # mapper包名：mapperPackage
    # 多数据库列表：baseDbInfoList
    dbs:
       mapperPackage: com.cjkj.mapper
       pollingWays: CYCLE
       baseDbInfoList: [{jdbcUrl: "jdbc:mysql://10.253.96.125:3306/asc_transport?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8", username: asc_user, password: Asc!@#123, driverClassName: com.mysql.cj.jdbc.Driver, weight: 1, toDataSourceKey: read1},
                   {jdbcUrl: "jdbc:mysql://10.253.96.125:3306/asc_trade?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8", username: asc_user, password: Asc!@#123, driverClassName: com.mysql.cj.jdbc.Driver, weight: 2, toDataSourceKey: read2},
                   {jdbcUrl: "jdbc:mysql://10.253.96.125:3306/asc_trade?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8", username: asc_user, password: Asc!@#123, driverClassName: com.mysql.cj.jdbc.Driver, weight: 1, toDataSourceKey: write1}]
  ```

* 在启动类上加上注解@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})

  ```java
  @EnableTransactionManagement
  @MapperScan("com.cjkj.mapper")
  @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
  public class DatasourceApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(DatasourceApplication.class, args);
      }
  
  }
  ```
  组件目前尚存在缺陷，子系统mapper包名目前只能命名成：com.cjkj.mapper
