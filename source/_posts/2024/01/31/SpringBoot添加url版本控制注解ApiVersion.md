---
title: SpringBoot添加url版本控制注解ApiVersion
date: 2024-01-31 14:17:33
keyword:
  - spring boot
  - 版本
  - version
  - url中的version
  - 固定的path variable
tags:
  - Java
  - Spring Boot
  - ApiVersion
categories:
  - [ Java, Spring Boot ]
---

作为服务端研发，在设计一个接口的时候，为了保证后续业务的新增和修改，会有各种的扩展性设计，其中关于URL的通用扩展设计是在path中加入版本。
例如: `https://domain.com/api/{version}/path` 。这种设计方便后续对接口升级时，只需要简单的升级版本号就可以了。

在有多个版本号之后，我们如何在Spring Boot中优雅的根据版本号进行不同的逻辑处理呢，本篇文章会介绍我正在使用的一种比较优雅的方法在Spring
Boot中来处理版本号。

<!--more-->

## 准备工作

分析现在的状况

1. 已有一个url并提供给了客户端进行使用，`https://www.domain.com/api/{version}/user/9527`
2. 现有的客户端提出一个不兼容的需求，但这个需求也是后续的通用需求
3. 这些改动不能影响已发布的接口，保持对外承认
4. 实现方式对现有代码侵入性小，且后续还能继续扩展版本

## 思路

1. 给出的url已经预留了version，我们可以基于这个字段进行扩展，讲url分为1.0, 2.0, 3.0, 4.0这样
    * https://www.domain.com/api/1.0/user/9527
    * https://www.domain.com/api/2.0/user/9527
    * https://www.domain.com/api/3.0/user/9527
    * https://www.domain.com/api/4.0/user/9527
2. 使用自定义ApiVersion注解，编写成如下代码
   ```java
   @RequestMapping("/api/{version}/user")
   @RestController
   public class UserController {
   
      @RequestMapping("/{userId}")
      public UserVo detail(@PathVariable userId) {
         return new UserVo();
      }
   
      @RequestMapping("/{userId}")
      @ApiVersion("1.0")
      public UserVo detailV1(@PathVariable userId) {
         return new UserVo();
      }
   
      @RequestMapping("/{userId}")
      @ApiVersion("2.0")
      public UserVo detailV2(@PathVariable userId) {
         return new UserVo();
      }
   
      @RequestMapping("/{userId}")
      @ApiVersion("3.0")
      public UserVo detailV3(@PathVariable userId) {
         return new UserVo();
      }
   
      @RequestMapping("/{userId}")
      @ApiVersion("4.0")
      public UserVo detailV4(@PathVariable userId) {
         return new UserVo();
      }
   }
   ```
3. ApiVersion注解的作用：在没有的时候version走模糊匹配；有ApiVersion将走路径的准确匹配，将{version}替换为注解的value。

## 调研

### 如何达到这个效果呢？

方案一：Spring在进行路径匹配时，`/api/1.0/user` 的优先级比 `/api/{version}/user`
高。只要我们将定义ApiVersion的RequestMapping变化一下，替换`{version}`变成全路径。

方案二：在原来的路径匹配的方法后面再加个尾巴，如果有ApiVersion就进行第二次判断，判断解析的PathVariable
version的值是否和注解中定义的value一致。

### 查找资料

#### 方案一

通过搜索引擎，查看代码，打断点debug等方法。在如下代码中有定义出`RequestMappingHandlerMapping`，这个类主要进行路径定义的匹配

```java
public class WebMvcConfigurationSupport implements ApplicationContextAware, ServletContextAware {
   @Bean
   @SuppressWarnings("deprecation")
   public RequestMappingHandlerMapping requestMappingHandlerMapping(
         @Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager,
         @Qualifier("mvcConversionService") FormattingConversionService conversionService,
         @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider) {
   
     RequestMappingHandlerMapping mapping = createRequestMappingHandlerMapping();
     mapping.setOrder(0);
     mapping.setInterceptors(getInterceptors(conversionService, resourceUrlProvider));
     mapping.setContentNegotiationManager(contentNegotiationManager);
     mapping.setCorsConfigurations(getCorsConfigurations());
   
     PathMatchConfigurer pathConfig = getPathMatchConfigurer();
     if (pathConfig.getPatternParser() != null) {
         mapping.setPatternParser(pathConfig.getPatternParser());
     }
     else {
         mapping.setUrlPathHelper(pathConfig.getUrlPathHelperOrDefault());
         mapping.setPathMatcher(pathConfig.getPathMatcherOrDefault());
   
         Boolean useSuffixPatternMatch = pathConfig.isUseSuffixPatternMatch();
         if (useSuffixPatternMatch != null) {
             mapping.setUseSuffixPatternMatch(useSuffixPatternMatch);
         }
         Boolean useRegisteredSuffixPatternMatch = pathConfig.isUseRegisteredSuffixPatternMatch();
         if (useRegisteredSuffixPatternMatch != null) {
             mapping.setUseRegisteredSuffixPatternMatch(useRegisteredSuffixPatternMatch);
         }
     }
     Boolean useTrailingSlashMatch = pathConfig.isUseTrailingSlashMatch();
     if (useTrailingSlashMatch != null) {
         mapping.setUseTrailingSlashMatch(useTrailingSlashMatch);
     }
     if (pathConfig.getPathPrefixes() != null) {
         mapping.setPathPrefixes(pathConfig.getPathPrefixes());
     }
   
     return mapping;
   }
}
```

