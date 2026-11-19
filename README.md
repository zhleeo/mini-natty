# mini-netty

[![Java](https://img.shields.io/badge/Java-17%2B-blue)](https://openjdk.org/)
[![Maven](https://img.shields.io/badge/Maven-3.6%2B-orange)](https://maven.apache.org/)

一个用于学习 **Netty 核心原理** 的简化版网络编程框架。基于 JDK 原生 NIO 从零实现，通过一系列递进的步骤逐步引入网络编程的核心概念，从而深入理解 Netty 内部的关键设计，而不是直接调用 Netty 的 API。

## 目录

- [项目介绍](#项目介绍)
- [学习路径](#学习路径)
- [学习建议](#学习建议)
- [快速开始](#快速开始)
- [核心组件](#核心组件)
- [技术栈](#技术栈)
- [延伸阅读](#延伸阅读)
- [版本](#版本)

## 项目介绍

**mini-netty** 通过「复刻 → 对比 → 理解」的方式，掌握 Netty 的核心设计思想：

- 📚 **渐进式学习**：分步骤推进，每个步骤对应一个可独立运行的 Git 分支。
- 🎯 **零运行时依赖**：纯 JDK NIO 实现，不依赖 Netty 框架本身（测试仅用 JUnit 5）。
- 🏗️ **真实架构**：概念划分与命名对齐 Netty，便于对照源码。
- 🔬 **测试驱动**：每个步骤配套单元测试。

### 架构概览

```
                           ┌─────────────────────────────────────┐
                           │           ServerBootstrap           │
                           └──────────────────┬──────────────────┘
                                              │
                    ┌─────────────────────────┼─────────────────────────┐
                    │                         │                         │
           ┌────────┴────────┐      ┌────────┴────────┐      ┌────────┴────────┐
           │  BossGroup (1)  │      │ WorkerGroup (N) │      │  ChannelOption  │
           └────────┬────────┘      └────────┬────────┘      └─────────────────┘
                    │                        │
           ┌────────┴────────┐      ┌────────┴────────┐
           │  NioEventLoop   │      │  NioEventLoop   │ × N
           │  (Selector)     │      │  (Selector)     │
           └────────┬────────┘      └────────┬────────┘
                    │                        │
           ┌────────┴────────┐      ┌────────┴────────┐
           │ ServerChannel   │─────▶│  SocketChannel  │ × Connections
           └────────┬────────┘      └────────┬────────┘
                    │                        │
           ┌────────┴────────────────────────┴────────┐
           │              ChannelPipeline             │
           │  ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐   │
           │  │HEAD │◀─▶│ H1  │◀─▶│ H2  │◀─▶│TAIL │   │
           │  └─────┘   └─────┘   └─────┘   └─────┘   │
           │    ▼ Inbound                 Outbound ▲  │
           └──────────────────────────────────────────┘
```

### 项目结构

```
src/main/java/com/learn/netty/
├── bootstrap/          # 启动器 (ServerBootstrap, Bootstrap)
├── buffer/             # 缓冲区 (ByteBuf, HeapByteBuf)
├── channel/            # 核心通道
│   ├── nio/            # NIO 实现 (NioEventLoop, NioChannel)
│   ├── Channel.java    # 通道接口
│   ├── ChannelPipeline.java
│   ├── ChannelHandler.java
│   └── EventLoop.java
└── handler/            # 内置处理器
    ├── codec/          # 编解码器
    ├── idle/           # 空闲检测处理器
    └── logging/        # 日志处理器
```

> 类名与职责划分对齐 Netty 的命名习惯，但包路径使用本项目实际的 `com.learn.netty`，避免与 `io.netty` 冲突。

## 学习路径

学习路径分 8 个阶段、共 37 个步骤，循序渐进覆盖从阻塞式 I/O 到主从 Reactor 多线程模型的完整链路。

### 阶段总览

| 阶段 | 主题 | 步骤 | 核心内容 |
|------|------|------|----------|
| 1 | 网络通信基础 | STEP-01 ~ STEP-07 | BIO → NIO 多路复用 |
| 2 | 事件循环 | STEP-08 ~ STEP-11 | 单线程 EventLoop |
| 3 | 通道与管道 | STEP-12 ~ STEP-21 | Channel + Pipeline 责任链 |
| 4 | 字节缓冲区 | STEP-22 ~ STEP-25 | ByteBuf 与引用计数 |
| 5 | 编解码器 | STEP-26 ~ STEP-29 | 粘包拆包与编解码 |
| 6 | 启动器 | STEP-30 ~ STEP-32 | Bootstrap 流式 API |
| 7 | 多线程模型 | STEP-33 ~ STEP-35 | 主从 Reactor |
| 8 | 实用 Handler | STEP-36 ~ STEP-37 | 内置处理器 |

### 第一阶段：网络通信基础 (STEP-01 ~ STEP-07)

从阻塞式 BIO 起步，过渡到 NIO 多路复用，建立对网络 I/O 模型的基本认知。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-01 | `simple-bio-server` | 用 `ServerSocket` 写一个单连接的服务端 |
| STEP-02 | `simple-bio-client` | 写一个能与服务端通信的客户端 |
| STEP-03 | `multi-thread-bio-server` | 每个连接分配一个线程，处理并发 |
| STEP-04 | `nio-channel-buffer` | 认识 NIO 的 `Channel` 与 `Buffer` |
| STEP-05 | `nio-selector` | 用一个 `Selector` 管理多个 `Channel` |
| STEP-06 | `nio-server-accept` | 在 NIO 服务端处理新连接接入 |
| STEP-07 | `nio-server-read-write` | 打通 NIO 的完整收发流程 |

### 第二阶段：事件循环 EventLoop (STEP-08 ~ STEP-11)

理解单线程事件循环如何驱动并处理多个连接。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-08 | `event-loop-interface` | 定义 `EventLoop` 接口 |
| STEP-09 | `single-thread-event-loop` | 实现单线程事件循环 |
| STEP-10 | `event-loop-task-queue` | 加入任务队列，把耗时操作转成异步任务 |
| STEP-11 | `event-loop-scheduled-task` | 支持定时任务 |

### 第三阶段：通道与管道 Channel / Pipeline (STEP-12 ~ STEP-21)

抽象出 `Channel` 与责任链式的 `Pipeline`。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-12 | `channel-interface` | 定义 `Channel` 接口 |
| STEP-13 | `nio-channel-impl` | 用 NIO 实现 `Channel` |
| STEP-14 | `channel-config` | 给 `Channel` 增加配置项 |
| STEP-15 | `channel-unsafe` | 封装底层不安全的 I/O 操作 |
| STEP-16 | `channel-handler-interface` | 定义 `Handler` 接口 |
| STEP-17 | `channel-pipeline-basic` | 用双向链表串起 `Pipeline` |
| STEP-18 | `channel-handler-context` | 引入 `HandlerContext` 传递事件 |
| STEP-19 | `inbound-handler` | 处理入站事件 |
| STEP-20 | `outbound-handler` | 处理出站事件 |
| STEP-21 | `handler-adapter` | 提供适配器并传播异常 |

### 第四阶段：字节缓冲区 ByteBuf (STEP-22 ~ STEP-25)

实现读写索引分离的缓冲区与引用计数机制。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-22 | `byte-buf-interface` | 定义 `ByteBuf` 接口 |
| STEP-23 | `heap-byte-buf` | 基于堆内存实现 `ByteBuf` |
| STEP-24 | `byte-buf-reference-count` | 引入引用计数与资源释放 |
| STEP-25 | `byte-buf-allocator` | 统一缓冲区分配入口 |

### 第五阶段：编解码器 Codec (STEP-26 ~ STEP-29)

解决 TCP 粘包/拆包，实现字节与业务消息的互转。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-26 | `byte-to-message-decoder` | 字节流转消息的基类 |
| STEP-27 | `fixed-length-decoder` | 固定长度帧解码 |
| STEP-28 | `length-field-decoder` | 基于长度字段的解码 |
| STEP-29 | `string-codec` | 字符串编解码 |

### 第六阶段：启动器 Bootstrap (STEP-30 ~ STEP-32)

用流式 API 封装启动流程。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-30 | `abstract-bootstrap` | 启动器抽象基类 |
| STEP-31 | `server-bootstrap` | 服务端启动器 |
| STEP-32 | `client-bootstrap` | 客户端启动器 |

### 第七阶段：多线程模型 (STEP-33 ~ STEP-35)

实现 Boss-Worker 主从 Reactor 与连接分配策略。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-33 | `event-loop-group` | 线程组统一管理多个 `EventLoop` |
| STEP-34 | `boss-worker-model` | 主从 Reactor 分工 |
| STEP-35 | `channel-chooser` | 连接分配到工作线程的策略 |

### 第八阶段：实用 Handler (STEP-36 ~ STEP-37)

补齐常用的内置处理器。

| 步骤 | 分支名 | 学习目标 |
|------|--------|----------|
| STEP-36 | `idle-state-handler` | 空闲连接检测 |
| STEP-37 | `logging-handler` | 日志记录 |

## 学习建议

1. **按顺序学习**：每个步骤都基于前一个步骤，建议按 STEP-01 → STEP-37 顺序学习。
2. **切换分支**：每个步骤对应一个 Git 分支，使用 `git checkout <分支名>` 切换。
3. **阅读 CHANGELOG**：每个步骤的改动记录在 [CHANGELOG.md](./CHANGELOG.md) 中。
4. **运行测试**：每个步骤配套对应的单元测试。
5. **动手实践**：尝试修改代码、对照 Netty 源码，加深理解。

## 快速开始

### 环境要求

- JDK 17 或更高版本
- Maven 3.6 或更高版本

### 构建项目

```bash
# 编译项目
mvn compile

# 运行测试
mvn test
```

## 核心组件

### EventLoop：事件循环

```java
EventLoopGroup group = new NioEventLoopGroup(4);
EventLoop eventLoop = group.next();

// 提交任务
eventLoop.execute(() -> System.out.println("Hello from EventLoop"));

// 定时任务
eventLoop.schedule(() -> System.out.println("Delayed"), 1, TimeUnit.SECONDS);
```

### Channel & Pipeline：通道和管道

```java
Channel channel = ...;
ChannelPipeline pipeline = channel.pipeline();

// 添加处理器
pipeline.addLast("decoder", new StringDecoder());
pipeline.addLast("encoder", new StringEncoder());
pipeline.addLast("handler", new MyHandler());
```

### ByteBuf：缓冲区

```java
ByteBufAllocator allocator = UnpooledByteBufAllocator.DEFAULT;
ByteBuf buf = allocator.buffer(256);

// 写入数据
buf.writeBytes("Hello".getBytes());
buf.writeInt(42);

// 读取数据
byte[] bytes = new byte[5];
buf.readBytes(bytes);
int value = buf.readInt();

// 释放资源
buf.release();
```

### ServerBootstrap：服务端启动器

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup(4);

ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
 .option(ChannelOption.SO_BACKLOG, 128)
 .childOption(ChannelOption.SO_KEEPALIVE, true)
 .childHandler(new ChannelInitializer<Channel>() {
     @Override
     protected void initChannel(Channel ch) {
         ch.pipeline().addLast(new LoggingHandler());
         ch.pipeline().addLast(new EchoServerHandler());
     }
 });

ChannelFuture f = b.bind(8080).sync();
```

## 技术栈

| 类别 | 技术 |
|------|------|
| 语言 | Java 17 |
| 构建工具 | Maven 3.6+ |
| 底层 I/O | JDK NIO（`Selector` / `ServerSocketChannel`） |
| 测试框架 | JUnit 5 |

## 延伸阅读

- [Netty Wiki](https://netty.io/wiki/)：官方文档与用户指南
- [Netty 源码](https://github.com/netty/netty)：对照源码深入理解
- [Netty in Action](https://www.manning.com/books/netty-in-action)：推荐书籍，系统讲解 Netty

## 版本

见 [CHANGELOG.md](./CHANGELOG.md)。