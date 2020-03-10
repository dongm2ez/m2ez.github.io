---
layout: post
title: RPC学习笔记
categories: development
tags: [RPC]
---

## RPC框架

* **Dubbo**：国内最早开源的 RPC 框架，由阿里巴巴公司开发并于 2011 年末对外开源，仅支持 Java 语言。
* **Motan**：微博内部使用的 RPC 框架，于 2016 年对外开源，仅支持 Java 语言。
* **Tars**：腾讯内部使用的 RPC 框架，于 2017 年对外开源，仅支持 C++ 语言。
* **Spring Cloud**：国外 Pivotal 公司 2014 年对外开源的 RPC 框架，仅支持 Java 语言
* **Yar**：PHP轻量的RPC框架，鸟哥开源，仅支持 PHP 语言

跨语言平台RPC

* gRPC：Google 于 2015 年对外开源的跨语言 RPC 框架，支持多种语言。
* Thrift：最初是由 Facebook 开发的内部系统跨语言的 RPC 框架，2007 年贡献给了 Apache 基金，成为 Apache 开源项目之一，支持多种语言。

## gRPC

gRPC是一个高性能开源的通用的RPC框架，可以跨语言使用。

所谓RPC是 `remote procedure call` 的简写，中文翻译为远程过程调用。

RPC框架提供了一套机制，使得应用程序之间可以高效通信，调用远程的接口就像调用本地的函数一样。

![RPC结构图](http://img.m2ez.com/15834709173813.jpg)


RPC是一种CS架构的，RPC也是底层也是基于HTTP协议的，gRPC严格的说是基于HTTP2.0的，2.0的HTTP可以提高接口效率

### gRPC的优势

* gRPC可以通过protobuf来定义接口，从而可以有更加严格的接口约束条件。
* 通过protobuf可以将数据序列化为二进制编码，这会大幅减少需要传输的数据量，从而大幅提高性能。
* gRPC可以方便地支持流式通信

### 使用场景

需要对接口进行严格约束的情况

如我们提供一个公共服务，很多人希望访问这个服务，这时我们希望有更加严格的约束，不希望客户端任意传递数据，尤其是安全性考虑

提高性能

服务之间需要大量的进行数据传递和调用，gRPC将数据压缩编码成二进制的格式，传递的数据要小的很多，而且HTTP2可以实现异步请求，从而提高了通信效率

## Thrift

![Thrift架构图](http://img.m2ez.com/15834974830545.jpg)

Thrift 是一种轻量级的跨语言 RPC 通信方案，支持多达 25 种编程语言。

跟 gRPC 一样，Thrift 也有一套自己的接口定义语言 IDL，可以通过代码生成器，生成各种编程语言的 Client 端和 Server 端的 SDK 代码，这样就保证了不同语言之间可以相互通信。

* 支持多种序列化格式：如 Binary、Compact、JSON、Multiplexed 等。
* 支持多种通信方式：如 Socket、Framed、File、Memory、zlib 等。
* 服务端支持多种处理方式：如 Simple 、Thread Pool、Non-Blocking 等。


## 总结

RPC可以在服务调用的时候调用远程方法像调用本地方法一样，同时相比普通的API调用更加高效。



