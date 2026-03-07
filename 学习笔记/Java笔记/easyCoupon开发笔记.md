
#### 项目根模块常用pom配置
``` xml
<properties>
    <java.version>17</java.version>
    <spring-boot.version>3.0.7</spring-boot.version>
    <spring-cloud.version>2022.0.3</spring-cloud.version>
    <spring-cloud-alibaba.version>2022.0.0.0-RC2</spring-cloud-alibaba.version>
    <mybatis-spring-boot-starter.version>3.0.2</mybatis-spring-boot-starter.version>
    <shardingsphere.version>5.3.2</shardingsphere.version>
    <jjwt.version>0.9.1</jjwt.version>
    <fastjson2.version>2.0.36</fastjson2.version>
    <mybatis-plus.version>3.5.3.1</mybatis-plus.version>
    <dozer-core.version>6.5.2</dozer-core.version>
    <hutool-all.version>5.8.20</hutool-all.version>
    <redisson.version>3.21.3</redisson.version>
    <guava.version>30.0-jre</guava.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>shardingsphere-jdbc-core</artifactId>
            <version>${shardingsphere.version}</version>
        </dependency>

        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>${jjwt.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
            <version>${fastjson2.version}</version>
        </dependency>

        <dependency>
            <groupId>com.github.dozermapper</groupId>
            <artifactId>dozer-core</artifactId>
            <version>${dozer-core.version}</version>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>${hutool-all.version}</version>
        </dependency>

        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson-spring-boot-starter</artifactId>
            <version>${redisson.version}</version>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>${guava.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>

```
- 在pom文件里的$<packaging>pom</packaging>$意思是不参与打包行为
#### SpringBoot Starter
- 是SpringBoot提供的一种简化配置和以来管理的机制
- 优点
	- 自动配置
	- 简化依赖配置
- 如何定义
	- 创建SpringBoot项目
	- 创建对应的业务代码，比如说全局异常拦截器
	- 定义xxxConfiguration配置类，提供一个@Bean容器，之前的在类上添加注解被扫描已经失效
	- resources路径下创建`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件
	- 文件里写入需要自动装配的配置类 `com.nageoffer.onecoupon.framework.config.WebAutoConfiguration`
#### 异常码
- 定义：使用无参数方法定义错误码和错误信息
```Java
public interface IErrorCode {  
  
    /**  
     * 错误码  
     */  
    String code();  
  
    /**  
     * 错误信息  
     */  
    String message();  
}
```
- 实现：用枚举类实现接口
```Java
public enum BaseErrorCode implements IErrorCode {  
  
    // ========== 一级宏观错误码 客户端错误 ==========    CLIENT_ERROR("A000001", "用户端错误"),  
  
    // ========== 二级宏观错误码 用户注册错误 ==========    USER_REGISTER_ERROR("A000100", "用户注册错误"),  
    USER_NAME_VERIFY_ERROR("A000110", "用户名校验失败"),  
    USER_NAME_EXIST_ERROR("A000111", "用户名已存在"),  
    USER_NAME_SENSITIVE_ERROR("A000112", "用户名包含敏感词"),  
    USER_NAME_SPECIAL_CHARACTER_ERROR("A000113", "用户名包含特殊字符"),  
    PASSWORD_VERIFY_ERROR("A000120", "密码校验失败"),  
    PASSWORD_SHORT_ERROR("A000121", "密码长度不够"),  
    PHONE_VERIFY_ERROR("A000151", "手机格式校验失败"),  
  
    // ========== 二级宏观错误码 系统请求缺少幂等Token ==========  
    IDEMPOTENT_TOKEN_NULL_ERROR("A000200", "幂等Token为空"),  
    IDEMPOTENT_TOKEN_DELETE_ERROR("A000201", "幂等Token已被使用或失效"),  
  
    // ========== 一级宏观错误码 系统执行出错 ==========    SERVICE_ERROR("B000001", "系统执行出错"),  
    // ========== 二级宏观错误码 系统执行超时 ==========    SERVICE_TIMEOUT_ERROR("B000100", "系统执行超时"),  
  
    // ========== 一级宏观错误码 调用第三方服务出错 ==========    REMOTE_ERROR("C000001", "调用第三方服务出错");  
  
    private final String code;  
  
    private final String message;  
  
