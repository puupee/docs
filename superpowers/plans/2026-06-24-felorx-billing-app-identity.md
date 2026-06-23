# Felorx Billing 应用身份迁移计划

## 目标

- 将 `apps/billings` 子应用目录迁移为 `apps/billing`。
- 将 Flutter 包名 `puupee_billings` 迁移为 `felorx_billing`。
- 将应用显示名 `Puupee Billings` 迁移为 `Felorx Billing`。
- 将应用 bundle id 从 `com.puupee.billings` 迁移为 `com.felorx.billing`，包含 dev/debug/test 变体。
- 将构建/官网/脚本中的应用 slug 路径从 `billings` 迁移为 `billing`。

## 非目标

- 不在本步骤修改账单业务代码里表示“账单列表”的变量名，例如 `billings`、`allBillings`。
- 不在本步骤修改 `Products.billings`、数据库模型、同步模型或服务端内容类型；这些归入后续 Felorx 模型迁移。
- 不重命名图片资产文件名，例如 `ic_launcher_billings.png`，除非验证表明应用身份迁移必须同步调整。

## 步骤

1. 移动 `apps/billings` 到 `apps/billing`，更新 workspace、builder、脚本和网站引用。
2. 更新 Flutter/Dart 包名、imports、manifest、desktop runner、installer、bundle id 和显示名。
3. 对当前应用和关键消费者运行 `dart pub get`、分析、目标测试与旧身份残留扫描。
4. 分别提交 docs 计划与顶层 Billing 应用身份迁移。
