---
title: ELK_Nignx_Configuration
date: 2021-04-15 15:02:09
keyword: ELK Nignx Configuration
tags: nginx elk ELK
categories: ELK
---

使用ELK对nginx日志进行收集，并分析

<!--more-->

## 使用ELK对nginx日志进行收集，并分析

## Filebeat配置

```conf
###################### Filebeat Configuration Example #########################

# This file is an example configuration file highlighting only the most common
# options. The filebeat.reference.yml file from the same directory contains all the
# supported options with more comments. You can use it as a reference.
#
# You can find the full configuration reference here:
# https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-input-log.html

# For more available modules and options, please see the filebeat.reference.yml sample
# configuration file.
#=========================== Filebeat inputs =============================
#
filebeat.inputs:
- type: log
  enabled: true
  paths:
  # 日志存储目录，path配置参考https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-input-log.html#input-paths
    #- /usr/local/nginx/logs/access.cn.log
    - /usr/local/nginx-1.17.9/logs/access.log
    #- /opt/webapp/filebeat-6.7.0-linux-x86_64/local.log
  #  - /data/logs/server2/*.log
  fields:
  # 该字段用于标记服务来源，为了和容器云保持一致，字段定义为kubernetes.namespace，对应的值根据实际用途定义
    namespace: nginx-log
  fields_under_root: true
  # 可多行读取，需根据多行的起始标识来配置pattern，改多行读取配置为apache日志的配置
  #multiline.pattern: '^\['
  #multiline.negate: true
  #multiline.match: after

#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: [""]
  worker: 4
  bulk_max_size: 2048

#============================== Xpack Monitoring ===============================
#xpack.monitoring.enabled: true
#xpack.monitoring:
#  enabled: true
#  elasticsearch:
#    hosts: [""]
```

## Logstash配置

```conf
input {
  beats {
    port => 5044
  }
}
filter{
  if [namespace] == "nginx-log" {
    grok {
      # match => { "message" => ["%{IPORHOST:[nginx][access][remote_ip]} - %{DATA:[nginx][access][user_name]} \[%{HTTPDATE:[nginx][access][time]}\] \"%{WORD:[nginx][access][method]} %{URIPATH:[nginx][access][path]}(?:%{URIPARAM:[nginx][access][params]})? HTTP/%{NUMBER:[nginx][access][http_version]:float}\" %{DATA:[nginx][access][http_host]} %{NUMBER:[nginx][access][response_code]:int} %{NUMBER:[nginx][access][body_sent][bytes]:int} \"%{DATA:[nginx][access][scheme]}\" \"%{DATA:[nginx][access][referrer]}\" \"%{DATA:[nginx][access][agent]}\" \"%{DATA:[nginx][access][forwarded]}\" %{NUMBER:[nginx][access][request_time]:float} %{NUMBER:[nginx][access][response_time]:float} %{HOSTPORT:[nginx][access][upstream]}"] }
       match => { "message" => [
         "%{IPORHOST:[nginx][access][remote_ip]} - %{DATA:[nginx][access][user_name]} \[%{HTTPDATE:[nginx][access][time]}\] \"%{WORD:[nginx][access][method]} %{URIPATH:[nginx][access][path]}(?:%{CUSTOMER_URIPARAM:[nginx][access][params]})? HTTP/%{NUMBER:[nginx][access][http_version]:float}\" %{DATA:[nginx][access][http_host]} %{NUMBER:[nginx][access][response_code]:int} %{NUMBER:[nginx][access][body_sent][bytes]:int} \"%{DATA:[nginx][access][scheme]}\" \"%{DATA:[nginx][access][referrer]}\" \"%{DATA:[nginx][access][agent]}\" \"%{DATA:[nginx][access][forwarded]}\" %{NUMBER:[nginx][access][request_time]:float} %{NUMBER:[nginx][access][response_time]:float}(, )?%{NUMBER:[nginx][access][upstream_switch_time]:float}? %{HOSTPORT:[nginx][access][upstream]}(, %{HOSTPORT:[nginx][access][upstreamSwitch]})?",
        "%{IPORHOST:[nginx][access][remote_ip]} - %{DATA:[nginx][access][user_name]} \[%{HTTPDATE:[nginx][access][time]}\] \"%{WORD:[nginx][access][method]} %{URIPATH:[nginx][access][path]}(?:%{CUSTOMER_URIPARAM:[nginx][access][params]})? HTTP/%{NUMBER:[nginx][access][http_version]:float}\" %{DATA:[nginx][access][http_host]} %{NUMBER:[nginx][access][response_code]:int} %{NUMBER:[nginx][access][body_sent][bytes]:int} \"%{DATA:[nginx][access][scheme]}\" \"%{DATA:[nginx][access][referrer]}\" \"%{DATA:[nginx][access][agent]}\" \"%{DATA:[nginx][access][forwarded]}\" %{NUMBER:[nginx][access][request_time]:float}"
       ] }
       pattern_definitions => {
         "CUSTOMER_URIPARAM" => "\?[A-Za-z0-9$.+!*'|(){},~@#%&/=:;_?\-\[\]<>\\]*"
       }
       remove_field => "message"
     }
     kv {
      source => "[nginx][access][params]" # 默认是message，我们这里只需要解析上面grok抽取出来的request字段
      target => "[nginx][access][paramsJson]"
      field_split => "&?"
      value_split => "="
      include_keys => [ "openId", "openid"]
    }
    urldecode {
      all_fields => true
    }
     mutate {
        add_field => { "read_timestamp" => "%{@timestamp}" }
      }
     date {
          match => [ "[nginx][access][time]", "dd/MMM/YYYY:HH:mm:ss Z" ]
        }
    }
    geoip {
      source => "[nginx][access][forwarded]"
      #target => "[nginx][access][geoip]"
    }
}
output{
  elasticsearch{
    # 添加对应集群的地址
    hosts => ["es-host"]
    index => "nginx-log-%{+yyyy.MM.dd}"
  }
}
```

## ES配置

需要在es中创建相应的索引
```template
PUT /_template/template_nginx?include_type_name=true
{
  "order" : 0,
  "version" : 10001,
  "index_patterns" : [
    "nginx-log-*"
  ],
  "settings" : {
    "index" : {
    "number_of_shards": "5",
    "number_of_replicas": "1",
    "refresh_interval" : "5s"
    }
  },
  "mappings" : {
    "_default_" : {
      "dynamic_templates" : [
        {
          "message_field" : {
            "path_match" : "message",
            "mapping" : {
              "norms" : false,
              "type" : "text"
            },
            "match_mapping_type" : "string"
          }
        },
        {
          "string_fields" : {
            "mapping" : {
              "norms" : false,
              "type" : "text",
              "analyzer": "ik_max_word",
              "fields" : {
                "keyword" : {
                  "ignore_above" : 256,
                  "type" : "keyword"
                }
              }
            },
            "match_mapping_type" : "string",
            "match" : "*"
          }
        }
      ]
}
```
