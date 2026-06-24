# Felorx Handcraft 应用身份迁移计划

## 目标

- 将 Flutter 包名 `puupee_handcraft` 迁移为 `felorx_handcraft`。
- 将英文应用显示名 `Puupee Handcraft` 迁移为 `Felorx Handcraft`。
- 将应用 bundle id 从 `com.puupee.handcraft` 迁移为 `com.felorx.handcraft`，包含 dev/debug/test 变体。
- 更新构建、文档、桌面 runner、installer、测试 imports、manifest 和脚本中的 Handcraft 应用身份。

## 非目标

- 不移动 `apps/handcraft` 目录；`handcraft` 已是单数应用 slug。
- 不在本步骤修改 Handcraft 业务模型、数据库模型、同步模型或服务端内容类型。
- 不修改通用 `puupee_shared`、`puupee_ui`、`puupee_utilities` 依赖包名。

## 步骤

1. 更新 `apps/handcraft` 内包名、imports、manifest、desktop runner、installer、bundle id、URL scheme 和英文显示名。
2. 更新构建脚本、文档和相关测试引用。
3. 重新生成路由/Provider 相关输出，对应用测试、官网类型检查和旧身份残留扫描进行验证。
4. 分别提交 docs 计划与顶层 Handcraft 应用身份迁移。
