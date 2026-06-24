# Felorx Cover 应用身份迁移计划

## 目标

- 将 Flutter 包名 `puupee_cover` 迁移为 `felorx_cover`。
- 将英文应用显示名 `Puupee Cover` 迁移为 `Felorx Cover`。
- 将应用 bundle id 从 `com.puupee.cover` 迁移为 `com.felorx.cover`，包含 dev/debug/test 变体。
- 更新构建、文档、桌面 runner、测试 imports、manifest 和官网应用数据中的 Cover 应用身份。

## 非目标

- 不移动 `apps/cover` 目录；`cover` 已是单数应用 slug。
- 不在本步骤修改 Cover 业务模型、数据库模型、同步模型或服务端内容类型。
- 不修改通用 `puupee_shared`、`puupee_ui`、`puupee_utilities` 依赖包名。

## 步骤

1. 更新 `apps/cover` 内包名、imports、manifest、desktop runner、bundle id、URL scheme 和英文显示名。
2. 更新官网应用数据、构建脚本、文档和相关测试引用。
3. 重新生成路由/Provider 相关输出，对应用测试、官网类型检查和旧身份残留扫描进行验证。
4. 分别提交 docs 计划与顶层 Cover 应用身份迁移。
