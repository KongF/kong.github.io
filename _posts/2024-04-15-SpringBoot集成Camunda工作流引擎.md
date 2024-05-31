# 版本对照：
|Spring Boot starter|Camunda 7 版本|Spring Boot 版本|
|-|-|-|
|1.0.0*	|7.3.0	|1.2.5.RELEASE|
|1.1.0*	|7.4.0	|1.3.1.RELEASE|
|1.2.0*	|7.5.0	|1.3.5.RELEASE|
|1.2.1*	|7.5.0	|1.3.6.RELEASE|
|1.3.0*	|7.5.0	|1.3.7.RELEASE|
|2.0.0**	|7.6.0	|1.4.2.RELEASE|
|2.1.x**	|7.6.0	|1.5.3.RELEASE|
|2.2.x**	|7.7.0	|1.5.6.RELEASE|
|2.3.x	|7.8.0	|1.5.8.RELEASE|
|3.0.x	|7.9.0	|2.0.x.RELEASE|
|3.1.x	|7.10.0	|2.0.x.RELEASE|
|3.2.x	|7.10.0	|2.1.x.RELEASE|
|3.3.1+	|7.11.0	|2.1.x.RELEASE|
|3.4.x	|7.12.0	|2.2.x.RELEASE|
![image.png](tycy-doc-wiki/common/file?uuid=a6d1823861694e82873a5ef55fac5864)

pom.xml

```xml
 <!-- 工作流相关 -->
        <dependency>
            <groupId>org.camunda.bpm.springboot</groupId>
            <artifactId>camunda-bpm-spring-boot-starter-webapp</artifactId>
            <version>${camunda.spring-boot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.camunda.bpm.springboot</groupId>
            <artifactId>camunda-bpm-spring-boot-starter-rest</artifactId>
            <version>${camunda.spring-boot.version}</version>
        </dependency>

```
yaml配置
```yaml
# Camunda配置
camunda:
    bpm:
        webapp:
            application-path: /workflow #Camunda web端管理
        database:
            type:  mysql
            schema-update: true
        filter:
            create: All tasks
        auto-deployment-enabled: true # 自动部署 resources 下的 bpmn文件
        deployment-resource-pattern: classpath*:bpm/*.bpmn
        admin-user:
            id: admin
            password: ktadmin
```
启动项目后，Camunda会自动在数据库中初始化建表。