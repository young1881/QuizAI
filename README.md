## QuizAI · AI 驱动的答题与问卷生成平台

> 作者：**wortox**  
> 技术栈：**Vue 3 / TypeScript / Ant Design 风格组件 / Spring Boot / MyBatis-Plus / Redis / Caffeine / Sharding-JDBC / RxJava / ChatGLM / Redisson / Swagger**

QuizAI 是一个 **AI 驱动的在线答题与问卷生成平台**。  
用户可以快速创建答题应用（如性格测试、知识测验、问卷调查等），并通过配置不同评分策略（规则评分 / AI 评分）和 AI 生成题目能力，实现从“出题 → 答题 → 评分 → 分析”的完整闭环。

---

### 核心亮点

- **灵活可扩展的评分策略模块**
  - 采用 **策略模式** 抽象统一评分接口，内置统计得分、AI 打分等多种实现。
  - 通过 **自定义注解 + 全局执行器** 扫描策略类，在运行时根据应用配置 **动态选择评分策略**。
  - 相比大量 if-else 分支，显著提升 **可扩展性** 和 **可维护性**，便于后续新增评分规则。

- **AI 生成题目的流式体验**
  - 结合 **ChatGLM 流式 API** 与 **RxJava 操作符链式调用**，高效处理大模型生成题目的异步数据流。
  - 后端通过 **SSE（Server-Sent Events）** 实时向前端推送每一道生成完成的题目，用户无需等待全部题目生成，大幅提升编辑体验。

- **高性能评分与缓存设计**
  - 使用 **Caffeine 本地缓存** 存储用户答案，将评分查询延迟从约 **10s 降至 5ms**。
  - 减少数据库压力，同时利用 **Redisson 分布式锁** 防止缓存击穿，在高并发场景下保持稳定。

- **面向增长的存储与扩展性设计**
  - 基于 **Sharding-JDBC** 对答题记录等高并发表进行分表（按应用 ID 取模），提升单表查询性能。
  - 通过合理的分层与配置抽象，支持后续横向扩展与多机部署。

---

### 功能概览

- **应用管理**
  - 创建 / 编辑 / 上线答题应用
  - 配置题目来源（手工 / AI 生成）、评分策略、展示信息等

- **题目管理**
  - 支持选择题、打分题等多种题型
  - 一键调用 AI 批量生成题目，支持实时预览与调整

- **答题 & 评分**
  - 用户在线答题，系统基于 **雪花算法** 为每次答题生成全局唯一 ID。
  - 根据应用绑定的评分策略自动路由到对应实现模块，支持同步 / 异步评分。

- **结果与统计**
  - 查看单次答题结果、历史记录
  - 统计不同应用的答题量、通过率、得分分布等，为后续运营与优化提供数据基础。

---

### 项目结构

```text
QuizAI/
├── backend/    # 后端：Spring Boot + MyBatis-Plus 等
└── frontend/   # 前端：Vue 3 + TypeScript
```

- **backend**（后端）
  - 基于 Spring Boot 2.7.x，采用分层架构（controller / service / manager / mapper / model 等）。
  - 整合 MyBatis-Plus、Redis、Sharding-JDBC、Caffeine、Redisson、Swagger 等常用组件。
  - 通过注解 + AOP 实现权限校验、全局异常处理、统一返回封装等通用能力。

- **frontend**（前端）
  - 使用 Vue 3 + TypeScript + Vue Router + Pinia。
  - 布局与组件风格接近 Ant Design，提供首页、应用列表、答题页面、答题结果页、管理后台等视图。
  - 与后端通过 REST API / SSE 交互，支持 AI 题目流式展示。

---

### 快速开始

#### 1. 环境准备

- Node.js（建议 ≥ 16）
- JDK 8
- Maven（可使用项目自带的 `mvnw` / `mvnw.cmd`）
- MySQL / Redis

---

#### 2. 后端启动（backend）

1. **创建数据库并初始化数据**

   - 创建数据库（例如）：
     - 名称：`quizai`
   - 执行脚本：
     - `backend/sql/create_table.sql`
     - `backend/sql/init_data.sql`

2. **配置数据库与 Redis**

   修改 `backend/src/main/resources/application.yml` 中相关配置：

   ```yml
   spring:
     datasource:
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://localhost:3306/quizai?useSSL=false&serverTimezone=Asia/Shanghai
       username: root
       password: 
     redis:
       host: localhost
       port: 6379
       database: 1
   ```

3. **运行后端服务**

   在 `backend` 目录下执行：

   ```bash
   # Windows
   .\mvnw.cmd spring-boot:run

   # 或者全平台通用（需本机已有 mvn）
   mvn spring-boot:run
   ```

4. **访问接口文档**

   后端启动成功后，可在浏览器访问（具体端口见 `application.yml`，默认 8101）：

   - `http://localhost:8101/api/doc.html`

---

#### 3. 前端启动（frontend）

1. 安装依赖：

   ```bash
   cd frontend
   npm install
   ```

2. 启动开发服务：

   ```bash
   npm run serve
   ```

3. 在浏览器中访问：

   - 通常为 `http://localhost:8080`

前端会通过代理或环境配置将接口请求转发到后端（默认 `http://localhost:8101`），可在 `frontend/vue.config.js` 中按需修改。

---

### 部署说明（简要）

- **后端**
  - 支持使用 Docker 镜像部署，示例见 `backend/Dockerfile`。
  - 使用 Maven 构建后，将生成的 `quizai-backend-0.0.1-SNAPSHOT.jar` 放入容器运行。

- **前端**
  - 通过 `npm run build` 生成静态文件，可部署至 Nginx 等静态资源服务器。
  - 示例 Nginx 配置参考 `frontend/docker/nginx.conf`。




