---
title: PKI体系（三）-EJBCA安装
date: 2018-07-11 14:17:24
keyword:
- PKI
- EJBCA
tags: 
- PKI
- EJBCA
- 安全
categories:
- 安全
---

EJBCA是一个全功能的CA系统软件，它基于J2EE技术，并提供了一个强大的、高性能并基于组件的CA。EJBCA兼具灵活性和平台独立性，能够独立使用，也能和任何J2EE应用程序集成。

<!--more-->

# 特点

* 特性： LGPL开源许可;

* 建立在J2EE 1.3（EJB2.0）规范之上;

* 灵活的、基于组件的体系结构;

* 多级CA

* 多个CA和多级CA，在一个EJBCA实例中建立一个或者多个完整的基础设施单独运行，或者在任何J2EE应用中集成它;

* 简单的安装和配置

* 强大的基于Web的管理界面，并采用了高强度的鉴别算法

* 支持基于命令行的管理，并支持脚本等功能

* 支持个人证书申请或者证书的批量生产

* 服务器和客户端证书能够采用PKCS12, JKS或者PEM格 式导出

* 支持采用Netscape, Mozilla, IE等浏览器直接进行证书申请

* 支持采用开放API和工具通过其它应用程序申请证书

* 由RA添加的新用户可以通过email进行提醒

* 对于新用户验证可以采用随机或者手工的方式生成密码

* 支持硬件模块，来集成硬件签发系统（例如智能卡）

* 支持SCEP

* 支持用特定用户权限和用户组的方式来进行多极化管理

* 对不同类型和内容的证书可以进行证书配置

* 对不同类型的用户可以进行实体配置

* 遵循X509和PKIX(RFC3280)标准

* 支持CRL

* 完全支持OCSP，包括AIA扩展

* CRL生成和基于URL的CRL分发点遵循RFC3280，可以在任何SQL数据库中存储证书和CRL（通过应用服务器来处 理）。

* 可选的多个发布器，以用来在LDAP中 发布证书和CRL

* 支持用来为指定用户和证书来恢复私钥的密钥恢复模块

* 基于组件的体系结构，用来发布证书和CRL到不 同的目的地

* 基于组件的体系结构，用来在发布证书时采用多种实体授权方法

* 容易集成到大型应用程序中，并为集成到业务流程进行了优化

* EJBCA完 全采用Java编写，能够在任何采用J2EE服 务器的平台上运行。开发和测试是在Linux和Windows上 进行的。

# 安装前的准备

1. JDK1.8
2. apache-ant-1.9.11-bin
3. Wildfly-12.0.0.Final
4. ejbca_ce_6_10_1_2
5. mysql-connector-java-5.1.46.jar
6. MySQL
7. Centos 7 (也可是其他系统)

> 将其中的Wildfly，ejbca_ce_6_10_1_2，mysql-connector-java-5.1.46.jar放到同一个目录下，比如`/opt/ca`

# Mysql建表

在mysql中新建一个表对应EJBCA的数据

```mysql
CREATE DATABASE ejbcatest CHARACTER SET utf8 COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON ejbcatest.* TO 'ejbca'@'%' IDENTIFIED BY 'ejbca';
```

# 系统准备

1. 安装JDK
2. 安装ant
    1. 解压ant `unzip apache-ant-1.9.11-bin.zip`
    2. 配置path `vim /etc/profile`, 在文件末尾添加`export PATH="/opt/apache-ant-1.9.11/bin:$PATH"`
    3. 编译生效`source /etc/profile`
    4. 检查`ant`是否成功安装`ant -version`
3. 为EJBCA新建用户
    ```shell
    adduser ca
    passwd ca
    ```
4. 将安装包的所有者改成`ca`
    ```shell
    chown ca:ca /opt/ca
    chown ca:ca /opt/ca/wildfly-12.0.0.Final.zip
    chown ca:ca /opt/ca/ejbca_ce_6_10_1_2.zip
    chown ca:ca /opt/ca/mysql-connector-java-5.1.46.jar
    ```

# 安装

1. 切换用户 `su - ca`
2. 解压Wildfly `unzip /opt/ca/wildfly-12.0.0.Final.zip`
3. 修改Wildfly运行配置文件
    1. `vim wildfly-12.0.0.Final/bin/standalone.conf`
    2. 找到53行
    3. 将改行注释掉`#JAVA_OPTS="-Xms64m -Xmx512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true"`
    4. 在该行下面加入一行`JAVA_OPTS="-Xms2048m -Xmx2048m -Djava.net.preferIPv4Stack=true"`
    5. 保存，退出