`RequestMappingHandlerMapping`中有个很重要的方法`match`，这是运行时进行Request匹配的方法，匹配到了就执行相应的类和方法。

```java
@Override
public RequestMatchResult match(HttpServletRequest request, String pattern) {
  Assert.isNull(getPatternParser(), "This HandlerMapping requires a PathPattern");
  RequestMappingInfo info = RequestMappingInfo.paths(pattern).options(this.config).build();
  RequestMappingInfo match = info.getMatchingCondition(request);
  return (match != null && match.getPatternsCondition() != null ?
          new RequestMatchResult(
                  match.getPatternsCondition().getPatterns().iterator().next(),
                  UrlPathHelper.getResolvedLookupPath(request),
                  getPathMatcher()) : null);
}
```

而在这个方法里面主要是调用了`RequestMappingInfo`的`getMatchingCondition`方法，方法如下。

作用是按`POST GET`，参数，header等顺序进行匹配，其中`PatternsRequestCondition`和`PathPatternsRequestCondition`是我们研究的重点。
`PathPatternsRequestCondition`是最新的path解析工具`PatternsRequestCondition`是老版解析，正在被废弃中。

所以我们就围绕这个`RequestMappingInfo`来进行。
```java
public RequestMappingInfo getMatchingCondition(HttpServletRequest request) {
   RequestMethodsRequestCondition methods = this.methodsCondition.getMatchingCondition(request);
   if (methods == null) {
      return null;
   }
   ParamsRequestCondition params = this.paramsCondition.getMatchingCondition(request);
   if (params == null) {
      return null;
   }
   HeadersRequestCondition headers = this.headersCondition.getMatchingCondition(request);
   if (headers == null) {
      return null;
   }
   ConsumesRequestCondition consumes = this.consumesCondition.getMatchingCondition(request);
   if (consumes == null) {
      return null;
   }
   ProducesRequestCondition produces = this.producesCondition.getMatchingCondition(request);
   if (produces == null) {
      return null;
   }
   PathPatternsRequestCondition pathPatterns = null;
   if (this.pathPatternsCondition != null) {
      pathPatterns = this.pathPatternsCondition.getMatchingCondition(request);
      if (pathPatterns == null) {
          return null;
      }
   }
   PatternsRequestCondition patterns = null;
   if (this.patternsCondition != null) {
      patterns = this.patternsCondition.getMatchingCondition(request);
      if (patterns == null) {
          return null;
      }
   }
   RequestConditionHolder custom = this.customConditionHolder.getMatchingCondition(request);
   if (custom == null) {
      return null;
   }
   return new RequestMappingInfo(this.name, pathPatterns, patterns,
          methods, params, headers, consumes, produces, custom, this.options);
}
```


`RequestMappingHandlerMapping`这个类有2个方法是预留给我们进行自定义的。我们在两个方法中创建自己的Condition。
```java
	protected RequestCondition<?> getCustomTypeCondition(Class<?> handlerType) {
		return null;
	}

	@Nullable
	protected RequestCondition<?> getCustomMethodCondition(Method method) {
		return null;
	}

```

如果选用方案一，我们就可以直接创建一个`PatternsRequestCondition`或者`PathPatternsRequestCondition`，这样我们全路径的Condition优先级将更高，那么将悠闲访问带版本号的RequestMapping，我们的目的就达到了。

代码如下
````java
@Slf4j
public class ApiVersionRequestMappingHandlerMapping extends RequestMappingHandlerMapping {

    private ApiVersionProperties apiVersionProperties;
    private ConditionFactory conditionFactory;


    public ApiVersionRequestMappingHandlerMapping(ApiVersionProperties apiVersionProperties, ConditionFactory conditionFactory) {
        this.apiVersionProperties = apiVersionProperties;
        this.conditionFactory = conditionFactory;
    }


    @Override
    protected RequestCondition<?> getCustomTypeCondition(Class<?> handlerType) {
        ApiVersions apiVersions = AnnotationUtils.findAnnotation(handlerType, ApiVersions.class);
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(handlerType, ApiVersion.class);
        if (apiVersions == null && apiVersion == null) {
            return super.getCustomTypeCondition(handlerType);
        }

        RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(handlerType, RequestMapping.class);

        if (apiVersions != null) {
            return createRequestCondition(requestMapping, null, apiVersions);
        }

        return createRequestCondition(requestMapping, null, apiVersion);
    }

