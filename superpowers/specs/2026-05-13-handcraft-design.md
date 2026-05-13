# 小汪手工设计规格

日期：2026-05-13

## 背景

小汪手工是一个 Puupee Flutter 子应用。用户上传一张包或衣服的款式照片，再上传一张布料或面料照片，应用生成使用该布料制作该物品的成品效果图、尺寸裁剪图，并生成展示“布料、裁剪、缝纫、成品”过程的视频。

第一版目标是做成可运行原型。生成能力使用本地模拟服务和占位结果，不接入真实 AI、对象存储或视频生成 API，但代码边界、数据字段和页面流程要按后续真实生成与媒体同步来设计。

## 范围

第一版包含：

- 新建子应用 `apps/puupee/handcraft`。
- package 名称为 `puupee_handcraft`。
- 应用中文名为 `小汪手工`。
- 路由前缀为 `/handcraft`。
- Android applicationId 和 namespace 为 `com.puupee.handcraft`。
- 创建作品历史、创作工作台、素材库和设置入口。
- 支持上传款式图和布料图的本地路径记录。
- 支持选择物品类型，填写目标尺寸；未填写时生成模拟尺寸。
- 支持完整模拟生成流程：成品效果图、裁剪图、过程视频占位。
- 数据模型预留远程媒体 URL、上传状态、生成状态和错误信息。
- 提供 EnvConfig、模型逻辑、生成服务和基础 Provider/Repo 测试。

第一版不包含：

- 真实图片生成、视频生成或 AI API 调用。
- 真实对象存储上传。
- 精确服装打版算法。
- 复杂媒体文件管理。
- 完整跨设备媒体同步。

## 用户流程

用户进入应用后可以从“创作”开始新作品，也可以从“作品”继续历史项目。

移动端使用分步向导：

1. 上传款式图。
2. 上传布料图。
3. 选择物品类型并填写尺寸。
4. 启动生成并查看结果。

桌面端使用工作台布局：

- 左侧：款式图、布料图、物品类型、尺寸字段、生成按钮。
- 右侧：阶段状态、成品效果图、裁剪图、过程视频占位。

生成阶段为：

1. `ready`
2. `renderingPreview`
3. `draftingPattern`
4. `makingVideo`
5. `completed`

失败时进入失败状态，展示错误信息并允许重试。

## 导航

应用包含四个主要入口：

- 创作：新建或编辑当前作品。移动端是向导，桌面端是工作台。
- 作品：同步型作品历史列表，可查看草稿、生成中、已完成、失败的作品。
- 素材：轻量媒体库视图，展示作品引用的款式图、布料图、生成结果图、裁剪图和视频引用。
- 我的：复用 Puupee 现有设置页体系，包括账号、存储、同步、反馈、关于等页面。

## 架构

新应用沿用现有 Puupee 子应用结构：

- `main.dart` 通过 `runMyApp` 启动。
- `env.dart` 提供 `HandcraftEnvConfig`，包含必需的左右布局默认值。
- `router.dart` 使用 typed `go_router`，生成 `router.g.dart`。
- Shell 使用 `AdaptiveLayout` 实现桌面侧边栏，移动端使用底部导航。
- UI 使用 `shadcn_flutter`、`puupee_ui` 和 `puupee_shared` 组件，不直接使用 `flutter/material.dart` 组件模式。
- 状态使用 Riverpod annotation provider。
- 页面不直接访问底层存储，统一通过 repo 和 provider。

主要模块：

- `models/handcraft_project.dart`：作品实体、枚举、尺寸和媒体引用。
- `repo/handcraft_project_repo.dart`：作品历史读写接口和第一版本地实现。
- `services/handcraft_generation_service.dart`：本地模拟生成服务。
- `providers/handcraft_providers.dart`：作品列表、当前作品和生成流程状态。
- `pages/handcraft_studio_page.dart`：创作工作台与移动端向导入口。
- `pages/handcraft_projects_page.dart`：作品历史。
- `pages/handcraft_assets_page.dart`：素材视图。
- `components/`：上传卡、尺寸编辑器、生成阶段条、结果预览卡、作品卡片。

## 数据模型

核心模型为 `HandcraftProject`。

字段分组：

- 基础信息：`id`、`title`、`itemType`、`createdAt`、`updatedAt`。
- 状态：`status`、`currentStage`。
- 输入媒体：`styleImageLocalPath`、`styleImageRemoteUrl`、`fabricImageLocalPath`、`fabricImageRemoteUrl`。
- 生成结果：`previewImageLocalPath`、`previewImageRemoteUrl`、`patternImageLocalPath`、`patternImageRemoteUrl`、`processVideoLocalPath`、`processVideoRemoteUrl`。
- 媒体同步：`styleUploadStatus`、`fabricUploadStatus`、`resultUploadStatus`。
- 尺寸：`dimensionMode`、`dimensions`。
- 生成说明：`generationPrompt`、`generationSeed`、`errorMessage`。