4. 启动Wildfly`./wildfly-12.0.0.Final/bin/standalone.sh`
5. 对Wildfly进行配置
    1. 启动客户端`./wildfly-12.0.0.Final/bin/jboss-cli.sh -c`。出现如下字符即进入成功 *[standalone@localhost:9990 /]*
    2. 配置数据源
        ```shell
        module add --name=com.mysql --resources=/opt/ca/mysql-connector-java-5.1.46.jar --dependencies=javax.api,javax.transaction.api

        /subsystem=datasources/jdbc-driver=mysql:add(driver-name="mysql",driver-module-name="com.mysql",driver-xa-datasource-class-name=com.mysql.jdbc.Driver)

        data-source add --name=ejbcads --driver-name="mysql" --connection-url="jdbc:mysql://127.0.0.1:3306/ejbcatest" --jndi-name="java:/EjbcaDS" --use-ccm=true --driver-class="com.mysql.jdbc.Driver" --user-name="username" --password="password" --validate-on-match=true --background-validation=false --prepared-statements-cache-size=50 --share-prepared-statements=true --min-pool-size=5 --max-pool-size=150 --pool-prefill=true --transaction-isolation=TRANSACTION_READ_COMMITTED --check-valid-connection-sql="select 1;"

        ```
        > 上面的命令和后续的配置命令都需要一条一条执行。注意替换`--connection-url="jdbc:mysql://127.0.0.1:3306/ejbcatest"`，`--user-name="username"`，`--password="password"`
    3. 配置Wildfly远程调用
        ```shell
        /subsystem=remoting/http-connector=http-remoting-connector:remove
        /subsystem=remoting/http-connector=http-remoting-connector:add(connector-ref="remoting",security-realm="ApplicationRealm")
        /socket-binding-group=standard-sockets/socket-binding=remoting:add(port="4447")
        /subsystem=undertow/server=default-server/http-listener=remoting:add(socket-binding=remoting)
        :reload
        ```
        > 输入完成，需等待Wildfly reload完成，可用命令`:read-attribute(name=server-state)`来进行检查。如果出现如下信息即代表reload成功。
        ```shell
        {
            "outcome" => "success",
            "result" => "running"
        }
        ```
    4. 配置Wildfly log
        ```shell
        /subsystem=logging/logger=org.ejbca:add
        /subsystem=logging/logger=org.ejbca:write-attribute(name=level, value=DEBUG)
        /subsystem=logging/logger=org.cesecore:add
        /subsystem=logging/logger=org.cesecore:write-attribute(name=level, value=DEBUG)
        ```
    5. 关闭jboos-cli
6. 对ejbca进行配置
    1. 解压文件，`unzip ejbca_ce_6_10_1_2.zip`
    2. 修改`ejbca.properties` 文件
        1. `cp ejbca_ce_6_10_1_2/conf/ejbca.properties.sample ejbca_ce_6_10_1_2/conf/ejbca.properties`
        2. `vim ejbca_ce_6_10_1_2/conf/ejbca.properties`
        3. 设置 `appserver.home` 的值(就是应用服务器的安装位置,对于我们来说是 `/opt/ca/wildfly-10.0.0.Final`)， 最终结果`appserver.home=/opt/ca/wildfly-12.0.0.Final`
    3. 修改`web.properties`文件
        1. `cp ejbca_ce_6_10_1_2/conf/web.properties.sample ejbca_ce_6_10_1_2/conf/web.properties`
        2. `vim ejbca_ce_6_10_1_2/conf/web.properties`
        3. 设置CA的超级管理员的证书密码,以及给应用服务器生成的服务器端证书的证书密码,和CA的truststory的密码等,这些密码的设置我们可以根据需要设置,或者保持默认的配置,需要注意的是httpsserver.hostname,这个要和后边的alias相对应,我的ip地址为 147.128.105.149,那这里我们设置为147.128.105.149.
        4. 最终修改，`httpsserver.hostname=127.0.0.1`
    4. 修改`database.properties`
        1. `cp ejbca_ce_6_10_1_2/conf/database.properties.sample ejbca_ce_6_10_1_2/conf/database.properties`
        2. `vim ejbca_ce_6_10_1_2/conf/database.properties`
        3. 实际上只要使用wildfly的数据源即可 。把`datasource.jndi-name=EjbcaDS`的注释取消， 还要把 数据库类型`database.name=mysql`这个注释也要放开。否则安装出错，会执行h2数据库的库表安装脚本。
    5. 修改`install.properties`
        1. `cp ejbca_ce_6_10_1_2/conf/install.properties.sample ejbca_ce_6_10_1_2/conf/install.properties`
        2. `vim ejbca_ce_6_10_1_2/conf/install.properties`
        3. 设置CA的名称,加密方式等，建议保持默认即可。