    @Override
    protected RequestCondition<?> getCustomMethodCondition(Method method) {
        Class<?> methodClass = method.getDeclaringClass();

        ApiVersions apiVersions = AnnotationUtils.findAnnotation(method, ApiVersions.class);
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(method, ApiVersion.class);
        if (apiVersions == null && apiVersion == null) {
            return super.getCustomMethodCondition(method);
        }

        RequestMapping classMapping = AnnotatedElementUtils.findMergedAnnotation(methodClass, RequestMapping.class);
        RequestMapping methodMapping = AnnotatedElementUtils.findMergedAnnotation(method, RequestMapping.class);


        if (apiVersions != null) {
            return createRequestCondition(classMapping, methodMapping, apiVersions);
        }

        return createRequestCondition(classMapping, methodMapping, apiVersion);
    }

    protected RequestCondition<?> createRequestCondition(RequestMapping classMapping, RequestMapping methodMapping, ApiVersions apiVersions) {
        return createRequestCondition(classMapping, methodMapping, apiVersions.value());
    }

    protected RequestCondition<?> createRequestCondition(RequestMapping classMapping, RequestMapping methodMapping, ApiVersion... apiVersions) {
        if (apiVersions == null || apiVersions.length < 1) {
            return null;
        }


        List<String> paths = null;
        if (classMapping == null && methodMapping == null) {
            return null;
        } else if (methodMapping == null) {
            String[] classPaths = classMapping.value();
            paths = Stream.of(classPaths).collect(Collectors.toList());
        } else if (classMapping == null) {
            String[] methodPaths = methodMapping.value();
            paths = Stream.of(methodPaths).collect(Collectors.toList());
        } else {
            String[] classPaths = classMapping.value();
            String[] methodPaths = methodMapping.value();
            paths = Arrays.stream(classPaths).flatMap(classPath -> Arrays.stream(methodPaths).map(methodPath -> classPath + methodPath)).collect(Collectors.toList());
        }

        List<String> apiVersionValues = Arrays.stream(apiVersions).map(ApiVersion::value).collect(Collectors.toList());

        String[] fullPaths = paths.stream().flatMap(path -> apiVersionValues.stream().map(apiVersion -> this.replacePlaceholder(path, apiVersion))).toArray(String[]::new);

        /* return new PatternsRequestCondition(fullPaths);*/
        return conditionFactory.create(fullPaths);
    }


    protected String replacePlaceholder(String text, String apiVersion) {
        if (ObjectUtils.isEmpty(text)) {
            return text;
        }
        int startIndex = text.indexOf(apiVersionProperties.getQuoteLeft());
        if (startIndex == -1) {
            return text;
        }
        int endIndex = text.indexOf(apiVersionProperties.getQuoteRight());
        if (endIndex == -1) {
            return text;
        }

        String startString = text.substring(0, startIndex);
        String endString = text.substring(endIndex + 1);

        return startString + apiVersion + endString;
    }
}

````

配置类中进行如下配置

本配置的作用是在低版本是没有`PathPatternsRequestCondition`,兼容旧版本
````java
@ConditionalOnClass(PathPatternsRequestCondition.class)
    @Bean
    public PathPatternsRequestConditionFactory pathPatternsRequestCondition() {
        return new PathPatternsRequestConditionFactory();
    }

    @ConditionalOnMissingClass("org.springframework.web.servlet.mvc.condition.PathPatternsRequestCondition")
    @Bean
    public PatternsRequestConditionFactory patternsRequestConditionFactory() {
        return new PatternsRequestConditionFactory();
    }

    @Bean
    public WebMvcRegistrations webMvcRegistrations(ApiVersionProperties apiVersionProperties, ConditionFactory conditionFactory) {
        return new WebMvcRegistrations() {
            @Override
            public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
                return new ApiVersionRequestMappingHandlerMapping(apiVersionProperties, conditionFactory);
            }
        };
    }
````

接下来在Controller中我们就可以如下使用了
````java
@RequestMapping("/version/{version}")
@RestController
public class VersionController {

    @RequestMapping("/test")
    public String v1() {
        return "v1";
    }

    @RequestMapping("/test")
    @ApiVersion("1")
    public String v11() {
        return "v11";
    }

    @RequestMapping("/test")
    @ApiVersion("2")
    public String v2() {
        return "v2";
    }

    @RequestMapping("/test")
    @ApiVersion("3")
    public String v3() {
        throw new RuntimeException("12312312");
    }

}
````

#### 方案二
暂时还没有好的思路，如果看的同学有较好的思路，可以在后面留言回复下。