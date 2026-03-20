---
title: spring ai实战
author: 李杰
pubDatetime: 2026-03-20T00:00:00Z
featured: false
draft: false
description: spring ai实战笔记
tags: 
 - 后端
 - spring ai
 - 大模型应用开发
---

-----------
可以观看下面的视频:

1.[黑马程序员SpringAI+DeepSeek大模型应用开发实战视频教程，传统Java项目AI化转型必学课程](https://www.bilibili.com/video/BV1MtZnYtEB3?spm_id_from=333.788.videopod.episodes&vd_source=2ca9cbdfe2c1aa2031a5848bdce23d69)

2.[官方笔记](https://my.feishu.cn/wiki/JBq3wsvR6iB5bqkrxqHcQlWynDh)

3.[网盘资料](https://pan.baidu.com/s/1yXjqD9HY4Apc3AwQ0I6HEg&pwd=1234)

-----------

## 认识大模型应用开发
### 1.1大模型应用
大模型应用是基于大模型的推理、分析、生成能力，结合传统编程能力，开发出的各种应用。

为什么要把传统应用与大模型结合呢？

别着急，我们先来看看传统应用、大模型各自擅长什么，不擅长什么

#### 1.1.1 传统应用
作为Java程序员，大家应该对传统Java程序的能力边界很清楚。
核心特点
基于明确规则的逻辑设计，确定性执行，可预测结果。
擅长领域
1. 结构化计算
  - 例：银行转账系统（精确的数值计算、账户余额增减）。
  - 例：Excel公式（按固定规则处理表格数据）。
2. 确定性任务
  - 例：排序算法（快速排序、冒泡排序），输入与输出关系完全可预测。
3. 高性能低延迟场景
  - 例：操作系统内核调度、数据库索引查询，需要毫秒级响应。
4. 规则明确的流程控制
  - 例：红绿灯信号切换系统（基于时间规则和传感器输入）。

不擅长领域
1. 非结构化数据处理
  - 例：无法直接理解用户自然语言提问（如"帮我写一首关于秋天的诗"）。
2. 模糊推理与模式识别
  - 例：判断一张图片是"猫"还是"狗"，传统代码需手动编写特征提取规则，效果差。
3. 动态适应性
  - 例：若用户需求频繁变化（如电商促销规则每天调整），需不断修改代码。


#### 1.1.2 AI大模型
传统程序的弱项，恰恰就是AI大模型的强项：
核心特点
基于数据驱动的概率推理，擅长处理模糊性和不确定性。
擅长领域
1. 自然语言处理
  - 例：ChatGPT生成文章、翻译语言，或客服机器人理解用户意图。
2. 非结构化数据分析
  - 例：医学影像识别（X光片中的肿瘤检测），或语音转文本。
3. 创造性内容生成
  - 例：Stable Diffusion生成符合描述的图像，或AI作曲工具创作音乐。
4. 复杂模式预测
  - 例：股票市场趋势预测（基于历史数据关联性，但需注意可靠性限制）。

不擅长领域
1. 精确计算
  - 例：AI可能错误计算"12345 × 6789"的结果（需依赖计算器类传统程序）。
2. 确定性逻辑验证
  - 例：验证身份证号码是否符合规则（AI可能生成看似合理但非法的号码）。
3. 低资源消耗场景
  - 例：嵌入式设备（如微波炉控制程序）无法承受大模型的算力需求。
4. 因果推理
  - 例：AI可能误判"公鸡打鸣导致日出"的因果关系。

#### 1.1.3 强强联合
传统应用开发和大模型有着各自擅长的领域：
- 传统编程：确定性、规则化、高性能，适合数学计算、流程控制等场景。
- AI大模型：概率性、非结构化、泛化性，适合语言、图像、创造性任务。

两者之间恰好是互补的关系，两者结合则能解决以前难以实现的一些问题：

- 混合系统（Hybrid AI）
  - 用传统程序处理结构化逻辑（如支付校验），AI处理非结构化任务（如用户意图识别）。
  - 示例：智能客服中，AI理解用户问题，传统代码调用数据库返回结果。
- 增强可解释性
  - 结合规则引擎约束AI输出（如法律文档生成时强制符合条款格式）。
- 低代码/无代码平台
  - 通过AI自动生成部分代码（如GitHub Copilot），降低传统开发门槛。

在传统应用开发中介入AI大模型，充分利用两者的优势，既能利用AI实现更加便捷的人机交互，更好的理解用户意图，又能利用传统编程保证安全性和准确性，强强联合，这就是大模型应用开发的真谛！

综上所述，大模型应用就是整合传统程序和大模型的能力和优势来开发的一种应用。

#### 1.1.4 大模型与大模型应用的关系
我们熟知的大模型比如GPT、DeepSeek都是生成式模型，顾名思义，根据前文不断生成后文。

不过，模型本身只具备生成后文的能力、基本推理能力。我们平常使用的AI对话产品除了生成和推理，还有会话记忆功能、联网功能等等。这些都是大模型不具备的。
要想让大模型产生记忆，联网等功能，是需要通过额外的程序来实现的，也就是基于大模型开发应用。

所以，我们现在接触的AI对话产品其实都是基于大模型开发的应用，并不是大模型本身，这一点大家千万要区分清楚。

下面我把常见的一些大模型对话产品及其模型的关系给大家罗列一下：
![大模型对话产品及其模型的关系](../../../public/blog/SpringAI/01.jpg)
当然，除了AI对话应用之外，大模型还可以开发很多其它的AI应用，常见的领域包括：
![大模型应用开发领域](../../../public/blog/SpringAI/02.jpg)

那么问题来了，如何进行大模型应用开发呢？

### 1.2. 大模型应用开发技术架构
基于大模型开发应用有多种方式，接下来我们就来了解下常见的大模型开发技术架构。
#### 1.2.1 技术架构
目前，大模型应用开发的技术架构主要有四种：
![大模型应用开发技术架构](../../../public/blog/SpringAI/03.jpg)

一、 纯Prompt模式
不同的提示词能够让大模型给出差异巨大的答案。
不断雕琢提示词，使大模型能给出最理想的答案，这个过程就叫做提示词工程（Prompt Engineering）。

很多简单的AI应用，仅仅靠一段足够好的提示词就能实现了，这就是纯Prompt模式。

其流程如图：
![纯Prompt模式](../../../public/blog/SpringAI/04.jpg)

二、 FunctionCalling
大模型虽然可以理解自然语言，更清晰弄懂用户意图，但是确无法直接操作数据库、执行严格的业务规则。这个时候我们就可以整合传统应用于大模型的能力了。

简单来说，可以分为以下步骤：
1. 我们可以把传统应用中的部分功能封装成一个个函数（Function）。
2. 然后在提示词中描述用户的需求，并且描述清楚每个函数的作用，要求AI理解用户意图，判断什么时候需要调用哪个函数，并且将任务拆解为多个步骤（Agent）。
3. 当AI执行到某一步，需要调用某个函数时，会返回要调用的函数名称、函数需要的参数信息。
4. 传统应用接收到这些数据以后，就可以调用本地函数。再把函数执行结果封装为提示词，再次发送给AI。
5. 以此类推，逐步执行，直到达成最终结果。

流程如图：
![FunctionCalling](../../../public/blog/SpringAI/05.jpg)

注意：
并不是所有大模型都支持Function Calling，比如DeepSeek-R1模型就不支持。

三、 RAG
RAG（Retrieval-Augmented Generation）叫做检索增强生成。简单来说就是把信息检索技术和大模型结合的方案。
大模型从知识角度存在很多限制：
- 时效性差：大模型训练比较耗时，其训练数据都是旧数据，无法实时更新
- 缺少专业领域知识：大模型训练数据都是采集的通用数据，缺少专业数据

可能有同学会说， 简单啊，我把最新的数据或者专业文档都拼接到提示词，一起发给大模型，不就可以了。

同学，你想的太简单了，现在的大模型都是基于Transformer神经网络，Transformer的强项就是所谓的注意力机制。它可以根据上下文来分析文本含义，所以理解人类意图更加准确。

但是，这里上下文的大小是有限制的，GPT3刚刚出来的时候，仅支持2000个token的上下文。现在领先一点的模型支持的上下文数量也不超过 200K token，所以海量知识库数据是无法直接写入提示词的。

怎么办呢？

RAG技术正是来解决这一问题的。

RAG就是利用信息检索技术来拓展大模型的知识库，解决大模型的知识限制。整体来说RAG分为两个模块：
- 检索模块（Retrieval）：负责存储和检索拓展的知识库
  - 文本拆分：将文本按照某种规则拆分为很多片段
  - 文本嵌入（Embedding)：根据文本片段内容，将文本片段归类存储
  - 文本检索：根据用户提问的问题，找出最相关的文本片段
- 生成模块（Generation）：
  - 组合提示词：将检索到的片段与用户提问组织成提示词，形成更丰富的上下文信息
  - 生成结果：调用生成式模型（例如DeepSeek）根据提示词，生成更准确的回答

由于每次都是从向量库中找出与用户问题相关的数据，而不是整个知识库，所以上下文就不会超过大模型的限制，同时又保证了大模型回答问题是基于知识库中的内容，完美！

流程如图：
![RAG](../../../public/blog/SpringAI/06.jpg)





四、 Fine-tuning
Fine-tuning就是模型微调，就是在预训练大模型（比如DeepSeek、Qwen）的基础上，通过企业自己的数据做进一步的训练，使大模型的回答更符合自己企业的业务需求。这个过程通常需要在模型的参数上进行细微的修改，以达到最佳的性能表现。

在进行微调时，通常会保留模型的大部分结构和参数，只对其中的一小部分进行调整。这样做的好处是可以利用预训练模型已经学习到的知识，同时减少了训练时间和计算资源的消耗。微调的过程包括以下几个关键步骤：
- 选择合适的预训练模型：根据任务的需求，选择一个已经在大量数据上进行过预训练的模型，如Qwen-2.5。
- 准备特定领域的数据集：收集和准备与任务相关的数据集，这些数据将用于微调模型。
- 设置超参数：调整学习率、批次大小、训练轮次等超参数，以确保模型能够有效学习新任务的特征。
- 训练和优化：使用特定任务的数据对模型进行训练，通过前向传播、损失计算、反向传播和权重更新等步骤，不断优化模型的性能。

模型微调虽然更加灵活、强大，但是也存在一些问题：
- 需要大量的计算资源
- 调参复杂性高
- 过拟合风险

总之，Fine-tuning成本较高，难度较大，并不适合大多数企业。而且前面三种技术方案已经能够解决常见问题了。

那么，问题来了，我们该如何选择技术架构呢？

1.2.2 技术选型
从开发成本由低到高来看，四种方案排序如下：
  Prompt < Function Calling < RAG < Fine-tuning

所以我们在选择技术时通常也应该遵循"在达成目标效果的前提下，尽量降低开发成本"这一首要原则。然后可以参考以下流程来思考：
![技术选型流程](../../../public/blog/SpringAI/07.jpg)

- 大模型应用开发框架
![大模型应用开发框架](../../../public/blog/SpringAI/08.jpg)

## SpringAI对话机器人
### 2.1 创建项目
jdk17
maven3.9.14
创建boot项目时添加依赖Spring Web、MySQL Driver、ollama;
为了方便后续开发，我们再手动引入一个Lombok依赖(创建项目时引入有bug、lombok会不起作用)：
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
</dependency>
```
### 2.2 快速入门
1.配置模型:
![快速入门](../../../public/blog/SpringAI/09.jpg)
```yaml
//resources/application.yaml文件
spring:
  application:
    name: heima-ai
  ai:
    ollama:
        base-url: http://localhost:11434
        chat: # 注释:选择chat类型的模型
          model: gemma3:4b

```
2.新增配置类:
```java
package com.itheima.ai.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.ollama.OllamaChatModel;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration //配置类
public class CommonConfiguration {
    @Bean //Bean注解，将方法返回的对象添加到Spring容器中
    public ChatClient chatClient(OllamaChatModel model) {
        return ChatClient
                .builder(model)
                .defaultSystem("你是一个人工智能助手，名字叫做小团团，请用小团团的身份和语气回答用户的问题。")
                .build();
    }
}

```
3.新增接口:
```java
package com.itheima.ai.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.reactive.result.condition.ProducesRequestCondition;

import lombok.RequiredArgsConstructor;
import reactor.core.publisher.Flux;

@RequiredArgsConstructor //RequiredArgsConstructor注解，用于在构造函数中注入依赖，这里注入ChatClient
@RestController
@RequestMapping("/ai")
public class ChatController {

    private final ChatClient chatClient;
    //阻塞式返回对话结果
    @RequestMapping("/chat0")
    public String chatBlock(String prompt) {
        return chatClient.prompt(prompt)
                .user(prompt)
                .call()
                .content();
    }

    //流式返回对话结果
    @RequestMapping(value = "/chat1", produces = "text/html;charset=UTF-8")//需要用produces = "text/html;charset=UTF-8"将返回的流式数据转换为HTML格式、UTF-8编码，否则会乱码
    public Flux<String> chat(String prompt) {
        return chatClient.prompt(prompt)
                .user(prompt)
                .stream()
                .content();
    }

}

```
测试链接:http://localhost:8080/ai/chat1?prompt=%E4%BD%A0%E6%98%AF%E8%B0%81%EF%BC%9F 测试前需启动项目和ollama服务
### 2.3 会话日志
![10.jpg](../../../public/blog/SpringAI/10.jpg)

```java
@Configuration //配置类
public class CommonConfiguration {
    @Bean //Bean注解，将方法返回的对象添加到Spring容器中
    public ChatClient chatClient(OllamaChatModel model) {
        return ChatClient
                .builder(model)
                .defaultSystem("你是一个人工智能助手，名字叫做小团团，请用小团团的身份和语气回答用户的问题。")
                .defaultAdvisors(new SimpleLoggerAdvisor())//添加日志记录器，用于记录对话日志
                .build();
    }
}
```
```yaml
spring:
  application:
    name: heima-ai
  ai:
    ollama:
        base-url: http://localhost:11434
        chat: # 注释:选择chat类型的模型
          model: gemma3:4b
logging: //日志配置
    level: //日志级别
      "[org.springframework.ai.chat.client.advisor]": debug # AI对话的日志级别
      "[com.itheima.ai]": debug # 本项目的日志级别

```
### 2.4 对接前端
新增一个配置类，用来实现前端到后端的跨域请求
```java
package com.itheima.ai.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfigration implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")// 对所有接口路径生效
                .allowedOrigins("*")// 允许任何域名的前端访问
                .allowedMethods("*")// 允许 GET、POST、PUT、DELETE 等所有 HTTP 方法
                .allowedHeaders("*");// 允许所有请求头
    }
}

