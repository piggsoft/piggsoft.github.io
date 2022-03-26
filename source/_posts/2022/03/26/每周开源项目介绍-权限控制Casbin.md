---
title: 每周开源项目介绍-权限控制Casbin
date: 2022-03-26 11:27:02
keyword: 
- Java
- 权限 
- Security
- casbin
- RBAC
- ABAC
tags:
- 权限系统
- RBAC
- ABAC
categories:
- 每周开源项目介绍
---

&ensp;&ensp;Hello,大家好，我是小毛驴。欢迎来到每周开源项目介绍栏目，今天的介绍对象为Github上的热点权限管理项目Casbin。通过本次的讲解大家将会了解到一些基础的权限控制的名称，常用解决方案，以及Casbin的特点，集成方式，使用场景等。

<!--more-->
## 什么是权限控制

&ensp;&ensp;在现实场景中有些操作只能由某个特定的人或特定岗位来完成，例如对外的公章，只能由老板来在确认后使用，或者你的请假条，将由你的直属上线来进行审批。  
&ensp;&ensp;将这些“权限要求”搬到信息化系统之后，就要求我们的信息化系统也能进行相应的权限控制，保证某些资源，操作只能由特定的人员，岗位来访问，操作。

## 权限管理范式有哪些

&ensp;&ensp;在IT届长时间的演化后，将一些权限系统设计的思路抽象化，形成了一些范式，这些范式就是：

* `ACL`
* `DAC`
* `MAC`
* `RBAC`
* `ABAC`
* `PBAC`
* `RAdAC`

### `ACL: Access Control List` 访问控制列表

#### 定义

* `Subject`能对`Object`进行`Action`
* `Subject`可以是单个用户也可以是群组

#### 示例

1. 给张三授予文档的创建权限。

    ```shell
    Subject: 张三
    Action: 创建
    Object: 文档
    ```

2. 张三现在可以创建文档了。

### `DAC: Mandatory Access Control` 自主访问控制

#### 定义

* `Subject`能对`Object`进行`Action`
* `Subject`能给其他`Subject`授权

#### 示例

1. 给张三授予文档的创建权限。

    ```shell
    Subject: 张三
    Action: 创建
    Object: 文档
    ```

2. 张三现在可以创建文档了，并且张三可以将这个权限授予给其他人。
3. 张三给李四授权创建文档权限。

    ```shell
    Subject: 李四
    Action: 创建
    Object: 文档
    ```

4. 李四现在可以创建文档了。

### `MAC: Mandatory Access Control` 强制访问控制

#### 定义

* `Subject`能对`Object`进行`Action`
* `Object`能被`Subject`进行`Action`
* `Subject`可以是单个用户也可以是群组

> 注意，MAC中的权限控制是双向的，既要确认用户是否有队资源的操作权限，也要确定资源是否能被这个用户进行操作的权限。适合保密程度要求搞的场景。

#### 示例

1. 给张三授予文档的创建权限。

    ```shell
    Subject: 张三
    Action: 创建
    Object: 文档
    ```

2. 将文档的被创建权限授权给张三。

    ```shell
    Subject: 文档
    Action: 被创建
    Object: 张三
    ```

3. 张三现在可以创建文档了。

### `RBAC: Role-Based Access Control` 基于角色的访问控制

### `ABAC: Attribute-Based Access Control` 基于属性的访问控制

### `PBAC: Policy-Based Access Control` 基于策略的访问控制

### `RAdAC: Risk Adaptive-Based Access Control` 基于风险自适应的访问控制 

## Casbin是什么？他采用了哪些策略

## Casbin的java路线的架构图

## 如下集成Casbin

## Casbin的优缺点