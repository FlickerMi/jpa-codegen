Table of Contents
=================

   * [简介](#简介)
   * [SpringBoot使用示例](#springboot使用示例)
   * [如何使用](#如何使用)
      * [导入仓库](#导入仓库)
      * [配置代码生成器](#配置代码生成器)
         * [配置文件](#配置文件)
      * [编写代码模板](#编写代码模板)
      * [编写生成器入口](#编写生成器入口)
   * [如何编写模板？](#如何编写模板)
      * [基本信息](#基本信息)
      * [实体信息](#实体信息)
      * [自定义配置](#自定义配置)
   * [什么是模块？](#什么是模块)
   * [Plans](#plans)

[![Build Status](https://travis-ci.org/gcdd1993/jpa-codegen.svg?branch=master)](https://travis-ci.org/gcdd1993/jpa-codegen)

# 简介

这是一款基于`Freemarker`模板驱动的代码生成器。

依据现有的实体类代码，自动生成CRUD代码，解放双手，加快开发速度。

生成的代码包括但不仅限于（可以自定义生成模块）

- Form表单代码
- Repository代码
- Service代码
- Controller代码

# SpringBoot使用示例

克隆[示例项目](https://github.com/gcdd1993/jpa-codegen-sample)，体会解放双手的美妙感受！

# 如何使用

## 导入仓库

```groovy
maven {
    url 'https://dl.bintray.com/gcdd1993/maven'
}
dependencies {
    // jpa code generator
    testCompile 'io.github.gcdd1993:jpa-codegen:v1.0.2'
    testCompile 'org.freemarker:freemarker:2.3.28'
}
```

## 配置代码生成器

### 配置文件

```properties
## 作者
author=gcdd1993
## 代码注释
comments=code generated by jpa-codegen
## 是否覆盖原文件，除非特殊情况，不然请不要覆盖
cover=false
## 代码模板目录
template.dir=src/test/resources/template/
## 实体类包名 Deprecated从v1.0.1开始从配置文件中移除
- entity.package=com.maxtropy.sample.entity
## 实体类标识符
entity.flag=entity
## 以下配置是模块配置(格式 模块名.配置名)，必须在模板目录下提供与模块名相同的模板
## 生成的代码后缀
repository.suffix=Repository
## 模板名称
repository.template=repository.ftl
## 模块标识符
repository.flag=entity.repo

service.suffix=Service
service.template=service.ftl
service.flag=service
form.suffix=Form
form.template=form.ftl
form.flag=form
controller.suffix=Controller
controller.template=controller.ftl
controller.flag=web
```

其中

```properties
repository.suffix=Repository
repository.template=repository.ftl
repository.flag=entity.repo
```

是模块配置，[什么是模块？](#什么是模块？)

## 编写代码模板

模板主要基于`Freemarker`，如`Spring Boot2.x`代码模板可以像下面这样

```java
package ${packageName};

import ${entity.packageName}.${entity.className};
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.querydsl.QuerydslPredicateExecutor;
<#list imports as import>
import ${import};
</#list>

/**
 * repository for ${entity.className} generated by jpa-codegen
 * ${comments}
 *
 * @author ${author}
 * Created On ${date}.
 */
public interface ${className} extends JpaRepository<${entity.className}, ${entity.id.className}>, QuerydslPredicateExecutor<${entity.className}> {
}
```

- [Spring Boot 2.x模板](<https://github.com/gcdd1993/jpa-codegen/tree/master/src/main/resources/template/spring2>)

- [如何编写模板?](#如何编写模板)

## 编写生成器入口

在test模块中编写生成器入口，如

```java
public class Codegen {

    @Test
    public void generate() {
        new CodeGenerator("src/test/resources/codegen.properties")
                .registerRender("repository")
                .generate();
    }
    
}
```

然后运行`generate()`，在项目目录下将会生成

![Imgur](https://i.imgur.com/WTg4qMx.png)

![Imgur](https://i.imgur.com/yJJ1d59.png)

生成的代码完全由模板以及实体类信息决定。

# 如何编写模板？

模板完全基于`FreeMarker`以及实体类信息，`FreeMarker`参考[FreeMarker Docs](<https://freemarker.apache.org/docs/index.html>)

支持的元素定义如下

## 基本信息

| Freemarker元素   | 解释                     | 示例输出                                                 |
| ---------------- | ------------------------ | -------------------------------------------------------- |
| `${ftlName}`     | 模板名称                 | `controller.ftl`                                         |
| `${ftlPath}`     | 模板目录                 | `src/main/resources/template/`                           |
| `${savePath}`    | 保存路径                 | `src/main/resources/io/github/gcdd1993/controller`       |
| `${packageName}` | java文件包名             | `io.github.gcdd1993.controller`                          |
| `${className}`   | java文件类名             | `UserController`                                         |
| `${author}`      | 作者                     | `gaochen`                                                |
| `${date}`        | 创建日期，默认为当前日期 | `2019/6/23`                                              |
| `${comments}`    | 注释信息                 | `generated by jpa-codegen`                               |
| `${imports}`     | java文件引入信息         | `org.springframework.beans.factory.annotation.Autowired` |

## 实体信息

| Freemarker元素                             | 解释                                     | 示例输出           |
| ------------------------------------------ | ---------------------------------------- | ------------------ |
| `${entity.className}`                      | 实体类名，`class.getSimpleName()`        | User               |
| `${entity.packageName}`                    | 实体包名，`class.getPackage().getName()` | io.github.gcdd1993 |
| `${entity.tableName}`                      | 实体表名，`@Table(name="")`              | sys_user           |
| `${entity.id.className}`                   | 实体主键类名，`@Id`注释的字段的类名      | Integer            |
| `${entity.id.packageName}`                 | 实体主键包名，`@Id`注释的字段的包名      | java.lang          |
| `${entity.fields.className}`               | 实体所有字段（只支持基本类型）类名       | String             |
| `${entity.fields.packageName}`             | 实体所有字段（只支持基本类型）包名       | java.lang          |
| `${entity.fields.name}`                    | 实体所有字段（只支持基本类型）属性名     | name               |
| `${entity.fields.annotations.className}`   | 实体所有字段注解的类名                   | Id                 |
| `${entity.fields.annotations.packageName}` | 实体所有字段注解的包名                   | javax.persistence  |

## 自定义配置

除了以上默认的信息之外，可能会有额外的信息需要填入生成的代码中，jpa-codegen提供直接将配置文件中的配置渲染到模板的能力。

例如在配置文件`autogen.properties`写下一行

```properties
custom.additional.comment=this is additional comment
```

在模板中可以使用`${otherParams.additional_comment}`获取到该配置。

要注意的是：自定义配置**使用`custom`开头**，后面的**配置会将.替换为_**作为`FreeMarker`模板的key，例如上述的`additional.comment`使用`${otherParams.additional_comment}`获取。

# 什么是模块？

由于代码千变万化，为了尽可能的做到通用性，[jpa-codegen](<https://github.com/gcdd1993/jpa-codegen>)将每一种类型的代码抽象为模块，每一个模块将使用各自的模板，依照实体信息生成代码。

需要为模板配置一下信息：

- repository.suffix=Repository

模块类名后缀，生成的类名规则由**实体类名+后缀构成**

- repository.template=repository.ftl

模块使用的`Freemarker`模板

- repository.flag=entity.repo

模块标识符，生成的代码包名由**实体类将实体标识符替换为模块标识符**来确认。

如

- 实体包名：`io.github.gcdd1993.entity`
- 实体标识符：`entity`
- 模块标识符：`entity.repo`

则生成的`repository`代码包名为  --> `io.github.gcdd1993.entity.repo`

# Plans

- 编写gradle插件

- 编写maven插件