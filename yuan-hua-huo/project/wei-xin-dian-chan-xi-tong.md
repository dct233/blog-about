## 项目设计

![部署架构](https://raw.githubusercontent.com/chanshiyucx/poi/master/2019/2-1%2B%E9%A1%B9%E7%9B%AE%E8%AE%BE%E8%AE%A1.jpg)

![数据库设计](https://raw.githubusercontent.com/chanshiyucx/poi/master/2019/2-3%2B%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1.jpg)

```sql
create table `product_info` (
  `product_id` varchar(32) not null,
  `product_name` varchar(64) not null comment '商品名称',
  `product_price` decimal(8,2) not null comment '商品单价',
  `product_stock` int not null comment '商品库存',
  `product_description` varchar(64) comment '商品描述',
  `product_icon` varchar(512) comment '商品小图',
  `category_type` int not null comment '类目编号',
  `create_time` timestamp not null default current_timestamp comment '创建时间',
  `update_time` timestamp not null default current_timestamp on update current_timestamp comment '修改时间',
  primary key (`product_id`)
) comment '商品表';

create table `product_category` (
  `category_id` int not null auto_increment,
  `category_name` varchar(64) not null comment '类目名称',
  `category_type` int not null comment '类目编号',
  `create_time` timestamp not null default current_timestamp comment '创建时间',
  `update_time` timestamp not null default current_timestamp on update current_timestamp comment '修改时间',
  primary key (`category_id`),
  unique key `uqe_category_type` (`category_type`)
) comment '类目表';

create table `order_master` (
  `order_id` varchar(32) not null,
  `buyer_name` varchar(32) not null comment '买家名字',
  `buyer_phone` varchar(32) not null comment '买家电话',
  `buyer_address` varchar(128) not null comment '买家地址',
  `buyer_openid` varchar(64) not null comment '买家微信openid',
  `order_amount` decimal(8,2) not null comment '订单金额',
  `order_status` tinyint(3) not null default '0' comment '订单状态',
  `pay_status` tintint(3) not null default '0' comment '支付状态',
  `create_time` timestamp not null default current_timestamp comment '创建时间',
  `update_time` timestamp not null default current_timestamp on update current_timestamp comment '修改时间',
   primary key (`order_id`),
   key `idx_buyer_openid` (`buyer_openid`)
) comment '订单主表';

 create table `order_detail` (
  `detail_id` varchar(32) not null,
  `order_id` varchar(32) not null comment '订单id',
  `product_id` varchar(32) not null comment '商品id',
  `product_name` varchar(64) not null comment '商品名称',
  `product_quantity` int not null comment '商品数量',
  `product_icon` varchar(512) comment '商品小图',
  `create_time` timestamp not null default current_timestamp comment '创建时间',
  `update_time` timestamp not null default current_timestamp on update current_timestamp comment '修改时间',
  primary key (`detail_id`),
  key `idx_order_openid` (`order_id`)
) comment '订单详情表';
```

## 环境搭建

- jdk
- nginx
- mysql
- redis

### 安装 jdk

```bash
# 更新软件源
sudo apt-get update

# 安装openjdk-8-jdk
sudo apt-get install openjdk-8-jdk

# 查看java版本
java -version
```

openjdk 默认安装路径为 `/usr/lib/jvm`，然后设置环境变量：

```bash
# 打开配置文件
vi /etc/profile

# 添加下列环境变量
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
JRE_HOME=$JAVA_HOME/jre
CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASSPATH PATH

# 使环境变量生效
source /etc/profile
```

### 安装 nginx

```bash
# 安装 nginx
sudo apt-get install nginx

# 启动 nginx
sudo /etc/init.d/nginx start
```

启动后访问 `http://localhost/` 检测是否正常。安装完成后各个文件存放位置：

```
配置文件：/etc/nginx
主程序：/usr/sbin/nginx
静态文件：/usr/share/nginx
日志：/var/log/nginx
```

> Linux 系统的配置文件一般放在 `/etc`，日志一般放在 `/var/log`，运行的程序一般放在 `/usr/sbin` 或者 `/usr/bin`。

### 安装 mysql

```bash
# 安装 mysql
sudo apt-get install mysql-server

# 启动 mysql
mysql -u root -pYourNewPassword
```

安装完成后各个文件存放位置：

```
启动脚本：/etc/init.d/mysql
数据库目录：/var/lib/mysql
配置文件：/usr/share/mysql（命令及配置文件），/etc/mysql（如：my.cnf）
相关命令：/usr/bin 和 /usr/sbin
```

## 日志 Logback

使用方式一：

```java
public class LoggerTest {

    private final Logger logger = LoggerFactory.getLogger(LoggerTest.class);

    @Test
    public void test() {
        logger.debug("debug...");
        logger.info("info...");
        logger.error("error...");
    }
}
```

使用方式二，使用 `Lombok` 注解：

首先引入依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

添加 `@Slf4j` 注解：

```java
@Slf4j
public class LoggerTest {
    @Test
    public void test() {
        log.debug("debug...");
        log.info("info...");
        log.error("error...");
    }
}
```

可以在 `application.yml` 中进行简单配置：

```yml
logging:
  pattern:
    console: '%d - %msg%n'
  # path: C:/Users/chanshiyu/Documents/log
  file: C:/Users/chanshiyu/Documents/log/sell.log
  level: debug
```

但更推荐在 `logback-spring.xml` 中配置，可以更加细致地控制日志输出：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
    <!-- 控制台输出配置 -->
    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d - %msg%n</pattern>
        </layout>
    </appender>

    <!-- Info 日志文件配置 -->
    <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤 Error 级别日志-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>Error</level>
            <onMatch>DENY</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <encoder>
            <pattern>%d - %msg%n</pattern>
        </encoder>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 路径 -->
            <fileNamePattern>C:/Users/chanshiyu/Documents/log/sell/info.%d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <!-- Error 日志文件配置 -->
    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤 Info 级别日志-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>Error</level>
        </filter>
        <encoder>
            <pattern>%d - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>C:/Users/chanshiyu/Documents/log/sell/error.%d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <!-- 将上述配置引入并控制日志级别 -->
    <root level="info">
        <appender-ref ref="consoleLog" />
        <appender-ref ref="fileInfoLog" />
        <appender-ref ref="fileErrorLog" />
    </root>
</configuration>
```

## 数据库

引入依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

配置 `application.yml`：

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 1124chanshiyu
    url: jdbc:mysql://127.0.0.1:3306/sell?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
  jpa:
    show-sql: true
```

## 买家端

DAO -> Service -> Controller

## Error

### 001 Table 'sell.hibernate_sequence' doesn't exist. could not read a hi value.

将主键生成策略修改为：

```java
/* 类目 id */
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Integer categoryId;
```