    BaseErrorCode(String code, String message) {  
        this.code = code;  
        this.message = message;  
    }  
  
    @Override  
    public String code() {  
        return code;  
    }  
  
    @Override  
    public String message() {  
        return message;  
    }  
}
```
#### 全局返回类
- 行业标准，规范化，为了友好地跟前端交流，一共分为四部分，返回码、返回信息、响应数据、请求ID
```Java
@Data  
@Accessors(chain = true)  
public class Result<T> implements Serializable {  
  
    @Serial  
    private static final long serialVersionUID = 114514926L;  
  
    /**  
     * 正确返回码  
     */  
    public static final String SUCCESS_CODE = "0";  
  
    /**  
     * 返回码  
     */  
    private String code;  
  
    /**  
     * 返回消息  
     */  
    private String message;  
  
    /**  
     * 响应数据  
     */  
    private T data;  
  
    /**  
     * 请求ID  
     */   
    private String requestId;  
  
    public boolean isSuccess() {  
        return SUCCESS_CODE.equals(code);  
    }  
}
  
```
- 返回码：表示此次请求处理的状态
- 返回信息：详细描述此次请求
- 响应数据：此次请求需要的数据
- 请求ID：标记是哪次请求
#### 全局异常拦截器
- 定义一个抽象父类，成员有错误码和错误信息
```java
  @Getter  
public abstract class AbstractException extends RuntimeException {  
  
    public final String errorCode;  
  
    public final String errorMessage;  
  
    public AbstractException(String message, Throwable throwable, IErrorCode errorCode) {  
        super(message, throwable);  
        this.errorCode = errorCode.code();  
        this.errorMessage = Optional.ofNullable(StringUtils.hasLength(message) ? message : null).orElse(errorCode.message());  
    }  
}
  ```
  - 客户端异常类定义
  ``` Java
  public class ClientException extends AbstractException {  
  
    public ClientException(IErrorCode errorCode) {  
        this(null, null, errorCode);  
    }  
  
    public ClientException(String message) {  
        this(message, null, BaseErrorCode.CLIENT_ERROR);  
    }  
  
    public ClientException(String message, IErrorCode errorCode) {  
        this(message, null, errorCode);  
    }  
  
    public ClientException(String message, Throwable throwable, IErrorCode errorCode) {  
        super(message, throwable, errorCode);  
    }  
  
    @Override  
    public String toString() {  
        return "ClientException{" +  
                "code='" + errorCode + "'," +  
                "message='" + errorMessage + "'" +  
                '}';  
    }  
}
  ```
#### CommandLineRunner接口和ApplicationContextAware接口
- 实现ApplicationContextAware中的public void setApplicationContext(Application application)方法可以把所有IOC容器里的bean传入application中
- 实现CommandLineRunner中的public void run(String... args)方法，该方法在接受外部请求前执行
- 流程图
  ``` mermaid
  graph TD
    A[Spring应用启动] --> B[Bean实例化]
    B --> C[依赖注入]
    C --> D[@PostConstruct执行]
    D --> E[BeanPostProcessor后处理]
    E --> F[ApplicationContext完全刷新]
    F --> G[CommandLineRunner.run执行]
    G --> H[开始接受外部请求]
  ```
### 关于分库分表
#### 什么场景分表
- 单表数据量过大
- 单表存在较多写入场景，可能引发锁竞争
- 当表中包含大量的TEXT、LONGTEXT和BLOB等大字段
#### 什么场景分库
- 当单个数据库支持的连接数已经不足以满足客户端需求
#### 什么场景分库分表
- 高并发写入场景
- 海量数据场景
#### 分库分表设计
##### 如何选择分片键
- 数据均匀性：分片键应该保证数据的均匀分布在各个分片上，避免出现热点数据集集中在某个分片上的情况
- 业务相关性：分片键应该与业务关联紧密，这样可以避免跨分片查询和跨库事务的复杂性
- 数据不可变：一旦选择了分片键，它应该是不可变的，不能随着业务变化而频繁改变
##### 分库分表算法
- HashMod：通过对分片键进行哈希取模的分片算法
- 时间范围：基于时间范围分片算法

## 开发中遇到的问题
### 1. redistemplate 无法自动注入(无法找到bean)
- 修改maven依赖坐标
### 2. Hash分片不均
- 使用自定义hash