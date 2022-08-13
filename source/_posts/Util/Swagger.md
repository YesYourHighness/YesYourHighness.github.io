---
title: Swagger
date: 2020-09-06 22:09:20
tags: 
- util
- Swagger
categories: 
- util
- Swagger
---

<center>
前端要接口文档？

NO！

Swagger就够了
</center>

<!-- more -->

# Swagger

前端要接口文档？

NO！

直接甩给他`http://xxx:8080/swagger-ui.html`~

## Swagger

- 号称世界上最流行的API框架
- RestFul Api 文档在线自动生成工具 ，APi与Api定义同步更新
- 直接运行，可以在线测试API接口
- 支持多种语言：Java、PHP...



## 使用Swagger

- 导入依赖：

```xml
<!-- spring swagger2整合 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```



- 设置配置：

需要配置Swagger的bean实例——Docket

```java
@Configuration
@EnableSwagger2
public class Swagger2Config {
    @Bean
    public Docket docket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                // 设置是否开启
                .enable(true)
                .select()
                // 配置要扫描接口的方式
                // basePackage(String path)：指定扫描的包
                // any() 扫描全部
                // none() 不扫描
                // withClassAnnotation() 扫描类上的注解
                // whthMethodAnnotation() 扫描方法上的注解
                .apis(RequestHandlerSelectors.basePackage("com.yunhuo.xcjd.controller"))
                // 过滤
                // ant() 扫描什么下的接口 例如扫描用户
                // any() 扫描所有
                .paths(PathSelectors.ant("/user/**"))
                .build();
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("mylord", "www.yesmylord.cn", "1046467756@qq.com");
        return new ApiInfoBuilder()
                .title("Swagger学习")
                .description("学习Swagger的基本用法等等")
                .contact(contact)
                .version("1.0")
                .build();
    }
}
```

访问`http://localhost:8080/swagger-ui.html`，看到我们配置的结果

<img src="http://img.yesmylord.cn//image-20200906122730541.png" alt="image-20200906122730541" style="width:50%;" />

- 设置为只有`dev`环境才开启swagger

```java
    @Bean
    public Docket docket(Environment environment){
        
        /**********获取项目的生产环境**********************/
        Profiles profiles = Profiles.of("dev");
        boolean env = environment.acceptsProfiles(profiles);
        /********************************************************/
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                // 设置是否开启
                .enable(env)
                .select()
                // 配置要扫描接口的方式
                // basePackage(String path)：指定扫描的包
                // any() 扫描全部
                // none() 不扫描
                // withClassAnnotation() 扫描类上的注解
                // whthMethodAnnotation() 扫描方法上的注解
                .apis(RequestHandlerSelectors.basePackage("com.yunhuo.xcjd.controller"))
                // 过滤
                // ant() 扫描什么下的接口 例如扫描用户
                // any() 扫描所有
                .paths(PathSelectors.ant("/user/**"))
                .build();
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("mylord", "www.yesmylord.cn", "1046467756@qq.com");
        return new ApiInfoBuilder()
                .title("Swagger学习")
                .description("学习Swagger的基本用法等等")
                .contact(contact)
                .version("1.0")
                .build();
    }
```

- 在测试中加上`TOKEN`，在docket中加上

```java
        ParameterBuilder tokenParam = new ParameterBuilder();
        List<Parameter> params = new ArrayList<>();
        tokenParam.name(AnswerConstant.HTTP_HEADER_ANSWER_ACCESS_TOKEN)
                .description("accessToken")
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false)
                .build();
        params.add(tokenParam.build());
```


## 相关注解

使用注解

1. `@Api`
   1. 功能:描述`controller`类
   2. 注解位置：类
   3. 常用注解属性
      1. `tags = "" // 描述此controller类`
2. `@ApiOperation`
   1. 功能：描述一个方法或者一个API接口
   2. 注解位置：方法
   3. 常用注解属性
      1. `value = "" // 描述方法`
      2. `notes = "" // 描述方法详细信息`
3. `@ApiImplicitParam`
   1. 功能：描述**方法或接口参数**
   2. 注解位置：方法
   3. 注解属性
      1. `name = "" // 方法或接口的形参, 注意要与方法的参数名称相同`
      2. `value = "" // 对参数的描述`
      3. `paramType = "" // 参数传递方式，此属性的可选值 ["header", "query", "path", "body", "form"]`
         1. header，使用`@RequestHeader`获取的参数
         2. query，使用`@RequestParam`获取的参数，常用于`GET`请求
         3. path，使用`@PathVariable`获取的参数
         4. body，使用`@RequestBody`获取的参数，常用于`POST`请求，对象参数
      4. `dataType = "" // 参数类型，例如 string, int, ArrayList, POJO类`
4. `@ApiImplicitParams`
   1. 功能：汇集多个参数
   2. 注解位置： 方法
   3. 注解属性
      1. `@ApiImplicitParam`组成的列表
5. `@ApiModel`
   1. 功能：描述Form表单类
   2. 注解位置：form表单类上
6. `@ApiModelProperty`
   1. 功能：描述表单类的各个字段
   2. 注解位置：form表单类上的各个属性



- Controller

```java
@RestController
@Api(tags = "User")
public class userController {

    @ApiOperation(value="hello", notes="无参数GET请求测试")
    @GetMapping("/hello")
    public String hello() {
        return "Hello World!";
    }

    @ApiOperation(value = "helloUser", notes = "单简单参数")
    @ApiImplicitParam(name = "name", value = "姓名", paramType = "query", dataType = "String")
    @GetMapping("/helloUser")
    public String hello2(@RequestParam String name) {
        return "Hello " + name;
    }

    @ApiOperation(value = "hello3", notes = "多简单参数")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "name", value = "用户名", paramType = "query", dataType = "string"),
            @ApiImplicitParam(name = "age", value = "密码", paramType = "query", dataType = "string")
    })
    @GetMapping("/login1")
    public String hello3(@RequestParam String name, @RequestParam Integer age) {
        return "Hello " + name + ", age " + age.toString();
    }

    @ApiOperation(value = "hello4", notes = "传递对象参数")
    @ApiImplicitParam(name = "user", value = "用户", paramType = "body", dataType = "Person")
    @PostMapping("/login2")
    public String hello4(@RequestBody UserForm userForm) {
        return "Hello, " + userForm.getUsername() + " " + userForm.getPassword();
    }

    @ApiOperation(value = "hello5", notes = "传递集合参数")
    @ApiImplicitParam(name = "persons", value = "集合列表", paramType = "body", dataType = "ArrayList<String>")
    @PostMapping("/delete")
    public String hello5(@RequestBody ArrayList<String> users) {
        StringBuilder sb = new StringBuilder();
        users.forEach(t -> sb.append(t).append("\n"));
        return sb.toString();
    }
}
```

- Form

```java
@Data
@ApiModel("用户表单类")
public class UserForm {
    @ApiModelProperty("用户名")
    private String username;
    @ApiModelProperty("密码")
    private String password;
}
```



打开swagger-ui可以看到

![image-20200906215934351](http://img.yesmylord.cn//image-20200906215934351.png)



![image-20200906215950080](http://img.yesmylord.cn//image-20200906215950080.png)