---
title: SSM
author: 李杰
pubDatetime: 2026-06-30T00:00:00Z
featured: false
draft: false
description: SSM学习
tags: 
 - 后端
 - SSM
 - spring
 - SpringMVC
 - MyBatisPlus
---



# 🚀 Spring Framework 核心基础 (Day 01) 精华总结

## 一、 为什么学 Spring？
* **痛点**：传统 Java Web 开发中，业务类中充斥着 `new` 对象的操作，导致代码**高耦合**，牵一发而动全身。
* **解药**：Spring 提供了**核心容器（Core Container）**，通过控制反转和依赖注入，实现全方位的代码解耦；此外，它还提供了 AOP（面向切面编程）和极强的第三方框架整合能力。

---

## 二、 核心灵魂概念：IOC 与 DI
* **IOC (控制反转)**：将对象的**创建权**从程序代码内部（`new`）交出去，转移给外部的 Spring 容器。
* **DI (依赖注入)**：在 Spring 容器内部，将有相互依赖关系的对象（比如 Service 和 DAO）动态绑定在一起的过程。
* **Bean**：在 Spring 的 IOC 容器中被创建、管理的所有对象，统称为 Bean。

---

## 三、 Bean 的基础配置与作用范围 (Scope)
* **基础配置**：在 XML 中使用 `<bean id="唯一标识" class="全限定类名"/>` 进行注册。
* **别名配置**：使用 `name="别名1, 别名2"` 为 Bean 起小名（多个别名可用逗号、分号或空格隔开）。
* **作用范围 (Scope)**：
    * `singleton` (默认)：**单例模式**。整个容器只有一个实例，性能高。适合无状态对象（如 Service、DAO）。
    * `prototype`：**非单例（多例）模式**。每次获取都会创建一个新实例。

---

## 四、 Bean 的实例化 (造对象的四种方式)
1. **构造方法实例化（最常用，占 95%）**：
    * 底层原理：Spring 默认通过反射调用类的**无参构造方法**创建对象。如果重写了有参构造，务必补上无参构造。
2. **静态工厂实例化（兼容老系统）**：
    * 配置：`<bean id="..." class="工厂类全名" factory-method="静态方法名"/>`。
3. **实例工厂实例化（了解即可）**：
    * 配置：需要先配置工厂 Bean，再配置目标 Bean 并指定 `factory-bean` 和 `factory-method`。
4. **FactoryBean（大厂整合第三方框架的绝对核心）**：
    * 实现：自定义类实现 `FactoryBean<T>` 接口，重写 `getObject()`（编写复杂创建逻辑）和 `getObjectType()` 方法。
    * 优势：极大地简化了复杂对象的 XML 配置，只配一行即可自动识别为工厂。

---

## 五、 Bean 的生命周期 (生老病死)
**1. 完整流程**：
实例化 (分配内存) ➡️ 依赖注入 (执行 set 方法) ➡️ **初始化** ➡️ 业务运行 ➡️ **销毁**。
*(注意：必定是先注入属性，再执行初始化 `afterPropertiesSet`)*

**2. 生命周期控制 (如何插手？)**：
* **XML 配置法**：在 `<bean>` 标签中指定 `init-method="方法名"` 和 `destroy-method="方法名"`。
* **接口法**：让类实现 `InitializingBean` (`afterPropertiesSet`) 和 `DisposableBean` (`destroy`) 接口。

**3. 优雅关闭容器 (触发销毁方法)**：
* 使用实现类 `ClassPathXmlApplicationContext` 独有的方法：
    * `ctx.close()`：立刻暴力关闭。
    * `ctx.registerShutdownHook()`：注册虚拟机的关闭钩子，在系统退出前自动优雅关闭。

---

## 六、 依赖注入 (DI) 的实战方式

### 1. Setter 注入（企业最常用，特别是可选依赖）
* **前提**：必须在类中提供对应属性的 `setter` 方法。
* **配置 (引用类型)**：`<property name="属性名" ref="Bean的ID"/>`
* **配置 (简单类型)**：`<property name="属性名" value="具体值"/>` (Spring 会自动转换基本数据类型和 String)。

### 2. 构造器注入（推荐用于强制依赖）
* **前提**：必须在类中提供带参构造方法。
* **配置**：`<constructor-arg name="参数名" ref="Bean的ID" />` (也可以用 `value` 注入简单类型)。
* *进阶*：解决参数名耦合，可替换为 `type` 或 `index` 属性按类型或索引注入。

### 3. 自动装配 (Autowire - 简化 XML 标签)
* **配置**：在 `<bean>` 标签上添加 `autowire="byType"`（按类型自动寻找并注入，最常用）或 `autowire="byName"`。
* **限制**：只能用于引用类型，且类中必须有 Setter 方法。如果通过 `byType` 找到了多个相同类型的 Bean，系统会报错 `NoUniqueBeanDefinitionException`。
* **注意**：自动装配优先级低于 setter 注入与构造器注入。

---

## 七、 集合注入 (批量塞数据)
当属性是数组、List、Set、Map 或 Properties 时：
* 使用 Setter 注入，在 `<property>` 标签内部嵌套具体的集合标签。
* **数组/List**：`<array>` 或 `<list>`，内部写 `<value>`（两者底层都是数组，可混用）。
* **Set**：`<set>`，内部写 `<value>` (自带去重)。
* **Map**：`<map>`，内部写 `<entry key="..." value="..."/>`。
* **Properties**：`<props>`，内部写 `<prop key="...">值</prop>`。
* *提示：如果集合里装的是引用类型（其他 Bean），把 `<value>` 换成 `<ref bean="..."/>` 即可。这些集合标签同样也可以写在 `<constructor-arg>` 内部。*