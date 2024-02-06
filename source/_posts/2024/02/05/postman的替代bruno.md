---
title: postman的替代bruno
date: 2024-02-05 09:44:06
  - 工具
  - Restful工具
  - API工具
  - http工具
categories:
  - [ "IT","工具","Restful工具" ]
---

`Postman`作为一款已有超过600万使用者的`Restful`/`API`
工具，其功能慢慢变得臃肿，内存占用越来越大，性能慢慢变差，并且还需要强制登录。使得我们需要寻找下一款`API`
工具，即使这款工具我们不是现在就更换，也需要为未来做下准备。

那么有哪些`API`工具可以替代`Postman`呢？

<!--more-->

## 可替代选择

当前开源方案中替代方案还是不少的。其中比较热门的方案是：

* [postwoman](#hoppscotch) (now hoppscotch)
* [insomnia](#insomnia)
* [insomnium](#insomnium)
* [bruno](#bruno)

## 介绍

### postman

#### 介绍

Postman是一款功能强大的HTTP协议接口调试与测试工具，它的主要特点在于使用简单且易用性良好。无论对于开发人员进行接口调试，还是测试人员做接口测试，Postman都是被广泛采用的工具之一。
Postman的功能众多，以下是其中一些主要的功能：

- 支持多种请求方式，包括`GET`、`POST`、`PUT`、`DELETE`等；
- 支持多种数据格式，如`JSON`、`XML`、`HTML`等；
- 支持自动化测试，可以编写测试脚本并执行；
- 支持接口集合，可以将多个接口保存到一个文件中；
- 支持接口分享，可以将接口分享给其他团队成员。
- 提供了直观的界面和丰富的功能，可以让开发者快速测试和调试API；
- 支持多种环境和变量，可以在不同的环境下进行测试和调试；
- 支持用例管理，可以轻松地管理和执行测试用例；
- 具有强大的抓包功能，可以方便地抓取和分析网络数据；
- 支持多终端同步，可以在不同设备上使用Postman；
- 可以将多个接口保存到一个文件中，便于管理和分享；
- 引入了工作流的概念，可以创建自动化的测试流程。

#### 地址

- [主页](https://www.postman.com/)
- [文档](https://learning.postman.com/docs/introduction/overview/)

#### 界面

![postman界面](postman-overview.png)

### hoppscotch

#### 介绍

在2018年Postman的开始对高级功能进行收费后，一群人就开始一个新的项目Postwoman，从名字来看就知道是冲着Postman去的，后面更名为Hoppscotch

作为一款API工具，Hoppscotch具有如下特色

- 轻量，简约的UI设计
- 实时快速
- 支持多种Http Method，例如`GET` `POST` `PUT` `PATCH` `DELETE` `HEAD` `CONNECT` `OPTIONS` `TRACE` `<custom>`(例如`LIST`)
- 自定义主题。白天，黑夜
- 支持协议：`WebSocket` `Server-Sent` `Events` `Socket.IO` `MQTT` `GraphQL`
- 支持鉴权：`None` `Basic` `Bearer Token` `OAuth 2.0` `OIDC Access Token/PKCE`
- 传参方式：`Headers` `Parameters` `Request Body`
- 支持Response预览，历史，请求集合（分类），Pre-Request Scripts，Workspaces，请求代理，i18n，数据同步，Post-Request
  Tests，环境配置，批量编辑，插件。

#### 地址

* [首页](https://hoppscotch.io/)
* [Github](https://github.com/hoppscotch/hoppscotch)
* [文档](https://docs.hoppscotch.io/)

#### 界面

![使用界面](hoppscotch-light.png)

### insomnia

#### 介绍

Insomnia是一个基于Node.js的API测试框架，它可以帮助开发人员快速测试和调试API。以下是Insomnia的功能列表和特点：
功能列表：

- 创建和管理API请求；
- 发送HTTP请求并接收响应；
- 支持多种HTTP方法和协议，如GET、POST、PUT等；
- 可以模拟各种HTTP状态码，如404、500等；
- 具有强大的断言功能，可以验证响应内容是否符合预期；
- 支持自定义请求头和查询参数；
- 可以将多个接口保存到一个文件中，便于管理和分享。
  特点：
- 提供了一个简单易用的界面，可以让开发人员轻松地创建测试用例和执行测试；
- 支持多种数据格式，如JSON、XML等；
- 可以与多种后端服务集成，如RESTful API、SOAP Web Service等；
- 具有强大的监控功能，可以实时跟踪和分析API流量；
- 支持多终端同步，可以在不同设备上使用Insomnia；
- 引入了工作流的概念，可以创建自动化的测试流程。

当然，Insomnia的功能和特点还有很多。首先，Insomnia的可视化界面非常直观、简洁，对新手用户来说非常友好，操作简单易于使用，可以快速开展接口测试工作。此外，Insomnia不仅支持Windows、Mac等桌面平台，还支持多种其他平台。
特别值得一提的是，Insomnia具有多级配置功能，可以设置三级的配置文件，用于多个不同环境的配置。最高级配置是Base
Environment，第二优先级配置是Sub Environments，在接口栏中创建文件夹，可以建立针对这个文件的环境，这是第三级配置。
另外，Insomnia会自动保存接口的访问结果，请求和响应数据都会完整的保存下来，需要的时候就可以再点开看。这个功能对于网络不稳定的情况尤其有用。

#### 地址

- [主页](https://insomnia.rest/)
- [Github](https://github.com/Kong/insomnia)
- [文档](https://docs.insomnia.rest/)

#### 界面

![insomnia.png](insomnia.png)

### insomnium

#### 介绍

Insomnia是一个跨平台的REST API客户端，它基于Electron构建，为开发者提供了测试和开发Web应用所需的工具和环境。它支持组织、运行和调试HTTP请求和API，包括REST,
GraphQL, gRPC等协议。Insomnia的特点包括：

- 本地运行：Insomnium完全在本地运行，不需要连接到任何云服务，保证了应用的隐私性和独立性。
- 跨平台兼容性：由于基于Electron，Insomnia可以在Windows、macOS和Linux等多种操作系统上运行。
- 功能丰富：Insomnia提供了各种功能，如请求链、环境变量、全局设置等，以帮助开发者更高效地进行API测试和开发。
- 用户界面友好：Insomnia的用户界面设计直观，便于开发者快速上手和使用。
- 隐私保护：由于其100%本地运行的特性，Insomnium没有后台与外部服务器的通信，确保了用户的数据隐私不会外泄。
- 支持多种协议：Insomnium支持包括GraphQL、REST、WebSockets、Server-sent events和gRPC等多种API测试，适用于各种开发场景。
- 使用便捷：尽管网上关于Insomnia的使用教程不多，但基础功能与其他常见的接口工具相似，因此上手相对容易。官方文档也提供了详尽的指导和进阶教程，如模板标签、请求链等。

综上所述，Insomnia是一个功能全面、用户友好的API客户端，适用于需要进行API测试和开发的开发者。

#### 地址

- [首页](https://archgpt.dev/insomnium)
- [Github](https://github.com/ArchGPT/insomnium)

#### 界面

![insomnium.png](insomnium.png)

### bruno

#### 介绍

Bruno 是一款全新且创新的 API 客户端，旨在颠覆 Postman 和其他类似工具。

Bruno 直接在您的电脑文件夹中存储您的 API 信息。我们使用纯文本标记语言 Bru 来保存有关 API 的信息。

您可以使用 Git 或您选择的任何版本控制系统来对您的API信息进行版本控制和协作。

Bruno 仅限离线使用。我们计划永不向 Bruno 添加云同步功能。我们重视您的数据隐私，并认为它应该留在您的设备上。

- 命令行友好：Bruno允许用户在命令行中方便地进行数据传输，这为喜欢使用终端的开发者提供了便利。
- 多协议支持：它支持多种协议，包括HTTP、FTP等，这使得Bruno能够适应不同的网络环境和使用场景。
- 丰富的选项和参数：Bruno提供了大量的选项和参数，用户可以通过这些选项和参数来定制请求，以满足各种不同的需求。
- API集合管理：它允许用户直接在文件系统上存储API集合，这意味着用户可以像管理普通文件一样管理自己的API接口集合。
- 使用Bru作为标记语言：Bruno使用一种名为Bru的文本标记语言来保存有关API请求的信息，这种方式简洁且易于理解。

#### 地址

- [首页](https://www.usebruno.com/)
- [Github](https://github.com/usebruno/bruno)
- [文档](https://docs.usebruno.com/)

#### 界面

![bruno.png](bruno.png)

## 对比

|                      | [Postman](#postman) | [hoppscotch](#hoppscotch) | [insomnia](#insomnia) | [insomnium](#insomnium) | [bruno](#bruno)   |
|----------------------|---------------------|---------------------------|-----------------------|-------------------------|-------------------|
| 跨平台                  | ✅                   | ✅                         | ✅                     | ✅                       | ✅                 |
| 基础API客户端             | ✅                   | ✅                         | ✅                     | ✅                       | ✅                 |
| 环境变量                 | ✅                   | ✅                         | ✅                     | ✅                       | ✅                 |
| API目录组织              | ✅                   | ✅                         | ✅                     | ✅                       | ✅                 |
| API存储                | ✅                   | ✅                         | ✅                     | ✅                       | ✅                 |
| 不需要登录                | ❌                   | ❌,可以选择不登录，但功能受限           | ❌,可以选择不登录，但功能受限       | ✅                       | ✅                 |
| Http Request         | ✅                   | ✅                         | ✅                     | ✅                       | ✅                 |
| Event Stream Request | ✅                   | ✅                         | ✅                     | ✅                       | ❌收费，  还未实现        |
| GraphQL Request      | ✅                   | ✅                         | ✅                     | ✅                       | ❌收费，  还未实现        |
| gRPC Request         | ✅                   | ✅                         | ✅                     | ✅                       | ❌收费，  还未实现        |
| Websocket Request    | ✅                   | ✅                         | ✅                     | ✅                       | ❌收费，  还未实现        |
| Socket.IO            | ✅                   | ❌                         | ❌                     | ❌                       | ❌收费，  还未实现        |
| MQTT                 | ✅                   | ❌                         | ❌                     | ❌                       | ❌收费，  还未实现        |
| History              | ✅                   | ✅                         | ✅                     | ✅                       | ❌                 |
| 插件                   | ❌                   | ✅                         | ✅                     | ✅                       | ❌,有计划支持           |
| From Curl            | ❌                   | ✅                         | ✅                     | ✅                       | ✅                 |
| 收藏的资源如何存储            | 全部在一个大json中         | 全部在一个大json中               | 全部在一个大json中           | 全部在一个大json中             | 一个request一个.bru文件 |
| 离线功能                 | ❌                   | ❌                         |                       | ✅                       | ✅                 | ✅|
| 团队协作                 | 收费                  | 收费                        | 收费                    | 收费                      | Git支持             | ✅|
| 上一个请求的结果当参数          | ✅                   | ✅                         | ✅                     | ✅                       | ✅                 |
| 测试断言                 | ✅                   | ✅                         | ❌                     | ❌                       | ✅                 |
| js脚本                 | ✅                   | ✅                         | ❌                     | ❌                       | ✅                 |
| 加载npm模块              | ❌                   | ❌                         | ❌                     | ❌                       | ✅                 |

## 推荐

我会推荐**Bruno**。

[bruno](#bruno)作为一款野心极强的新型Api工具，他现有的功能已经满足了Api测试工具的需求，并且无需登录，无需将信息存储到云端引起安全性问题。

通过观察其[Roadmap](https://github.com/usebruno/bruno/discussions/384)来看，基础功能在未来都会有，进阶功能会做成收费版本。

如果后续[bruno](#bruno)收费后有作恶的嫌疑，我推荐[insomnium](#insomnium)。之所以不第一推荐[insomnium](#insomnium)
的原因在于其维护的前景还不太明朗，后续和[insomnia](#insomnia)之间代码更新合并的问题也未看到明确规划。