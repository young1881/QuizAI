## QuizAI · AI 驱动的答题与问卷生成平台（后端）

> 作者：wortox  
> 技术栈：Spring Boot / MyBatis-Plus / Redis / Caffeine / Sharding-JDBC / RxJava / ChatGLM / Redisson / Swagger

QuizAI 是一个 **AI 驱动的在线答题与问卷生成平台**。用户可以创建答题应用、配置题目与评分策略，平台支持基于规则和大模型的多种评分方式，并提供实时答题与结果反馈能力。

---

### 核心亮点

- **评分模块采用策略模式**
  - 针对统计得分、AI 评分等多种评分算法，抽象统一策略接口。
  - 通过自定义注解 + 全局执行器自动扫描策略类，并在运行时按应用配置动态选择评分策略。
  - 相比传统 if-else 分支，大幅提升系统的 **可扩展性** 和 **可维护性**。

- **AI 生成题目流式推送**
  - 针对 AI 生成题目延迟较长的问题，基于 **ChatGLM 流式 API + RxJava 操作符链式调用** 高效处理异步数据流。
  - 后端通过 **SSE（Server-Sent Events）** 将每一道生成完成的题目实时推送给前端，显著提升答题编辑体验。

- **评分性能优化**
  - 使用 **Caffeine 本地缓存** 存储用户答题数据，将评分查询延迟从约 10 秒优化到约 5 ms。
  - 有效降低数据库查询压力，并通过 **Redisson 分布式锁** 防止缓存击穿，保障高并发场景下的稳定性。

---

### 后端技术栈与功能概览

- **基础框架**
  - Spring Boot 2.7.x
  - Spring MVC
  - MyBatis + MyBatis-Plus（分页、条件构造器）
  - Spring AOP、Spring Scheduler、声明式事务

- **数据与基础设施**
  - MySQL：核心业务数据存储（应用、题目、用户、答题记录、评分结果等）
  - Redis：分布式登录、限流与缓存支撑
  - Caffeine：热点答题数据本地缓存
  - Sharding-JDBC：对答题记录等高并发表进行分库分表（按应用 ID 取模）以提升查询性能
  - Elasticsearch（可选）：复杂检索场景支持
  - 对象存储（COS 等）：题目配图、附件等静态资源存储

- **通用能力**
  - 全局异常处理与统一错误码
  - 统一响应封装
  - 登录鉴权与权限校验
  - 全局请求日志与跨域配置
  - Swagger + Knife4j 在线接口文档

---

### 核心业务模块

- **应用管理**
  - 创建 / 编辑 / 上线答题应用
  - 配置评分策略（统计评分、AI 评分等）和题目生成方式

- **题目管理**
  - 手工创建题目（选择题、评分题等）
  - 基于 ChatGLM 的 AI 批量生成题目（流式输出）

- **答题与评分**
  - 用户在线答题，后端为每次答题生成全局唯一 ID（雪花算法）
  - 根据应用绑定的策略自动路由到对应评分实现模块
  - 支持同步/异步评分与结果查询

- **结果统计**
  - 按应用、用户维度统计得分与结果分布
  - 为前端提供可视化统计接口

---

### 快速开始（后端）

1. **克隆项目**

```bash
git clone <your-repo-url>
cd QuizAI/backend
```

2. **配置 MySQL**

- 在本地创建数据库（名称自定，如 `quizai`），执行 `sql/create_table.sql` 和 `sql/init_data.sql` 初始化表结构与基础数据。
- 修改 `src/main/resources/application.yml` 中的数据源配置为你的本地环境：

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/quizai?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
```

3. **配置 Redis / 缓存等（可按需调整）**

```yml
spring:
  redis:
    host: localhost
    port: 6379
    database: 1
    timeout: 5000
```

4. **启动项目**

- 使用 IDE（如 IDEA）运行 `MainApplication`，或使用 Maven 命令：

```bash
mvn spring-boot:run
```

- 启动成功后，可通过浏览器访问接口文档（示例）：
  - `http://localhost:8101/api/doc.html`

---

### 前端与整体联调简述

- 前端位于 `frontend` 目录，基于 **Vue 3 + TypeScript + Ant Design Vue**（可从代码结构中查看实际使用的组件库）。
- 默认通过环境配置将请求代理到后端（如 `http://localhost:8101`），可在 `frontend/vue.config.js` 中调整。
- 启动步骤（示例）：

```bash
cd ../frontend
npm install
npm run serve
```

访问前端页面后，即可体验 QuizAI 的答题创建、AI 生成题目、在线答题与评分等完整流程。

---

### 目录结构（后端简要）

- `src/main/java/com/wortox/quizai`：后端业务代码
  - `controller`：REST 接口层
  - `service`：业务服务与评分策略实现
  - `manager`：与第三方（AI、大数据等）的集成封装
  - `model`：DTO / VO / 实体对象
  - `config`：全局配置（跨域、拦截器等）
- `src/main/resources`
  - `application*.yml`：多环境配置
  - `mapper`：MyBatis XML 映射文件
  - `sql`：建表与初始化脚本

---

### 开发者说明

- 作者：**wortox**
- 如需在此基础上二次开发，可重点关注：
  - 评分策略接口与实现类（扩展新的评分规则）
  - AI 题目生成相关的 manager 与配置（替换为你自己的大模型服务）
  - 分库分表与缓存策略（根据真实业务量级进行调整）

欢迎在 QuizAI 的基础上继续扩展你自己的问卷与测评类应用。 
