# Felorx Box 应用身份迁移计划

## 目标

- 将 `apps/boxes` 子应用目录迁移为 `apps/box`。
- 将 Flutter 包名 `puupee_boxes` 迁移为 `felorx_box`。
- 将英文应用显示名 `Puupee Boxes` 迁移为 `Felorx Box`。
- 将应用 bundle id 从 `com.puupee.boxes` 迁移为 `com.felorx.box`，包含 dev/debug/test 变体。
- 将构建、文档和路由里的应用 slug 从 `boxes` 迁移为 `box`。

## 非目标

- 不在本步骤修改物品、资产、盘点等业务模型。
- 不在本步骤修改 `Products.boxes`、数据库模型、同步模型或服务端内容类型；这些归入后续 Felorx 模型迁移。
- 不强行修改中文商品名“小汪记物”。

## 步骤

1. 移动 `apps/boxes` 到 `apps/box`，更新 workspace、builder、脚本和文档引用。
2. 更新 Flutter/Dart 包名、imports、manifest、desktop runner、installer、bundle id 和英文显示名。
3. 重新生成路由/Provider 相关输出，对应用和关键脚本运行分析、目标测试与旧身份残留扫描。
4. 分别提交 docs 计划与顶层 Box 应用身份迁移。
