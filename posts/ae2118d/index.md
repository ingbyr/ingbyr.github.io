# Java8 Spring MVC 5 项目迁移 Java21 SpringBoot 3.5 记录


总结了迁移 JDK8 Spring MVC 5 项目到 JDK21 SpringBoot 3.5 的一些处理方式与步骤。

<!--more-->

## 前置检查

- 检查与HTTP相关的第三方依赖和私有依赖，是否已经有支持Jakarta的版本，没有则需要克隆源码自行修改后发布到私有仓库。
- 检查项目中是否使用了Oracle相关的代码库，删除或使用其他开源库替代。

## 升级依赖

### 修改版本控制

修改pom中的 parent 为 `spring-boot-starter-parent`，将大部分依赖版本号管理委托至 spring boot，
降低不同版本不兼容风险，例如：

```xml

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.8</version>
</parent>
```

添加版本控制后，需要将旧版本依赖的版本号全部删除，执行 `maven dependency:resolve` 查看哪些依赖版本号缺失，
并添加缺失的依赖版本号。

### 替换 spring-boot-starter 系列

一些依赖组件提供了原生的 spring-boot-starter，最好直接使用starter依赖替换旧的版本管理，降低迁移成本，例如：

- spring-boot-starter-web
- spring-boot-starter-test
- spring-boot-starter-validation
- mybatis-spring-boot-starter
- 其他的 starter 可以在官网查看

### 排除重复依赖

有一些无法升级或老旧的第三方包，可能兼容Java21，但自身会带来旧版本Spring或其他的依赖，此时使用命令
`mvn dependency:tree -Dverbose`检查依赖传递情况，找到重复依赖并排除，例如下面需要排除 commons-fileupload，转而使用
新版本 fileupload2：

```xml

<dependencyManagement>
    <dependency>
        <groupId>com.bstek.ureport</groupId>
        <artifactId>ureport2-console</artifactId>
        <version>${ureport.version}</version>
        <exclusions>
            <exclusion>
                <groupId>commons-fileupload</groupId>
                <artifactId>commons-fileupload</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-fileupload2-jakarta-servlet6</artifactId>
        <version>${commons-fileupload.version}</version>
    </dependency>
</dependencyManagement>
```

### 定制化停止更新的依赖

一些老旧的依赖可能在发布 Jakarta 前就已停更，此时需要获取到源码工程，修改源码兼容Java21和Spring Boot 3.5，并发布到私有仓库。
例如本次迁移中项目中使用Ureport2已经停更，且不支持Java21 Jakarta，修改源码大致步骤：

- Ureport源码中添加 spring-boot parent，删除由spring-boot管理的版本号。
- 修改或升级Ureport中的三方依赖。
- 修改源码以兼容新版的 Jakarta命名空间，操作同后续内容中的项目源码修改

### 升级 maven 插件

一些maven插件在Java21环境下也需要升级，可以一并升级到最新版本，这个一般不会有兼容性问题。查看具体的版本传递可以使用插件

```xml

<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>flatten-maven-plugin</artifactId>
    <version>1.7.3</version>
    <configuration>
    </configuration>
    <executions>
        <!-- enable flattening -->
        <execution>
            <id>flatten</id>
            <phase>process-resources</phase>
            <goals>
                <goal>flatten</goal>
            </goals>
        </execution>
        <!-- ensure proper cleanup -->
        <execution>
            <id>flatten.clean</id>
            <goals>
                <goal>clean</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 项目代码适配

### javax 迁移 jakarta 命名空间
### 迁移 spring xml 配置为代码配置方式
### 旧配置文件适配 spring boot 3

---

> 作者: <no value>  
> URL: https://ingbyr.github.io/posts/ae2118d/  

