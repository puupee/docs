# Felorx OPC 应用身份迁移计划

## 目标

- 将 Flutter 包名 `puupee_opc` 迁移为 `felorx_opc`。
- 将英文应用显示名 `Puupee OPC` / `Puupee Opc` 迁移为 `Felorx OPC` / `Felorx Opc`。
- 将 URL scheme 从 `puupeeopc` 迁移为 `felorxopc`。
- 将 Android、iOS、macOS、Linux、Windows 配置中的应用标识改为 Felorx 前缀。

## 非目标

- 不移动 `apps/opc` 目录；`opc` 已是单数应用 slug。
- 不重命名 `puupee_shared`、`puupee_ui` 等共享依赖包。
- 不调整未迁移的 CLI 命令名或共享模型名。

## 步骤

1. 更新 `apps/opc` 内包名、imports、desktop/mobile runner、URL scheme 和英文显示名。
2. 将 Android Kotlin package 路径从 `com/puupee/opc` 移到 `com/felorx/opc`。
3. 重新生成路由/Provider 相关输出。
4. 对应用分析、测试、官网类型检查和旧身份残留扫描进行验证。
5. 分别提交 docs 计划与顶层 OPC 应用身份迁移。
