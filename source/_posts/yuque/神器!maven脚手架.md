---
title: '神器:maven脚手架'
urlname: up8qfd
date: '2020-11-30 20:28:00 +0800'
categories:
  - util
permalink: maven-archetype-plugin
tags: []
---

# 前言

项目中为了将功能进行归类, 通常会将具有同种属性的类放在同一个模块中, 这样每次新建项目时其实都是一个依据模板项目修改项目名的重复操作. 这种情况可以使用`maven-archetype-plugin`插件来一次性解决

# 整体流程

1. 依据模板项目生成骨架项目
1. 在骨架中增加 maven 私服地址并修改配置文件
1. 将骨架上传至私服
1. 依据私服中骨架搭建项目

<!--more-->

![](https://cdn.nlark.com/yuque/0/2020/png/178066/1606740813481-127759cc-df1e-40be-bdd4-43f4e7c2c3d1.png#align=left&display=inline&height=323&margin=%5Bobject%20Object%5D&originHeight=323&originWidth=792&size=0&status=done&style=none&width=792)

# 准备工作

1. maven 的`setting.xml`文件中已配置了私服地址及用户名密码
1. 准备一个模板项目大致模块如下

![image.png](https://cdn.nlark.com/yuque/0/2020/png/178066/1606740903170-388cbb29-534e-49cf-8568-e3c920e812d5.png#align=left&display=inline&height=280&margin=%5Bobject%20Object%5D&name=image.png&originHeight=280&originWidth=259&size=12512&status=done&style=none&width=259)

# 依据模板项目生成脚手架

## 模板项目 pom

引入脚手架插件

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-archetype-plugin</artifactId>
  <version>3.0.1</version>
</plugin>
```

## 生成脚手架项目

控制台进入模板项目根路径, 运行 `mvn archetype:create-from-project` 生成脚手架项目
运行成功后, 骨架会生成在 `项目路径/target/generated-sources` 目录下, 名称为  archetype

# 脚手架项目配置

## 配置私服地址(必选)

在骨架项目中找到项目文件中的父 pom ,增加私服配置
`vi target/generated-sources/archetype/pom.xml`

```xml
    <distributionManagement>
        <repository>
            <id>releases</id>
            <url>私服地址</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <url>私服地址</url>
        </snapshotRepository>
    </distributionManagement>
```

## 自定义配置(可选)

项目中可能存在没有在项目根路径下的文件, 比如 `src/main/resources/bootstrap.yml`文件, 这种文件默认是不做替换的, 我们可以在脚手架项目中找到对应文件, 并将其中对应包名替换为 `${package}` ,groupId 替换为`${groupId}` 其他替换方式可在脚手架已生成内容中查找
![image.png](https://cdn.nlark.com/yuque/0/2020/png/178066/1606743413136-c16e26b6-9a95-41e6-949d-4710b3262a0a.png#align=left&display=inline&height=418&margin=%5Bobject%20Object%5D&name=image.png&originHeight=418&originWidth=804&size=72280&status=done&style=none&width=804)
然后修改脚手架项目路径下 `target/generated-sources/archetype/src/main/resources/META-INF/maven/archetype-metadata.xml` 配置文件
先找到需要修改的文件所在模块,在模块中找到对应的文件配置, 在 fileSet 中添加`filtered="true"`属性,使在生成代码时替换该文件中代码
![image.png](https://cdn.nlark.com/yuque/0/2020/png/178066/1606743628543-6ff9d49e-6e76-4a07-9dfe-7da9c6b3c9f2.png#align=left&display=inline&height=648&margin=%5Bobject%20Object%5D&name=image.png&originHeight=648&originWidth=986&size=91865&status=done&style=none&width=986)

# 本地测试

1. 在生成的骨架项目根路径运行`mvn install`进行打包, 可以看到在本地仓库中生成了`模板项目名-archetype` 的骨架项目包
1. 在其他路径下新建文件夹 temp, 我是在 target 目录下新建了 temp 文件夹, 进入 temp 文件夹, 运行

```bash
mvn archetype:generate -DarchetypeArtifactId=ArtifactId -DarchetypeGroupId=GroupId -DarchetypeVersion=版本号 -DarchetypeCatalog=local
```

上述代码需将=后文字替换为骨架项目实际参数

3. 运行项目

# 上传私服

`mvn deploy`

# 使用骨架项目生成代码

## 命令行方式

```bash
mvn archetype:generate -DarchetypeArtifactId=ArtifactId -DarchetypeGroupId=GroupId -DarchetypeVersion=版本号 -DarchetypeCatalog=remote
```

## idea 方式

![image.png](https://cdn.nlark.com/yuque/0/2020/png/178066/1606744343556-a7724b38-b566-4847-86bc-676eaa071343.png#align=left&display=inline&height=141&margin=%5Bobject%20Object%5D&name=image.png&originHeight=141&originWidth=515&size=115655&status=done&style=none&width=515)

![image.png](https://cdn.nlark.com/yuque/0/2020/png/178066/1606744390734-c2b600dc-cac5-4442-89b2-0408df09f61b.png#align=left&display=inline&height=674&margin=%5Bobject%20Object%5D&name=image.png&originHeight=674&originWidth=1153&size=182952&status=done&style=none&width=1153)
点击下一步后输入需要生成的项目名等信息按照步骤进行即可
idea 导入 maven 脚手架项目后, 如果想删除则需在本地缓存中进行删除, 文件路径`Library/Caches/JetBrains/InteliJIdea2020.2/Maven/Indices/UserArchetypes.xml`

# 参考文档

[官方文档](http://maven.apache.org/archetype/maven-archetype-plugin/index.html)
