---
layout:     post
title:      "关于 XxPay 的整理"
subtitle:   "做最专业的开源聚合支付系统"
date:       2018-08-31 19:41:59
author:     "于磊"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 聚合支付
    - Java

---



### 脑图 【[点我下载](https://github.com/yuleizhuai/resources/blob/master/project/opensource/XxPay/XxPay%E8%84%91%E5%9B%BE.pdf)】

![xxpay](/img/xxpay/XX PAY.jpg)





### 项目结构

xxpay-master

- xxpay-common
  - 公共模块（常量、工具类等），jar发布
- xxpay-dal
  - 支付数据持久层，jar 发布
- xxpay-mgr
  - 端口：8092
  - 支付运营平台
- xxpay-shop
  - 端口：8081
  - 支付商城演示系统
- xxpay4spring-cloud
  - 支付中心 spring-cloud 架构实现
    - xxpay-config
      - 端口：2020
      - 支付服务配置中心
    - xxpay-gateway
      - 端口：3020
      - 支付服务 API 网关
    - xxpay-server
      - 端口：2000
      - 支付服务注册中心
    - xxpay-service
      - 端口：3000
      - 支付服务生产者
    - xxpay-web
      - 端口：3010
      - 支付服务消费者
    - 项目启动顺序
      - xxpay-server
      - xxpay-config
      - xxpay-service
      - xxpay-web
      - xxpay-gateway
- xxpay4dubbo
  - 支付中心 spring-boot-dubbo 架构实现
    - xxpay4dubbo-api
      - API 接口定义
    - xxpay4dubbo-service
      - 端口：20880
      - 支付服务生产者
    - xxpay4dubbo-web
      - 端口：3020
      - 支付服务消费者
    - 项目启动顺序
      - xxpay4dubbo-service
      - xxpay4dubbo-web
- xxpay4springboot
  - 支付中心 spring-mvc 架构实现



### 项目部署

#### 服务器配置

- CPU：1核
- 内存：2GB
- 操作系统：CentOS 6.8 64位

#### 安装软件

- JDK 1.8
  - spring boot 对低版支持没有测过
- ActiveMQ 5.11.1
  - 高版本也可以，如：5.14.3
- MySQL 5.7.17
  - 要在5.6以上，否则初始化 SQL 会报错，除非手动修改建表语句

### 特点

#### 代码开源

- JAVA 开发
- 代码完全开源
- MIT 授权协议

#### 文档完整

- 部署文档
- 开发文档
- 接口文档
- 在线文档库

#### 多种架构

- Spring Cloud
- Dubbo
- Spring Boot
- 三种架构支持分布式微服务部署

#### 功能全面

- 强大的运营管理平台
- 商户管理
- 支付渠道
- 订单管理
- 商户通知

### 介绍

- 做最专业的开源聚合支付系统

- JAVA 开发、分布式架构、微服务

- 易于二次开发、可直接用于生产

- 已接入微信、支付宝等主流支付渠道，可直接用于生产环境

- 目前已经接入支付渠道：微信（公众号支付、扫码支付、APP 支付、H5支付）、支付宝（电脑网站支付、手机网站支付、APP 支付、当面付）

### Alipay 下单时序图

![Alipay_Create_Order](/img/xxpay/AliPay_Create_Order_Sequence_Diagram.png)

