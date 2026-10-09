---
title: Gradle多模块项目配置
date: 2024-02-04 16:04:12
keyword:
  - Gradle多模块项目配置
  - Gradle multi module config
  - Gradle dependency management
  - Gradle Spring Dependency Management Plugin
  - Gradle spring-boot-dependencies
tags:
  - Java
  - Spring
  - SpringBoot
  - Gradle
categories: 
  - ["Java", "Gradle"]
---

使用Gradle快速创建一个多模块工程，并引入`io.spring.dependency-management`进行`dependencyManagement`管理。

为子模块定义公共依赖的版本，类似于Maven中的`DependencyManagement`一样。子模块只需引入`groupId`和`artifactId`即可。

<!--more-->
## 工程目录
````shell
├──gradle-study
│  ├──settings.gradle.kts
│  ├──build.gradle.kts
│  ├──gradle-study-common
│  │  ├──build.gradle.kts
│  │  ├──src
│  │  │  ├──main
│  │  │  │  ├──com/piggsoft/gradle/study
│  │  │  │  │  ├──StringUtils.java
│  ├──gradle-study-spring-boot
│  │  ├──build.gradle.kts
│  │  ├──src
│  │  │  ├──main
│  │  │  │  ├──com/piggsoft/gradle/study/spring/boot
│  │  │  │  │  ├──StudyApplication.java
│  │  │  │  ├──com/piggsoft/gradle/study/spring/boot/vo
│  │  │  │  │  ├──User.java
│  │  │  │  ├──com/piggsoft/gradle/study/spring/boot/controller
│  │  │  │  │  ├──HomeController.java
````

## 各文件内容

### gradle-study

#### `settings.gradle.kts`

````kotlin

rootProject.name = "gradle-study"

include("gradle-study-spring-boot")
include("gradle-study-common")
````

#### `build.gradle.kts`

````kotlin
import org.springframework.boot.gradle.plugin.SpringBootPlugin

plugins {
    java
    idea
    id("io.spring.dependency-management") version "1.1.4"
    id("org.springframework.boot") version "2.7.6" apply false
}


subprojects {

    group = "com.piggsoft"
    version = "1.0-SNAPSHOT"

    apply {
        plugin("java")
        plugin("idea")
        plugin("io.spring.dependency-management")
    }

    dependencyManagement {
        imports {
            mavenBom(SpringBootPlugin.BOM_COORDINATES)
        }
    }

    repositories {
        mavenCentral()
    }

    java {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }



    dependencies {
        compileOnly("org.projectlombok:lombok")
        annotationProcessor("org.projectlombok:lombok")
        testImplementation("org.junit.jupiter:junit-jupiter")
    }

    tasks.test {
        useJUnitPlatform()
    }

}
tasks.withType<Test> {
    useJUnitPlatform()
}
````
### gradle-study-common

#### build.gradle.kts

````kotlin
dependencies {
    implementation("org.apache.commons:commons-lang3")
}
````

#### StringUtils.java
````java
package com.piggsoft.gradle.study;

public class StringUtils {

    public static boolean hasText(String s) {
        return org.apache.commons.lang3.StringUtils.isNotEmpty(s);
    }


}
````

### gradle-study-spring-boot

#### build.gradle.kts

````kotlin
plugins {
    id("org.springframework.boot")
}


configurations {
    compileOnly {
        extendsFrom(configurations.annotationProcessor.get())
    }
}


dependencies {
    implementation(project(":gradle-study-common"))


    implementation("org.springframework.boot:spring-boot-starter-web")
    annotationProcessor("org.springframework.boot:spring-boot-configuration-processor")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}
````

#### User.java

````java
package com.piggsoft.gradle.study.spring.boot.vo;

import lombok.Data;

@Data
public class User {

    private String id = "1";
    private String name = "user";

}
````


#### HomeController.java

````java
package com.piggsoft.gradle.study.spring.boot.controller;

import com.piggsoft.gradle.study.StringUtils;
import com.piggsoft.gradle.study.spring.boot.vo.User;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RequestMapping("/home")
@RestController
public class HomeController {

    @RequestMapping
    public User home() {
        StringUtils.hasText("123");
        return new User();
    }

}
````

#### StudyApplication.java

````java
package com.piggsoft.gradle.study.spring.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudyApplication.class, args);
    }
}
````

## 重点配置文件解释

### gradle-study/build.gradle.kts

作为父工程的配置文件，主要和maven的父pom左右类似，进行基础配置定义。例如

引入插件，在引入插件时会默认apply，如果父工程不需要这些插件，加上`apply false`。这个样子module可以直接引用这个plugin而不需要再次定义版本。

`mavenBom(SpringBootPlugin.BOM_COORDINATES)`中的版本是和`io.spring.dependency-management`定义的同源的版本，如果需要升级版本，可以修改。

`subprojects`是gradle的一个配置，从字面上理解是为了所有子项目定义的配置。还有一个是`allprojects`，是为整个目录树中所有的项目配置，这个demo不需要，有需要的可自己尝试定义。

### gradle-study-common/build.gradle.kts

`org.apache.commons:commons-lang3`的版本在`org.springframework.boot:spring-boot-dependencies`已经有定义了，且我们使用插件`io.spring.dependency-management`进行了`import bom`之后我们可以直接使用。

### gradle-study-spring-boot/build.gradle.kts

`org.springframework.boot`插件版本已在父module中定义，本文件可不写版本直接使用。

`org.projectlombok:lombok`已在父module中的`subprojects`定义，可直接使用。