# Talker 自定义日志架构设计

## 背景

仓库当前已经通过 `packages/core/puupee_utilities/lib/talker.dart` 暴露全局 `talker`，并在开发者日志页使用 `TalkerScreen(talker: talker)` 查看日志。`packages/sync/puupee_sync/lib/src/sync_server.dart` 直接持有 `Talker _logger = talker`，并大量调用 `info`、`warning`、`error` 等基础日志方法。

这种写法可以输出日志，但所有业务日志会落到 `info`、`warning`、`error` 这些通用 key 下。用户在 Talker UI 中只能按等级筛选，无法单独查看 `sync-server` 相关日志。Talker 5.1.17 已支持通过 `TalkerData.key` 过滤自定义日志，也支持通过 `settings.registerKeys()` 把自定义 key 放入设置页，因此可以在不重写日志 UI 的前提下建立领域日志体系。

## 目标

- 基于 Talker 原生 `TalkerLog` 设计 Puupee 自定义日志模型。
- 让 `sync_server.dart` 的日志统一进入 `sync-server` key，能在现有 TalkerScreen 中单独筛选。
- 保留 Talker 的日志等级、异常、堆栈、console 输出和 history 能力。
- 将通用日志基础设施放在 `puupee_utilities`，避免 sync 包各文件自行拼装 `TalkerLog`。
- 第一阶段只迁移 `sync_server.dart`，建立可验证的最小闭环。
- 为后续 `sync-client`、`sync-storage`、`sync-auth`、`sync-mcp` 等模块扩展留出稳定 API。

## 非目标

- 不重写 `TalkerScreen` 或自研日志控制台。
- 不在第一阶段迁移 `puupee_sync` 的所有文件。
- 不把每个细分场景都做成独立 Talker key，避免筛选项膨胀。
- 不引入除 Talker 之外的新日志依赖。
- 不在日志中保存敏感正文、token、密钥或大体积请求体。

## 推荐方案

采用“模块 key + 场景标签”的结构。

- 模块 key 写入 `TalkerData.key`，用于 Talker 原生筛选，例如 `sync-server`。
- 场景标签写入日志 message 前缀，例如 `[pairing] 处理配对请求失败`。
- 日志等级仍使用 `LogLevel.info`、`LogLevel.warning`、`LogLevel.error` 等。
- 异常与堆栈继续走 `TalkerLog.exception` 和 `TalkerLog.stackTrace`。

这样 Talker UI 可以通过 `sync-server` 一键筛出同步服务器日志，也可以通过搜索 `[pairing]`、`[storage]`、`[http]` 等场景继续缩小范围。

## 架构

### puupee_utilities

新增通用日志基础设施：

- `PuupeeTalkerLog extends TalkerLog`：Puupee 自定义 Talker 日志模型。
- `PuupeeLogger`：业务侧使用的轻量门面，封装 `talker.logCustom(...)`。
- `PuupeeLogModule`：集中定义模块 key，例如 `syncServer`、`syncClient`。
- `PuupeeLogScene`：集中定义通用场景名，例如 `lifecycle`、`http`、`storage`。
- `registerPuupeeTalkerKeys(Talker talker)`：集中注册自定义 key、title 和颜色。

同时修正 `createTalker({settings})`：当前实现忽略了传入的 `settings`，后续应改为把 settings 传给 `Talker`，否则自定义 title/color 配置无法可靠生效。

### puupee_sync

第一阶段只在 `sync_server.dart` 使用：

- `_logger` 类型从 `Talker` 改为 `PuupeeLogger`。
- `_logger` 初始化为 `PuupeeLogger.syncServer()` 或等价工厂。
- 所有日志都通过 `PuupeeLogger` 写入，模块 key 固定为 `sync-server`。
- 重要日志补充 scene；低价值日志可以先使用 `lifecycle` 或保留无 scene，后续再细分。

## 核心模型

`PuupeeTalkerLog` 应基于 Talker 原生字段承载信息：

- `key`：模块 key，例如 `sync-server`。
- `title`：默认等于模块展示名，例如 `sync-server`。
- `logLevel`：Talker 原生日志等级。
- `message`：带 scene 前缀的正文，例如 `[db] Fields: ...`。
- `exception`：异常对象。
- `stackTrace`：异常堆栈。
- `time`：沿用 Talker 默认生成时间。
- `pen`：可选，自定义 console 颜色；优先由 Talker settings 按 key 解析。

`PuupeeLogger` 对业务侧暴露与 Talker 习惯一致的方法：

- `debug(message, {scene, exception, stackTrace})`
- `info(message, {scene, exception, stackTrace})`
- `warning(message, {scene, exception, stackTrace})`
- `error(message, {scene, exception, stackTrace})`
- `verbose(message, {scene, exception, stackTrace})`
- `critical(message, {scene, exception, stackTrace})`

