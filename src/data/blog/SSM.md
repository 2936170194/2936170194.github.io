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



# Spring Day 02 学习笔记

## 1. IOC/DI 配置管理第三方 Bean (XML 方式)
在实际开发中，经常需要将第三方 jar 包中的类交给 Spring 管理（例如数据库连接池） [cite: 8]。

### 1.1 管理数据源 (Druid & C3P0)
* **实现步骤**：
  1. 导入第三方坐标依赖（如 `druid` 或 `c3p0`） [cite: 58-67, 127-136]。
  2. 在 `applicationContext.xml` 中使用 `<bean>` 标签定义对象 [cite: 86-94, 177-189]。
  3. 使用 `<property>` 标签通过 `setter` 方法注入数据库连接四要素（驱动、URL、用户名、密码） [cite: 88-91, 122, 179-186]。

### 1.2 加载 Properties 配置文件
为了避免硬编码，推荐将配置提取到 `.properties` 文件中 [cite: 220-221]。
* **开启命名空间**：在 XML 中添加 `context` 命名空间 [cite: 239-244]。
* **加载文件**：`<context:property-placeholder location="jdbc.properties"/>` [cite: 259-260]。
* **注入属性**：使用 `${key}` 读取文件中的值 [cite: 262, 290-293]。
* **注意事项**：为了防止自定义属性名（如 `username`）与系统环境变量冲突，建议加上 `system-properties-mode="NEVER"` 属性 [cite: 415, 440-442]。

---

## 2. 核心容器
### 2.1 容器创建与 Bean 获取
* **创建容器**：
  * `ClassPathXmlApplicationContext`（推荐）：从类路径加载 XML [cite: 588-589]。
  * `FileSystemXmlApplicationContext`：从文件系统绝对路径加载 XML [cite: 592-593]。
* **获取 Bean 的三种方式**：
  1. `ctx.getBean("id")`：按名称获取，需类型强转 [cite: 614-615]。
  2. `ctx.getBean("id", Class.class)`：按名称和类型获取 [cite: 617-618]。
  3. `ctx.getBean(Class.class)`：按类型获取（要求容器中该类型 Bean 唯一） [cite: 620-622]。

### 2.2 容器类层次与加载机制
* **BeanFactory**：顶层接口，**延迟加载**（获取 Bean 时才创建） [cite: 650, 687]。
* **ApplicationContext**：核心接口，**立即加载**（容器启动时预先创建） [cite: 651, 688]。

---

## 3. IOC/DI 注解开发（核心重点）
Spring 3.0 开启了纯注解开发模式，使用 Java 类完全替代 XML 配置文件 [cite: 972-973]。

### 3.1 注解定义 Bean
* **核心注解**：
  * `@Component`：将类标记为 Spring 组件 [cite: 851, 970]。
  * **衍生注解**：`@Controller`（表现层）、`@Service`（业务层）、`@Repository`（数据层），作用与 `@Component` 完全相同，仅用于增强代码可读性 [cite: 950, 968-970]。
* **默认名称**：若不指定 Bean 名称，默认为类名首字母小写 [cite: 947]。

### 3.2 纯注解配置类
* `@Configuration`：标识当前类为 Spring 配置类（替代 XML 文件） [cite: 984, 1044]。
* `@ComponentScan("包路径")`：开启组件扫描（替代 `<context:component-scan>`） [cite: 989-991, 1046]。
* **启动容器**：使用 `AnnotationConfigApplicationContext(配置类.class)` [cite: 1001, 1042]。

### 3.3 Bean 的作用范围与生命周期
* `@Scope`：定义作用范围，默认 `singleton`（单例），可改为 `prototype`（多例） [cite: 1128, 1148]。
* `@PostConstruct`：标在方法上，设定初始化方法 [cite: 1179, 1233]。
* `@PreDestroy`：标在方法上，设定销毁方法 [cite: 1198, 1235]。

### 3.4 注解依赖注入 (DI)
* **`@Autowired`**：按**类型**自动注入引用数据类型（可用于属性或 setter 方法上，推荐直接写在私有属性上，依靠反射赋值） [cite: 1345, 1366-1370, 1521]。
* **`@Qualifier("bean名称")`**：当同类型 Bean 有多个时，配合 `@Autowired` 按**名称**注入 [cite: 1429-1435, 1449]。
* **`@Value`**：注入简单类型（基本数据类型或字符串），常配合 `${key}` 读取配置文件 [cite: 1465-1468, 1499-1501]。
* **`@PropertySource("classpath:文件名")`**：在配置类上加载外部 properties 文件（不支持通配符 `*`） [cite: 1488, 1516-1519]。

---

## 4. IOC/DI 注解管理第三方 Bean
无法修改第三方源码贴 `@Component` 时，需使用 `@Bean` 方法 [cite: 1529-1530]。

* **`@Bean`**：添加在配置类的方法上，将方法返回值作为 Bean 交给 Spring 管理 [cite: 1623, 1747]。
* **`@Import`**：在主配置类上导入其他专门的配置类（如 `JdbcConfig`），形成清晰的模块化配置 [cite: 1723, 1729]。
* **第三方 Bean 资源注入**：
  * **简单类型**：在配置类中定义成员变量使用 `@Value` 注入，然后在 `@Bean` 方法中使用 [cite: 1821-1833]。
  * **引用类型**：直接为 `@Bean` 方法设置形参，Spring 会自动按类型去容器中寻找匹配的 Bean 进行注入 [cite: 1885-1894, 1901]。

---

## 5. Spring 整合 MyBatis
整合核心思想：将 MyBatis 中用到的对象交给 Spring 管理 [cite: 2189]。

### 5.1 核心配置步骤
1. **导入依赖**：需增加 `spring-jdbc` 和 `mybatis-spring`（整合包） [cite: 2232-2236, 2243-2254]。
2. **配置数据源**：在 `JdbcConfig` 中使用 `@Bean` 提供 `DruidDataSource` [cite: 2266-2272, 2296-2302]。
3. **管理 SqlSessionFactory**：在 `MybatisConfig` 中配置 `SqlSessionFactoryBean`，并为其注入数据源和别名包扫描路径 [cite: 2314-2318, 2374-2377]。
4. **管理 Mapper 接口**：配置 `MapperScannerConfigurer`，指定 Dao 接口扫描路径，底层会自动创建代理对象并放入 Spring 容器 [cite: 2339-2344, 2378, 2404-2406]。

---

## 6. Spring 整合 JUnit
使用 Spring 提供的测试运行环境，可以在测试类中直接 `@Autowired` 注入对象 [cite: 2439, 2497-2501]。

* **导入依赖**：`junit` 和 `spring-test` [cite: 2471-2486]。
* **类注解配置**：
  * `@RunWith(SpringJUnit4ClassRunner.class)`：替换 JUnit 默认运行器为 Spring 运行器 [cite: 2491, 2523]。
  * `@ContextConfiguration(classes = 配置类.class)`：指定 Spring 核心配置类所在位置 [cite: 2493, 2521]。