# 配置
- 基本上是固定写法，注意修改packages-to-sc
``` yaml
springdoc:  
  default-flat-param-object: true  
  swagger-ui:  
    path: /swagger-ui.html  
    tags-sorter: alpha  
    operations-sorter: alpha  
  api-docs:  
    path: /v3/api-docs  
  group-configs:  
    - group: 'default'  
      paths-to-match: '/**'  
      packages-to-scan: org.klsa.easyCoupon
```
# 配置类
``` Java
@Slf4j  
@Configuration  
public class SwaggerConfiguration implements ApplicationRunner {  
  
    @Value("${server.port:8080}")  
    private String serverPort;  
    @Value("${server.servlet.context-path:}")  
    private String contextPath;  
  
    /**  
     * 自定义 openAPI 个性化信息  
     */  
    @Bean  
    public OpenAPI openAPI() {  
        return new OpenAPI()  
                .info(new Info() // 基本信息配置  
                .title("易券-商家后台管理系统") // 标题  
                .description("创建优惠券、店家查看以及管理优惠券、创建优惠券发放批次等") // 描述 Api 接口文档的基本信息  
                .version("v1.0.0") // 版本  
                // 设置 OpenAPI 文档的联系信息，包括联系人姓名为"Qifei"，邮箱为"2935790378@qq.com"  
                .contact(new Contact().name("Qifei").email("2935790378@qq.com"))  
                );  
    }  
  
  
    @Override  
    public void run(ApplicationArguments args) throws Exception {  
        log.info("API Document: http://127.0.0.1:{}{}/doc.html", serverPort, contextPath);  
    }  
}
```