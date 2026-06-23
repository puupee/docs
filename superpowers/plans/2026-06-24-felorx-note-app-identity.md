# Felorx Note 应用身份迁移计划

## 目标

- 将 `apps/notes` 子应用目录迁移为 `apps/note`。
- 将 Flutter 包名 `puupee_notes` 迁移为 `felorx_note`。
- 将英文应用显示名 `Puupee Notes` 迁移为 `Felorx Note`。
- 将应用 bundle id 从 `com.puupee.notes` 迁移为 `com.felorx.note`，包含 dev/debug/test 变体。
- 将构建、文档和路由里的应用 slug 从 `notes` 迁移为 `note`。

## 非目标

- 不在本步骤修改笔记、内容块、同步等业务模型。
- 不在本步骤修改 `Products.notes`、数据库模型、同步模型或服务端内容类型；这些归入后续 Felorx 模型迁移。
- 不强行修改中文商品名和中文文案里的自然语言“笔记”。

## 步骤

1. 移动 `apps/notes` 到 `apps/note`，更新 workspace、builder、脚本和文档引用。
2. 更新 Flutter/Dart 包名、imports、manifest、desktop runner、installer、bundle id 和英文显示名。
3. 重新生成路由/本地化相关输出，对应用运行分析与旧身份残留扫描。
4. 分别提交 docs 计划与顶层 Note 应用身份迁移。
