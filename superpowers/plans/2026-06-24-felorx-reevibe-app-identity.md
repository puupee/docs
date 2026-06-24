# Felorx ReeVibe 应用身份迁移计划

## 目标

- 将 Flutter 包名 `puupee_reevibe` 迁移为 `felorx_reevibe`。
- 将应用 bundle id 从 `com.puupee.reevibe` 迁移为 `com.felorx.reevibe`，包含 dev/debug/test 变体。
- 将 URL scheme 从 `puupeereevibe` 迁移为 `felorxreevibe`。
- 更新 Android、iOS、macOS、Linux、Windows 配置、路由 imports 和默认 client id。

## 非目标

- 不移动 `apps/reevibe` 目录；`reevibe` 已是单数应用 slug。
- 不重命名 `puupee_shared`、`puupee_ui` 等共享依赖包。
- 不修改 ReeVibe 业务功能或页面结构。

## 步骤

1. 更新 `apps/reevibe` 内包名、imports、manifest、desktop runner、bundle id、URL scheme 和默认 client id。
2. 重新生成路由/Provider 相关输出。
3. 对应用分析、官网类型检查和旧身份残留扫描进行验证。
4. 分别提交 docs 计划与顶层 ReeVibe 应用身份迁移。
