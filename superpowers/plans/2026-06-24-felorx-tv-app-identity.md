# Felorx TV 应用身份迁移计划

## 目标

- 将 Flutter 包名 `puupee_tv` 迁移为 `felorx_tv`。
- 将英文应用显示名 `Puupee TV` / `Puupee Tv` 迁移为 `Felorx TV`。
- 将应用 bundle id 从 `com.puupee.tv` 迁移为 `com.felorx.tv`，包含 dev 变体。
- 更新 Android 配置、文档、manifest、imports 和 URL scheme 中的 TV 应用身份。

## 非目标

- 不移动 `apps/tv` 目录；`tv` 已是单数应用 slug。
- 不在本步骤修改通用共享包、同步模型、AI/媒体业务模型或 `puupee_shared` 等依赖名。
- 不调整历史文档中对 Puupee CRDT/功能模型的技术命名，后续统一处理功能模型迁移。

## 步骤

1. 更新 `apps/tv` 内包名、Android namespace/applicationId、显示名和英文品牌文案。
2. 移动 Android Kotlin 包目录到 `com/felorx/tv`。
3. 对应用分析、官网类型检查和旧身份残留扫描进行验证。
4. 分别提交 docs 计划与顶层 TV 应用身份迁移。