业务代码不直接构造 `PuupeeTalkerLog`，避免重复填 key、title 和 logLevel。

## 模块与场景

第一批模块 key：

- `sync-server`
- `sync-client`
- `sync-storage`
- `sync-auth`
- `sync-mcp`

第一阶段只注册并使用 `sync-server`。其它 key 可以同时注册，也可以等迁移对应模块时再注册。为了让设置页不过早出现空分类，推荐第一阶段只注册 `sync-server`。

`sync_server.dart` 第一批 scene：

- `lifecycle`：启动、停止、重复启动、端口监听。
- `db`：SQLite、Postgres、CRDT 字段初始化。
- `mq`：AMQP exchange、content event publisher。
- `http`：非 2xx 响应、请求处理异常。
- `storage`：本地上传、下载、文件凭证、容量。
- `pairing`：配对请求、配对确认、公钥和 token 验证。
- `auth`：权限检查、User-Agent 版本校验、认证分支。
- `mcp`：MCP 路由、SSE 连接和命令处理。
- `cache`：changeset 缓存清理、App 信息缓存。
- `crdt`：changeset、merge、record 校验。
- `content-event`：Puupee content event 发布。
- `app-info`：App 标题解析和 API 兜底。

## 数据流

1. `sync_server.dart` 调用 `_logger.warning('处理配对请求失败', scene: PuupeeLogScene.pairing, exception: e, stackTrace: st)`。
2. `PuupeeLogger` 创建 `PuupeeTalkerLog`，写入 `key = sync-server`、`logLevel = warning`、`message = [pairing] 处理配对请求失败`。
3. `PuupeeLogger` 调用全局 `talker.logCustom(log)`。
4. Talker 按现有 settings/filter 处理日志，写入 stream、history 和 console。
5. `TalkerScreen(talker: talker)` 从全局 Talker history 读取日志。
6. 用户在 Talker UI 中勾选或搜索 `sync-server`，即可单独查看同步服务器日志。

## 错误处理

- Logger 自身不应抛出业务异常；日志写入失败不应影响同步流程。
- `exception` 和 `stackTrace` 必须原样传给 Talker，不拼进 message，避免丢失结构化信息。
- 请求/响应体日志继续遵循现有大小和内容类型限制，避免为了场景化日志扩大敏感信息暴露面。
- 对 token、公钥私钥、API key、认证 header 等敏感信息，只记录脱敏摘要或上下文，不记录原文。

## 迁移步骤

1. 在 `puupee_utilities` 新增 `PuupeeTalkerLog`、`PuupeeLogger`、模块/场景常量和注册函数。
2. 修正 `createTalker({settings})`，并在全局 `talker` 初始化后注册 `sync-server`。
3. 在 `sync_server.dart` 将 `_logger` 替换为 `PuupeeLogger.syncServer()`。
4. 按场景迁移现有 `_logger.info/warning/error` 调用。
5. 保留原日志文本语义，不做无关重写。
6. 运行 `dart format`、`dart analyze` 和相关 `puupee_sync` 测试。

## 测试策略

基础设施测试放在 `puupee_utilities`：

- `PuupeeTalkerLog` 应写入预期 `key`、`title`、`logLevel`。
- 有 scene 时 message 应以 `[scene] ` 开头。
- 无 scene 时 message 不应带空括号或多余空格。
- `exception` 与 `stackTrace` 应能完整传递。
- `createTalker(settings: ...)` 应保留传入 settings。

同步包测试放在 `puupee_sync`：

- `sync_server.dart` 使用的 logger 生成 `key == sync-server`。
- 关键路径日志按预期 scene 分类，例如启动为 `lifecycle`、数据库初始化为 `db`、配对为 `pairing`。
- 现有 `sync_server_test.dart` 不应因为日志迁移改变行为。

## 风险与取舍

- Talker UI 的内置筛选主要基于 key，scene 只能通过搜索过滤。这是有意取舍：避免设置页出现过多细碎 key。
- 第一阶段只迁移 `sync_server.dart`，其它 sync 日志仍会落在普通等级 key 下。后续按模块逐步迁移。
- `PuupeeLogger` 是对 Talker 的轻封装，不应发展成独立日志框架；它只负责补充模块 key、scene、等级和异常传递。
- 如果未来某个 scene 日志量过大，可以将它升级为独立模块 key，例如从 `sync-server` 的 `[storage]` 升级为 `sync-storage`。

## 验收标准

- 打开现有开发者日志页后，可以看到并筛选 `sync-server` 日志。
- `sync_server.dart` 迁移后的日志仍保留原有等级、异常和堆栈。
- 不需要改造 `TalkerScreen`。
- 不引入新的日志依赖。
- 第一阶段代码变更仅限日志基础设施与 `sync_server.dart` 的调用迁移。