7. ejbca打包部署
    ```shell
    cd /opt/ejbca_ce_6_5.0.5/
    ant clean deployear
    ant runinstall
    ant deploy-keystore
    ```
8. 去除Wildfly当前的HTTPS和TLS配置，需要使用`./wildfly-12.0.0.Final/bin/jboss-cli.sh -c`
    ```shell
    /subsystem=undertow/server=default-server/http-listener=default:remove
    /subsystem=undertow/server=default-server/https-listener=https:remove
    /socket-binding-group=standard-sockets/socket-binding=http:remove
    /socket-binding-group=standard-sockets/socket-binding=https:remove
    ```
9. 配置TLS
    1. 配置外部可访问
        ```shell
        /interface=http:add(inet-address="0.0.0.0")
        /interface=httpspub:add(inet-address="0.0.0.0")
        /interface=httpspriv:add(inet-address="0.0.0.0")
        /socket-binding-group=standard-sockets/socket-binding=http:add(port="8080",interface="http")
        /socket-binding-group=standard-sockets/socket-binding=httpspriv:add(port="8443",interface="httpspriv")
        /socket-binding-group=standard-sockets/socket-binding=httpspub:add(port="8442", interface="httpspub")
        /subsystem=undertow/server=default-server/http-listener=http:add(socket-binding=http)
        /subsystem=undertow/server=default-server/http-listener=http:write-attribute(name=redirect-socket, value="httpspriv")
        :reload
        ```
    2. 配置端口绑定
        ```shell
        /core-service=management/security-realm=SSLRealm:add()
        /core-service=management/security-realm=SSLRealm/server-identity=ssl:add(keystore-path="${jboss.server.config.dir}/keystore/keystore.jks", keystore-password="serverpwd", alias="127.0.0.1")
        /core-service=management/security-realm=SSLRealm/authentication=truststore:add(keystore-path="${jboss.server.config.dir}/keystore/truststore.jks", keystore-password="changeit")
        :reload
        ```
        > 其中`keystore-password="serverpwd"`对应`web.properties`里面的`httpsserver.password`。`alias="127.0.0.1"`对应`web.properties`里面的`httpsserver.hostname`
    3. 重启wildfly`:shutdown(restart=true)`,等待重启完毕
    4. 配置tls
        ```shell
        /subsystem=undertow/server=default-server/https-listener=httpspriv:add(socket-binding=httpspriv, security-realm="SSLRealm", verify-client=REQUIRED)
        /subsystem=undertow/server=default-server/https-listener=httpspriv:write-attribute(name=max-parameters, value="2048")
        /subsystem=undertow/server=default-server/https-listener=httpspub:add(socket-binding=httpspub, security-realm="SSLRealm")
        /subsystem=undertow/server=default-server/https-listener=httpspub:write-attribute(name=max-parameters, value="2048")
        :reload
        ```
    5. 配置编码等其他重要配置
        ```shell
        /system-property=org.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH:add(value=true)
        /system-property=org.apache.catalina.connector.CoyoteAdapter.ALLOW_BACKSLASH:add(value=true)
        /system-property=org.apache.catalina.connector.URI_ENCODING:add(value="UTF-8")
        /system-property=org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING:add(value=true)
        /subsystem=webservices:write-attribute(name=wsdl-host, value=jbossws.undefined.host)
        /subsystem=webservices:write-attribute(name=modify-wsdl-address, value=true)
        :reload
        ```
    6. 重启wildfly`:shutdown(restart=true)`,等待重启完毕
10. 访问[http://127.0.0.1:8080/ejbca](http://127.0.0.1:8080/ejbca)进行验证
11. 下载管理员证书，将`/opt/ca/ejbca_ce_6_10_1_2/p12`下的`superadmin.p12`拷贝到本地，对证书进行安装
12. 访问[https://127.0.0.1:8443/ejbca](https://127.0.0.1:8443/ejbca)进行验证
> 注：所有的ip需要换成对应的ip或者域名