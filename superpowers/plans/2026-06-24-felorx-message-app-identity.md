# Felorx Message 应用身份迁移计划

## 目标

- 将 `apps/messages` 子应用目录迁移为 `apps/message`。
- 将 Flutter 包名 `puupee_messages` 迁移为 `felorx_message`。
- 将英文应用显示名 `Puupee Messages` 迁移为 `Felorx Message`。
- 将应用 bundle id 从 `com.puupee.messages` 迁移为 `com.felorx.message`，包含 dev/debug/test 变体。
- 将构建、文档、官网数据、通知跳转和路由里的应用 slug 从 `messages` 迁移为 `message`。

## 非目标

- 不在本步骤修改消息、发布者、模板等业务模型。
- 不在本步骤修改 `Products.messages`、数据库模型、同步模型或服务端内容类型；这些归入后续 Felorx 模型迁移。
- 不强行修改中文自然语言“消息”或普通变量名 `messages`。

## 步骤

1. 移动 `apps/messages` 到 `apps/message`，更新 workspace、builder、脚本、文档和官网数据引用。
2. 更新 Flutter/Dart 包名、imports、manifest、desktop runner、installer、bundle id 和英文显示名。
3. 更新共享通知入口与订阅测试中的应用路由和 appId。
4. 重新生成路由/本地化相关输出，对应用、共享包目标测试、官网类型检查和旧身份残留扫描进行验证。
5. 分别提交 docs 计划、website 数据与顶层 Message 应用身份迁移。
