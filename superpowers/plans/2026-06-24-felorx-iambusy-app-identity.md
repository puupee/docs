# Felorx Iambusy 应用身份迁移计划

## 目标

- 将 Flutter 包名 `puupee_iambusy` 迁移为 `felorx_iambusy`。
- 将英文应用显示名 `Puupee Iambusy` 迁移为 `Felorx Iambusy`。
- 将 URL scheme 从 `puupeeiambusy` 迁移为 `felorxiambusy`。
- 将 Android、iOS、macOS、Linux、Windows、Web 配置中的应用标识改为 Felorx 前缀。

## 非目标

- 不移动 `apps/iambusy` 目录；`iambusy` 已是单数应用 slug。
- 不重命名 `puupee_shared`、`puupee_ui` 等共享依赖包。
- 不调整未迁移的 CLI 命令名或共享模型名。

## 步骤

1. 更新 `apps/iambusy` 内包名、imports、desktop/mobile/web runner、URL scheme 和英文显示名。
2. 重新生成路由/Provider 相关输出。
3. 对应用分析、官网类型检查和旧身份残留扫描进行验证。
4. 分别提交 docs 计划与顶层 Iambusy 应用身份迁移。