```
### 2.5 会话记忆
![11.jpg](../../../public/blog/SpringAI/11.jpg)
- ChatMemoryRepository是SpringAI提供的会话记忆存储接口，强调一下，这个不是会话历史。因为它每次保存会话都会删除旧的会话。

- ChatMemoryRepository有很多种实现方式，也就是说你可以用不同的方式来存储会话记忆。例如：
  - InMemoryChatMemoryRepository：基于内存存储，底层是ConcurrentHashMap，默认方案
  - JdbcChatMemoryRepository：基于JDBC在关系数据库中存储，支持多种数据库
  - CassandraChatMemoryRepository：基于Apache Cassandra 存储消息。

下面是采用InMemoryChatMemoryRepository的实现：

定义会话存储方式
```java
    // ✅ 1. 用接口（关键！）
    @Bean
    public ChatMemoryRepository chatMemoryRepository() {
        return new InMemoryChatMemoryRepository();
    }

    // ✅ 2. 解耦 repository
    @Bean
    public ChatMemory chatMemory(ChatMemoryRepository chatMemoryRepository) {
        return MessageWindowChatMemory.builder()
                .chatMemoryRepository(chatMemoryRepository)
                .maxMessages(20)//最大消息数
                .build();
    }
```
配置会话记忆Advisor
```java
@Bean //Bean注解，将方法返回的对象添加到Spring容器中
public ChatClient chatClient(OllamaChatModel model) {
    return ChatClient
            .builder(model)
            .defaultSystem("你是一个人工智能助手，名字叫做小团团，请用小团团的身份和语气回答用户的问题。")
            .defaultAdvisors(
                    new SimpleLoggerAdvisor(),//添加日志记录器，用于记录对话日志
                    MessageChatMemoryAdvisor.builder(chatMemory).build()//添加会话记忆，用于存储会话历史
                )
            .build();
}
```
添加会话id
```java
    @RequestMapping(value = "/chat", produces = "text/html;charset=UTF-8")
    public Flux<String> chat(String prompt, String chatId) {
        return chatClient.prompt(prompt)
                .user(prompt)
                .advisors(as -> as.param(ChatMemory.CONVERSATION_ID, chatId))//添加会话id，用于存储会话历史
                .stream()
                .content();
    }
```
