# Personal-Knowledge-Warehouse
自己开发过程中记录一些知识点
# 环境框架搭建
## mybatis-plus配置(使用mysql数据库)
1.引入依赖
 ```
<dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.4.0</version>
        </dependency>
 ```
2.yml中设置数据源 和 注解配合

 ```
 spring:
    datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/数据库名?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone = GMT
        username: root #数据库账号
        password: root #密码
 ```
springboot在启动类上 配置@MapperScan,扫描生成的文件夹或mapper文件夹

注:上述是username,别写成了data-username.且注意要是否引入mysql驱动
```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

**3.代码生成器使用**

1)引入依赖
```
<!--模板引擎,它需要使用,与前端页面使用模板无关 -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.2</version>
</dependency>
<!--代码生成器,mybatis-plus 3.0以后需单独引入-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.0</version>
</dependency>
```
2)使用java代码配置运行
```
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("ty"); //设置作者名称,也就是自己
        gc.setOpen(false);
        gc.setServiceName("%sService");
        gc.setFileOverride(true); //设置生成文件是否覆盖原文件
        //gc.setSwagger2(true); //实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/DeSheng?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone = GMT");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("root");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("mapper"));
        pc.setParent("com"); //这里设置你需要放在哪个包下面
        mpg.setPackageInfo(pc);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        //控制 不生成 controller  空字符串就行
        templateConfig.setController("");
        templateConfig.setService("");
        templateConfig.setServiceImpl("");
        templateConfig.setXml("");
        //删除上述即可生成service
        mpg.setTemplate(templateConfig);
        mpg.setTemplateEngine(new VelocityTemplateEngine());

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        // 写于父类中的公共字段
        strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.execute();
    }
}
```
注:个人配置,使用默认配置只需修改数据源配置和包配置;默认只生成mapper和entity,如需要生成service和controller请看copy官网配置后修改;
运行后该main函数依次输入**生成代码所在文件夹的名称** 以及 ***数据库表名***

## springboot热部署
1.引入依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```
2.yml配置
```
spring:
  devtools:
    restart:
      enabled: true  #设置开启热部署
```
另外用thymeleaf模板可以
```
spring:
  thymeleaf:
    cache: false    #页面不加载缓存，修改即时生效
```
3.尽量开启idea离开页面就自动加载资源和编译

## lomok
1.引入依赖
```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```