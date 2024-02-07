---
title: Apple-pkl介绍
date: 2024-02-07 11:20:31
keyword:
tags:
  - Spring
  - Spring boot
  - Spring boot Configuration
  - application.yml
  - application.properties
  - Apple
  - Apple PKL
  - application.okl
categories:
  - [ "IT", "工具" ]
  - [ "Java", "Spring Boot" ]
---

在当今快速发展的软件开发领域，配置管理一直是一个重要且挑战性的议题。随着应用程序变得越来越复杂，对配置文件的需求也随之增加，这促使开发者寻找更加高效、安全且易于维护的解决方案。苹果公司在这方面迈出了重要的一步，推出了[Pkl](https://github.com/apple/pkl)
（Property List）语言，这是一种旨在提供丰富数据模板和验证支持的可嵌入配置语言。在这篇文章中，我们将深入探讨Pkl语言的核心特性、设计理念以及它在现代软件开发中的应用前景。

<!--more-->

## 背景

[Pkl](https://github.com/apple/pkl)
语言起源于苹果生态系统对配置文件处理需求的深刻理解，它脱胎于macOS和iOS系统中广泛使用的.plist文件格式。自诞生之初，[Pkl](https://github.com/apple/pkl)
就以其简洁明了的键值结构设计和对多种文件格式的支持赢得了开发者的关注。不仅如此，Pkl还致力于提供一种安全可靠的配置处理方式，通过引入验证机制来确保配置数据的准确性和完整性。

作为一种静态配置文件格式，[Pkl](https://github.com/apple/pkl)
支持JSON、XML和YAML等流行格式，这意味着开发者可以根据项目需求灵活选择最合适的格式。它的设计哲学中包含了两个核心目标：语法安全性和可扩展性。前者确保了配置文件的语法分析是安全的，减少了因错误的配置而导致的安全风险；后者则体现在Pkl支持类、函数、条件和循环等高级编程特性，使其能够应对更加复杂的配置场景。

苹果公司对Pkl的开发并没有止步于此。2024年2月2日，Apple公司在Github上开源了该项目[Pkl](https://github.com/apple/pkl)
,一经发布立即登上了[Github Tending](https://github.com/trending)。

## 初探

定义一个配置文件`bird.pkl`

```pkl
name = "Swallow"

job {
  title = "Sr. Nest Maker"
  company = "Nests R Us"
  yearsOfExperience = 2
}
```

这个配置文件等同于

`bird.json`

```json
{
  "name": "Swallow",
  "job": {
    "title": "Sr. Nest Maker",
    "company": "Nests R Us",
    "yearsOfExperience": 2
  }
}
```

`bird.yaml`

```yaml
name: Swallow
job:
  title: Sr. Nest Maker
  company: Nests R Us
  yearsOfExperience: 2
```

`bird.properties`

```properties
name = Swallow
job.title = Sr. Nest Maker
job.company = Nests R Us
job.yearsOfExperience = 2
```

`bird.plist`，流行于MacOS,IOS的配置文件格式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>name</key>
        <string>Swallow</string>
        <key>job</key>
        <dict>
            <key>title</key>
            <string>Sr. Nest Maker</string>
            <key>company</key>
            <string>Nests R Us</string>
            <key>yearsOfExperience</key>
            <integer>2</integer>
        </dict>
    </dict>
</plist>
```

上面4中文件格式类型，我们可以通过工具将kpl转换为指定需求的类型。

## 代码生成

讲Pki的相关依赖嵌入到应用程序时，还会更具`.pkl`文件生成相应语言的代码。

例如`.pkl`文件如下：

```pkl
// Code generated from Pkl module `example.myAppConfig`. DO NOT EDIT.
package myappconfig

import (
	"context"

	"github.com/apple/pkl-go/pkl"
)

type MyAppConfig struct {
	// The hostname for the application
	Host string `pkl:"host"`

	// The port to listen on
	Port uint16 `pkl:"port"`
}
```

即可**自动生成**如下语言的文件，后续可能还会扩充

`Java`

```Java
package example;

public final class MyAppConfig {
  /**
   * The hostname for the application
   */
  public final @NonNull String host;

  /**
   * The port to listen on
   */
  public final int port;

  public MyAppConfig(
        @Named("host") @NonNull String host,
        @Named("port") int port) {
    this.host = host;
    this.port = port;
  }

  public MyAppConfig withHost(@NonNull String host) { /*...*/ }

  public MyAppConfig withPort(int port) { /*...*/ }

  @Override
  public boolean equals(Object obj) { /*...*/ }

  @Override
  public int hashCode() { /*...*/ }

  @Override
  public String toString() { /*...*/ }
}
```

`Kotlin`

```Kotlin
package example

data class MyAppConfig(
  /**
   * The hostname for the application
   */
  val host: String,
  /**
   * The port to listen on
   */
  val port: Int
)
```

`Swift`

```Swift
// Code generated from Pkl module `example.myAppConfig`. DO NOT EDIT.
enum MyAppConfig {}

extension MyAppConfig {
    struct Module {
        /// The hostname for the application
        let host: String

        /// The port to listen on
        let port: UInt16
    }
}
```

`Go`

```go
// Code generated from Pkl module `example.myAppConfig`. DO NOT EDIT.
package myappconfig

import (
	"context"

	"github.com/apple/pkl-go/pkl"
)

type MyAppConfig struct {
	// The hostname for the application
	Host string `pkl:"host"`

	// The port to listen on
	Port uint16 `pkl:"port"`
}
```

## IDEA支持

当前已经支持`.pkl`的IDEA有，其他正在进行中

- IntelliJ
- Visual Studio Code
- Neovim

![intellij.gif](intellij.gif)

## 开发时校验

在拥有丰富的类型和校验系统支持的情况下，我们可以在开发时就得到配置文件格式和值错误的信息

```
email: String = "dev-team@company.com"

port: Int(this > 1000) = 80
```

```
–– Pkl Error ––
Type constraint `this > 1000` violated.
Value: 80

3 | port: Int(this > 1000) = 80
              ^^^^^^^^^^^
at config#port (config.pkl, line 3)

3 | port: Int(this > 1000) = 80
                             ^^
at config#port (config.pkl, line 3)

106 | text = renderer.renderDocument(value)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

## 例子-SpringBoot集成

[代码地址](https://github.com/apple/pkl-spring)

### pkl-spring

Pkl-Spring 通过boot-starter配置将Pkl引入到Spring Boot中，具体的代码如下

`spring.factories`

```
org.springframework.boot.env.PropertySourceLoader=org.pkl.spring.boot.PklPropertySourceLoader
org.springframework.boot.autoconfigure.EnableAutoConfiguration=org.pkl.spring.boot.PklAutoConfiguration
```

`PklPropertySourceLoader`

```Java
public class PklPropertySourceLoader implements PropertySourceLoader {
  @Override
  public String[] getFileExtensions() {
    return new String[] {"pkl", "pcf"};
  }

  @Override
  public List<PropertySource<?>> load(String propertySourceName, Resource resource)
      throws IOException {
    var text = StreamUtils.copyToString(resource.getInputStream(), StandardCharsets.UTF_8);

    PModule module;
    try (var evaluator = EvaluatorBuilder.preconfigured().build()) {
      module = evaluator.evaluate(ModuleSource.create(resource.getURI(), text));
    }

    var result = new LinkedHashMap<String, Object>();
    module.getProperties().forEach((name, value) -> flatten(name, value, result));
    return List.of(new MapPropertySource(propertySourceName, result));
  }

  private static void flatten(
      String propertyName, Object propertyValue, Map<String, Object> result) {
    if (propertyValue instanceof Composite) {
      flatten(propertyName, ((Composite) propertyValue).getProperties(), result);
    } else if (propertyValue instanceof Map<?, ?>) {
      var map = (Map<?, ?>) propertyValue;
      if (map.isEmpty()) {
        result.put(propertyName, Collections.emptyMap());
      } else {
        map.forEach((name, value) -> flatten(propertyName + '.' + name, value, result));
      }
    } else if (propertyValue instanceof Collection) {
      var collection = (Collection<?>) propertyValue;
      if (collection.isEmpty()) {
        result.put(
            propertyName,
            propertyValue instanceof Set ? Collections.emptySet() : Collections.emptyList());
      } else {
        var index = 0;
        for (var element : collection) {
          flatten(propertyName + '[' + index++ + ']', element, result);
        }
      }
    } else {
      result.put(propertyName, propertyValue);
    }
  }
}
```

`PklAutoConfiguration`

```Java
@Configuration
public class PklAutoConfiguration {
  public PklAutoConfiguration(ConfigurableEnvironment env) {
    // otherwise `Environment.getProperty("pklPropertyWithNullValue")` fails with
    // `ConverterNotFoundException`
    env.getConversionService().addConverter(new PNullConverter());
  }

  @Component
  @SuppressWarnings("unused")
  @ConfigurationPropertiesBinding
  public static class PNullConverter implements GenericConverter {
    @Override
    public @Nullable Set<ConvertiblePair> getConvertibleTypes() {
      return Set.of(new ConvertiblePair(PNull.class, Object.class));
    }

    @Override
    public @Nullable Object convert(
        @Nullable Object source, TypeDescriptor sourceType, TypeDescriptor targetType) {
      assert source == PNull.getInstance();
      return targetType.getType() == Optional.class ? Optional.empty() : null;
    }
  }
}
```

### pkl-spring使用例子

#### 引入依赖

`build.gradle.kts`

```Kotlin
dependencies {
  compile "org.pkl-lang:pkl-spring:0.15.0"
}
```

#### 定义配置模型

`AppConfig.pkl`

```Pkl
// this module name determines the package and
// class name of the generated Java config class
module samples.boot.AppConfig

server: Server

class Server {
  endpoints: Listing<Endpoint>
}

class Endpoint {
  name: String
  port: UInt16
}
```

#### 定义Spring Boot配置文件

`application.pkl`

```Pkl
amends "modulepath:/appConfig.pkl"

server {
  endpoints {
    new {
      name = "endpoint1"
      port = 1234
    }
    new {
      name = "endpoint2"
      port = 5678
    }
  }
}
```

#### 添加Pkl插件和生成配置

`build.gradle.kts`

```Kpl
plugins {
  id("org.pkl-lang") version "$pklVersion"
}

pkl {
  javaCodeGenerators {
    register("configClasses") {
      generateGetters.set(true)
      generateSpringBootConfig.set(true)
      sourceModules.set(files("src/main/resources/AppConfig.pkl"))
    }
  }
}
```

#### Spring Boot中代码配置

`Application.java`

```Java
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application { ... }
```

`Service.java`
这里就可以直接注入和使用`AppConfig`

```Java
@Service
public class Service {
  public Service(AppConfig.Server config) { ... }
}
```

`Service1.java`

```Java
@Service
public class Service1 {
  public Service1(AppConfig config) { ... }
}
```

## 参考

- https://pkl-lang.org/
- https://pkl-lang.org/main/current/introduction/use-cases.html
- https://pkl-lang.org/go/current/index.html
- https://pkl-lang.org/spring/current/usage.html