枚举包括：

- `HandcraftItemType`：`bag`、`clothing`、`other`。
- `HandcraftProjectStatus`：`draft`、`generating`、`completed`、`failed`。
- `HandcraftGenerationStage`：`ready`、`renderingPreview`、`draftingPattern`、`makingVideo`、`completed`。
- `HandcraftUploadStatus`：`localOnly`、`queued`、`uploading`、`uploaded`、`failed`。
- `HandcraftDimensionMode`：`manual`、`estimated`。

第一版 repo 使用本地内存或轻量本地数据源保证应用可运行。接口设计按未来替换到 Puupee/Sync 的形状保留：新增作品、更新作品、按状态查询、启动生成、重试生成。

## 尺寸和裁剪图

尺寸规则采用“用户可填，不填就模拟估算”。

包类默认尺寸示例：

- 宽度、主体高度、底宽、提手长度、缝份。

衣服类默认尺寸示例：

- 衣长、胸围、肩宽、袖长、缝份。

其他类型默认尺寸示例：

- 宽度、高度、厚度、缝份。

裁剪图第一版是可视化裁片说明图，不是精确工业制版。裁剪图内容包括主体片、侧片、提手或袖片、缝份、推荐布料用量和简要制作备注。

## 模拟生成服务

`HandcraftGenerationService` 负责本地模拟生成。

输入：

- 款式图本地路径。
- 布料图本地路径。
- 物品类型。
- 尺寸参数。

输出：

- 更新后的作品状态。
- 成品效果图占位引用。
- 裁剪图占位引用。
- 过程视频占位引用。
- 生成说明和 seed。

服务按阶段推进：

1. 校验款式图和布料图。
2. 标记状态为 `generating`，阶段为 `renderingPreview`。
3. 生成成品效果图占位。
4. 推进到 `draftingPattern`，生成裁剪图占位。
5. 推进到 `makingVideo`，生成过程视频占位。
6. 标记状态为 `completed`。

模拟失败时状态为 `failed`，写入 `errorMessage`，保留已有输入，用户可重试。

## 错误处理

- 未上传款式图或布料图时不允许开始生成。
- 生成中禁用重复提交。
- 图片路径为空或不可读时显示提示，不清空用户已填信息。
- 模拟服务失败后保留草稿和错误信息。
- 重试生成时清除上一轮错误信息，并重新推进阶段。

## Android 和环境配置

Android 平台从现有轻量 Puupee 子应用复制并调整：

- `namespace = "com.puupee.handcraft"`。
- `applicationId = "com.puupee.handcraft"`。
- Kotlin 和 Java 目标版本为 17。
- 保留 core library desugaring。
- 保留 XG manifest placeholders。
- 使用 `@string/app_name`，提供默认和中文 strings。
- 保留 FileProvider 和 `file_paths.xml`。

`HandcraftEnvConfig` 必须包含：

- `env`
- `appId = "handcraft"`
- `appTitle = "小汪手工"`
- `apiUrl`
- `authUrl`
- `authClientId`
- `feedbackUrl`
- `defaultLeftFlex = 0.5`
- `defaultRightFlex = 0.5`
- `defaultLeftMinWidth = 250`
- `defaultRightMinWidth = 400`

测试需断言 `defaultLeftSize` 和 `defaultRightSize` 为 null。

## 测试和验收

需要的验证：

- `test/env_test.dart`：EnvConfig 必需字段和布局默认值。
- 模型测试：默认尺寸、状态流转、媒体字段复制和错误字段。
- 生成服务测试：完整生成、缺图报错、失败后可重试。
- Provider/Repo 测试：新增作品、更新尺寸、更新媒体、触发生成。
- 路由代码生成后运行单应用 analyze。

建议命令：

```bash
cd apps/puupee/handcraft
dart run build_runner build --delete-conflicting-outputs
dart analyze
flutter test
flutter build apk --debug --target-platform android-arm64 --no-pub
```

验收标准：

- Android 项目存在且包含 `AndroidManifest.xml`。
- 应用可启动到 `/handcraft`。
- 可以创建作品并填写两张图片路径。
- 不填尺寸也可以生成模拟尺寸。
- 生成阶段能完整推进到完成。
- 作品历史能看到已完成作品。
- 素材页能看到输入和生成结果引用。
- EnvConfig 测试通过。

## 后续扩展

后续真实版本可在不重写 UI 的前提下替换这些边界：

- 将 repo 替换为 Puupee/Sync 数据源。
- 将本地媒体引用替换为对象存储上传结果。
- 将模拟生成服务替换为真实图片生成、裁剪图生成和视频生成服务。
- 将裁剪图从说明图升级为更专业的服装打版或包款制版算法。
