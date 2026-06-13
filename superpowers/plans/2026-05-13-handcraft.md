# 小汪手工实现计划

> **给智能体工作者：** 必须使用 `superpowers:subagent-driven-development`（推荐）或 `superpowers:executing-plans` 按任务逐项执行。本计划使用复选框语法记录进度。

**目标：** 创建新的 `handcraft` Flutter 子应用，让用户创建手工作品，上传款式图和布料图，填写或模拟估算尺寸，并跑通成品效果图、裁剪图、过程视频引用的本地模拟生成流程。

**架构：** `apps/handcraft` 作为标准 Puupee 子应用接入，启动走 `runMyApp`，路由走类型化 `go_router`，状态走 Riverpod 注解，UI 使用 `shadcn_flutter`、`puupee_ui` 和 `puupee_shared`。作品读写全部放在 `HandcraftProjectRepo` 后面，生成逻辑全部放在 `HandcraftGenerationService` 后面，第一版使用内存数据源和本地模拟结果，字段和接口按后续 Puupee/Sync、对象存储、真实 AI 生成预留。

**技术栈：** Dart 3.8、Flutter、shadcn_flutter、go_router / go_router_builder、Riverpod 注解、hooks_riverpod、puupee_shared、puupee_ui、build_runner、Android Kotlin/Java 17。

---

## 文件结构

- 修改 `pubspec.yaml`：把 `apps/handcraft` 加入工作区。
- 创建 `apps/handcraft/pubspec.yaml`：定义 `puupee_handcraft` Flutter 包。
- 创建 `apps/handcraft/.metadata`：让 Flutter 工具识别为应用项目。
- 创建 `apps/handcraft/lib/env.dart`：实现 `HandcraftEnvConfig`。
- 创建 `apps/handcraft/lib/main.dart`：通过 `runMyApp` 启动。
- 创建 `apps/handcraft/lib/models/handcraft_project.dart`：作品模型、枚举、尺寸逻辑、状态标签。
- 创建 `apps/handcraft/lib/services/handcraft_generation_service.dart`：本地模拟生成服务。
- 创建 `apps/handcraft/lib/repo/handcraft_project_repo.dart`：作品仓库接口和内存实现。
- 创建 `apps/handcraft/lib/providers/handcraft_providers.dart`：仓库、项目流、当前工作台、生成控制器提供器。
- 生成 `apps/handcraft/lib/providers/handcraft_providers.g.dart`。
- 创建 `apps/handcraft/lib/router.dart`：创作、作品、素材、我的四个分支路由。
- 生成 `apps/handcraft/lib/router.g.dart`。
- 创建 `apps/handcraft/lib/components/` 下的工作台组件：自适应 Shell、作品卡、媒体输入卡、尺寸编辑器、生成结果面板。
- 创建 `apps/handcraft/lib/pages/` 下的页面：创作页、作品页、素材页。
- 创建 `apps/handcraft/android/`：从现有 Puupee 应用复制 Android 模板并改为 `com.puupee.handcraft`。
- 创建 `apps/handcraft/README.md`：说明第一版范围和验证命令。
- 创建 `apps/handcraft/test/` 下的测试：EnvConfig、模型、生成服务、仓库、提供器、页面冒烟测试。

---

### 任务 1：创建应用骨架和 EnvConfig

**文件：**
- 修改：`pubspec.yaml`
- 创建：`apps/handcraft/pubspec.yaml`
- 创建：`apps/handcraft/.metadata`
- 创建：`apps/handcraft/lib/env.dart`
- 创建：`apps/handcraft/lib/main.dart`
- 测试：`apps/handcraft/test/env_test.dart`

- [ ] **步骤 1：先写失败的 EnvConfig 测试**

创建 `apps/handcraft/test/env_test.dart`：

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:puupee_handcraft/env.dart';

void main() {
  test('HandcraftEnvConfig 提供必需的左右布局默认值', () {
    final env = HandcraftEnvConfig();

    expect(env.appId, 'handcraft');
    expect(env.appTitle, '小汪手工');
    expect(env.defaultLeftFlex, 0.5);
    expect(env.defaultRightFlex, 0.5);
    expect(env.defaultLeftMinWidth, 250);
    expect(env.defaultRightMinWidth, 400);
    expect(env.defaultLeftSize, isNull);
    expect(env.defaultRightSize, isNull);
  });
}
```

- [ ] **步骤 2：运行测试确认当前失败**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/env_test.dart --no-pub
```

预期：目录或 `pubspec.yaml` 还不存在，命令失败。

- [ ] **步骤 3：加入工作区**

在根目录 `pubspec.yaml` 的工作区列表中，紧跟 `apps/builder` 后加入：

```yaml
  - apps/handcraft
```

- [ ] **步骤 4：创建应用 pubspec**

创建 `apps/handcraft/pubspec.yaml`：

```yaml
name: puupee_handcraft
resolution: workspace
version: 0.1.0
publish_to: none
description: 小汪手工 - 布料成品效果、裁剪图和制作过程模拟生成工具

environment:
  sdk: ^3.8.1

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  puupee_shared: ^0.0.41
  puupee_ui: ^0.0.3+3
  puupee_utilities: ^0.0.13+3
  puupee: ^1.1.3
  puupee_sync: ^0.0.30+2

  flutter_riverpod: ^2.5.1
  hooks_riverpod: ^2.5.1
  riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

  go_router: ^14.2.0
  shadcn_flutter: ^0.0.52

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0
  build_runner: ^2.4.15
  riverpod_generator: ^2.6.5
  go_router_builder: 2.8.2

flutter:
  uses-material-design: true
  generate: true
```

- [ ] **步骤 5：创建 Flutter 项目元数据**

创建 `apps/handcraft/.metadata`：

```yaml
# This file tracks properties of this Flutter project.
# Used by Flutter tool to assess capabilities and perform upgrades etc.
#
# This file should be version controlled and should not be manually edited.

version:
  revision: "adc901062556672b4138e18a4dc62a4be8f4b3c2"
  channel: "stable"

project_type: app

migration:
  platforms:
    - platform: root
      create_revision: adc901062556672b4138e18a4dc62a4be8f4b3c2
      base_revision: adc901062556672b4138e18a4dc62a4be8f4b3c2
    - platform: android
      create_revision: adc901062556672b4138e18a4dc62a4be8f4b3c2
      base_revision: adc901062556672b4138e18a4dc62a4be8f4b3c2

  unmanaged_files:
    - 'lib/main.dart'
```

- [ ] **步骤 6：实现 EnvConfig**

创建 `apps/handcraft/lib/env.dart`：

```dart
import 'package:puupee_shared/env.dart';

/// 小汪手工应用环境配置。
class HandcraftEnvConfig extends EnvConfig {
  HandcraftEnvConfig()
    : super(
        env: 'production',
        appId: 'handcraft',
        appTitle: '小汪手工',
        apiUrl: const String.fromEnvironment(
          'API_URL',
          defaultValue: 'https://api.puupee.com',
        ),
        authUrl: const String.fromEnvironment(
          'AUTH_URL',
          defaultValue: 'https://auth.puupee.com',
        ),
        authClientId: const String.fromEnvironment(
          'AUTH_CLIENT_ID',
          defaultValue: 'Puupee_Sync_Node',
        ),
        feedbackUrl: const String.fromEnvironment(
          'FEEDBACK_URL',
          defaultValue: 'https://feedback.puupee.com',
        ),
        defaultLeftFlex: 0.5,
        defaultRightFlex: 0.5,
        defaultLeftMinWidth: 250,
        defaultRightMinWidth: 400,
      );
}
```

- [ ] **步骤 7：实现应用入口**

创建 `apps/handcraft/lib/main.dart`：

```dart
import 'package:go_router/go_router.dart';
import 'package:puupee_handcraft/env.dart';
import 'package:puupee_handcraft/router.dart';
import 'package:puupee_shared/app/startup.dart';

/// 小汪手工应用入口。
void main() async {
  await runMyApp(
    env: HandcraftEnvConfig(),
    createRouter: (ref) =>
        GoRouter(initialLocation: '/handcraft', routes: $appRoutes),
  );
}
```

说明：`router.dart` 会在任务 5 创建，因此任务 1 结束时只运行 EnvConfig 测试。

- [ ] **步骤 8：运行 EnvConfig 测试**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/env_test.dart --no-pub
```

预期：测试通过。

- [ ] **步骤 9：提交任务 1**

运行：

```bash
git add pubspec.yaml apps/handcraft/pubspec.yaml apps/handcraft/.metadata apps/handcraft/lib/env.dart apps/handcraft/lib/main.dart apps/handcraft/test/env_test.dart
git commit -m "feat(handcraft): 创建小汪手工应用骨架"
```

---

### 任务 2：实现作品模型和尺寸逻辑

**文件：**
- 创建：`apps/handcraft/lib/models/handcraft_project.dart`
- 测试：`apps/handcraft/test/handcraft_project_test.dart`

- [ ] **步骤 1：先写失败的模型测试**

创建 `apps/handcraft/test/handcraft_project_test.dart`：

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:puupee_handcraft/models/handcraft_project.dart';

void main() {
  test('新建项目默认是草稿并使用本地媒体状态', () {
    final now = DateTime(2026, 5, 13, 9);
    final project = HandcraftProject.create(
      id: 'project-1',
      title: '帆布托特包',
      itemType: HandcraftItemType.bag,
      now: now,
    );

    expect(project.status, HandcraftProjectStatus.draft);
    expect(project.currentStage, HandcraftGenerationStage.ready);
    expect(project.styleUploadStatus, HandcraftUploadStatus.localOnly);
    expect(project.fabricUploadStatus, HandcraftUploadStatus.localOnly);
    expect(project.resultUploadStatus, HandcraftUploadStatus.localOnly);
    expect(project.createdAt, now);
    expect(project.updatedAt, now);
    expect(project.hasRequiredInputs, isFalse);
  });

  test('更新两张输入图后可以开始生成', () {
    final project = HandcraftProject.create(
      id: 'project-1',
      title: '帆布托特包',
      itemType: HandcraftItemType.bag,
      now: DateTime(2026, 5, 13, 9),
    ).copyWith(
      styleImageLocalPath: '/tmp/bag.jpg',
      fabricImageLocalPath: '/tmp/canvas.jpg',
    );

    expect(project.hasRequiredInputs, isTrue);
    expect(project.styleImageLocalPath, '/tmp/bag.jpg');
    expect(project.fabricImageLocalPath, '/tmp/canvas.jpg');
  });

  test('不同类型作品有不同模拟尺寸', () {
    final bag = HandcraftDimensions.estimatedFor(HandcraftItemType.bag);
    final clothing = HandcraftDimensions.estimatedFor(
      HandcraftItemType.clothing,
    );
    final other = HandcraftDimensions.estimatedFor(HandcraftItemType.other);

    expect(bag.mode, HandcraftDimensionMode.estimated);
    expect(bag.values['widthCm'], 32);
    expect(bag.values['seamAllowanceCm'], 1);
    expect(clothing.values['chestCm'], 96);
    expect(clothing.values['sleeveCm'], 58);
    expect(other.values['heightCm'], 24);
  });

  test('用户填写尺寸时优先使用手动尺寸', () {
    final project = HandcraftProject.create(
      id: 'project-1',
      title: '衬衫',
      itemType: HandcraftItemType.clothing,
      now: DateTime(2026, 5, 13, 9),
    ).copyWith(
      dimensions: const HandcraftDimensions(
        mode: HandcraftDimensionMode.manual,
        values: {'lengthCm': 70, 'chestCm': 102},
      ),
    );

    expect(project.effectiveDimensions.mode, HandcraftDimensionMode.manual);
    expect(project.effectiveDimensions.values['lengthCm'], 70);
    expect(project.effectiveDimensions.values['chestCm'], 102);
  });
}
```

- [ ] **步骤 2：运行测试确认失败**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_project_test.dart --no-pub
```

预期：缺少 `models/handcraft_project.dart`，测试失败。

- [ ] **步骤 3：实现模型**

创建 `apps/handcraft/lib/models/handcraft_project.dart`，必须包含：

- 枚举：`HandcraftItemType`、`HandcraftProjectStatus`、`HandcraftGenerationStage`、`HandcraftUploadStatus`、`HandcraftDimensionMode`。
- 类：`HandcraftDimensions`，包含 `estimatedFor` 和 `copyWith`。
- 类：`HandcraftProject`，包含设计规格中的基础信息、状态、输入媒体、生成结果、媒体同步、尺寸、生成说明字段。
- 工厂方法：`HandcraftProject.create`。
- 便捷 getter：`hasRequiredInputs`、`effectiveDimensions`。
- `copyWith` 必须允许把可空字段显式设为 null，可使用 `_sentinel` 实现。
- 中文标签扩展：`HandcraftItemTypeLabel`、`HandcraftProjectStatusLabel`、`HandcraftGenerationStageLabel`。

关键尺寸默认值：

```dart
HandcraftItemType.bag => {
  'widthCm': 32,
  'bodyHeightCm': 28,
  'bottomDepthCm': 10,
  'handleLengthCm': 46,
  'seamAllowanceCm': 1,
}

HandcraftItemType.clothing => {
  'lengthCm': 68,
  'chestCm': 96,
  'shoulderCm': 40,
  'sleeveCm': 58,
  'seamAllowanceCm': 1.2,
}

HandcraftItemType.other => {
  'widthCm': 24,
  'heightCm': 24,
  'depthCm': 6,
  'seamAllowanceCm': 1,
}
```

中文标签必须返回：

```dart
HandcraftItemType.bag => '包'
HandcraftItemType.clothing => '衣服'
HandcraftItemType.other => '其他'

HandcraftProjectStatus.draft => '草稿'
HandcraftProjectStatus.generating => '生成中'
HandcraftProjectStatus.completed => '已完成'
HandcraftProjectStatus.failed => '失败'

HandcraftGenerationStage.ready => '准备'
HandcraftGenerationStage.renderingPreview => '生成成品效果图'
HandcraftGenerationStage.draftingPattern => '生成裁剪图'
HandcraftGenerationStage.makingVideo => '生成过程视频'
HandcraftGenerationStage.completed => '完成'
```

- [ ] **步骤 4：运行模型测试**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_project_test.dart --no-pub
```

预期：测试通过。

- [ ] **步骤 5：提交任务 2**

运行：

```bash
git add apps/handcraft/lib/models/handcraft_project.dart apps/handcraft/test/handcraft_project_test.dart
git commit -m "feat(handcraft): 添加手工作品模型"
```

---

### 任务 3：实现本地模拟生成服务

**文件：**
- 创建：`apps/handcraft/lib/services/handcraft_generation_service.dart`
- 测试：`apps/handcraft/test/handcraft_generation_service_test.dart`

- [ ] **步骤 1：先写失败的生成服务测试**

创建 `apps/handcraft/test/handcraft_generation_service_test.dart`：

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:puupee_handcraft/models/handcraft_project.dart';
import 'package:puupee_handcraft/services/handcraft_generation_service.dart';

void main() {
  HandcraftProject readyProject() {
    return HandcraftProject.create(
      id: 'project-1',
      title: '帆布托特包',
      itemType: HandcraftItemType.bag,
      now: DateTime(2026, 5, 13, 9),
    ).copyWith(
      styleImageLocalPath: '/tmp/bag.jpg',
      fabricImageLocalPath: '/tmp/canvas.jpg',
    );
  }

  test('模拟生成会依次产出效果图、裁剪图和过程视频引用', () async {
    final stages = <HandcraftGenerationStage>[];
    final service = MockHandcraftGenerationService(delay: Duration.zero);

    final result = await service.generate(
      readyProject(),
      onStageChanged: stages.add,
    );

    expect(result.status, HandcraftProjectStatus.completed);
    expect(result.currentStage, HandcraftGenerationStage.completed);
    expect(result.previewImageLocalPath, 'mock://handcraft/project-1/preview.png');
    expect(result.patternImageLocalPath, 'mock://handcraft/project-1/pattern.png');
    expect(result.processVideoLocalPath, 'mock://handcraft/project-1/process.mp4');
    expect(result.resultUploadStatus, HandcraftUploadStatus.localOnly);
    expect(result.generationPrompt, contains('帆布托特包'));
    expect(result.generationSeed, isNotNull);
    expect(stages, [
      HandcraftGenerationStage.renderingPreview,
      HandcraftGenerationStage.draftingPattern,
      HandcraftGenerationStage.makingVideo,
      HandcraftGenerationStage.completed,
    ]);
  });

  test('缺少输入图时拒绝生成', () async {
    final service = MockHandcraftGenerationService(delay: Duration.zero);
    final project = HandcraftProject.create(
      id: 'project-1',
      title: '帆布托特包',
      itemType: HandcraftItemType.bag,
      now: DateTime(2026, 5, 13, 9),
    );

    expect(
      () => service.generate(project),
      throwsA(isA<HandcraftGenerationException>()),
    );
  });

  test('可以指定阶段模拟失败', () async {
    final service = MockHandcraftGenerationService(
      delay: Duration.zero,
      failAtStage: HandcraftGenerationStage.draftingPattern,
    );

    await expectLater(
      service.generate(readyProject()),
      throwsA(
        isA<HandcraftGenerationException>().having(
          (error) => error.stage,
          'stage',
          HandcraftGenerationStage.draftingPattern,
        ),
      ),
    );
  });
}
```

- [ ] **步骤 2：运行测试确认失败**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_generation_service_test.dart --no-pub
```

预期：缺少生成服务文件，测试失败。

- [ ] **步骤 3：实现生成服务**

创建 `apps/handcraft/lib/services/handcraft_generation_service.dart`，内容结构如下：

```dart
import 'dart:async';

import 'package:puupee_handcraft/models/handcraft_project.dart';

abstract class HandcraftGenerationService {
  Future<HandcraftProject> generate(
    HandcraftProject project, {
    FutureOr<void> Function(HandcraftGenerationStage stage)? onStageChanged,
  });
}

class HandcraftGenerationException implements Exception {
  const HandcraftGenerationException(this.message, {this.stage});

  final String message;
  final HandcraftGenerationStage? stage;

  @override
  String toString() {
    final stageText = stage == null ? '' : ' at ${stage!.name}';
    return 'HandcraftGenerationException$stageText: $message';
  }
}

class MockHandcraftGenerationService implements HandcraftGenerationService {
  const MockHandcraftGenerationService({
    this.delay = const Duration(milliseconds: 320),
    this.failAtStage,
  });

  final Duration delay;
  final HandcraftGenerationStage? failAtStage;

  @override
  Future<HandcraftProject> generate(
    HandcraftProject project, {
    FutureOr<void> Function(HandcraftGenerationStage stage)? onStageChanged,
  }) async {
    if (!project.hasRequiredInputs) {
      throw const HandcraftGenerationException('请先上传款式图和布料图');
    }

    var current = project.copyWith(
      status: HandcraftProjectStatus.generating,
      errorMessage: null,
    );

    current = await _advance(
      current,
      HandcraftGenerationStage.renderingPreview,
      onStageChanged,
    );
    current = current.copyWith(
      previewImageLocalPath: 'mock://handcraft/${project.id}/preview.png',
    );

    current = await _advance(
      current,
      HandcraftGenerationStage.draftingPattern,
      onStageChanged,
    );
    current = current.copyWith(
      patternImageLocalPath: 'mock://handcraft/${project.id}/pattern.png',
    );

    current = await _advance(
      current,
      HandcraftGenerationStage.makingVideo,
      onStageChanged,
    );
    current = current.copyWith(
      processVideoLocalPath: 'mock://handcraft/${project.id}/process.mp4',
    );

    current = await _advance(
      current,
      HandcraftGenerationStage.completed,
      onStageChanged,
    );

    return current.copyWith(
      status: HandcraftProjectStatus.completed,
      currentStage: HandcraftGenerationStage.completed,
      resultUploadStatus: HandcraftUploadStatus.localOnly,
      generationPrompt: _buildPrompt(current),
      generationSeed: current.id.hashCode.abs(),
      errorMessage: null,
    );
  }

  Future<HandcraftProject> _advance(
    HandcraftProject project,
    HandcraftGenerationStage stage,
    FutureOr<void> Function(HandcraftGenerationStage stage)? onStageChanged,
  ) async {
    if (failAtStage == stage) {
      throw HandcraftGenerationException('模拟生成在 ${stage.name} 阶段失败', stage: stage);
    }
    if (delay > Duration.zero) {
      await Future<void>.delayed(delay);
    }
    await onStageChanged?.call(stage);
    return project.copyWith(currentStage: stage);
  }

  String _buildPrompt(HandcraftProject project) {
    final dimensions = project.effectiveDimensions.values.entries
        .map((entry) => '${entry.key}=${entry.value}')
        .join(', ');
    return '使用 ${project.fabricImageLocalPath} 的面料制作 ${project.title}，尺寸参数：$dimensions';
  }
}
```

- [ ] **步骤 4：运行生成服务测试**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_generation_service_test.dart --no-pub
```

预期：测试通过。

- [ ] **步骤 5：提交任务 3**

运行：

```bash
git add apps/handcraft/lib/services/handcraft_generation_service.dart apps/handcraft/test/handcraft_generation_service_test.dart
git commit -m "feat(handcraft): 添加模拟生成服务"
```

---

### 任务 4：实现作品仓库和 Riverpod 状态

**文件：**
- 创建：`apps/handcraft/lib/repo/handcraft_project_repo.dart`
- 创建：`apps/handcraft/lib/providers/handcraft_providers.dart`
- 生成：`apps/handcraft/lib/providers/handcraft_providers.g.dart`
- 测试：`apps/handcraft/test/handcraft_project_repo_test.dart`
- 测试：`apps/handcraft/test/handcraft_providers_test.dart`

- [ ] **步骤 1：先写仓库测试**

创建 `apps/handcraft/test/handcraft_project_repo_test.dart`，覆盖：

- `createProject` 新建草稿。
- `saveProject` 更新本地图片路径。
- `getProject` 可以取回指定作品。
- `watchProjects` 会推送列表变化。
- `listProjects(status: ...)` 可以按状态筛选。

关键断言：

```dart
expect(updated.styleImageLocalPath, '/tmp/bag.jpg');
expect(await repo.getProject('project-1'), updated);
expect(completed, hasLength(1));
expect(drafts, isEmpty);
```

- [ ] **步骤 2：实现仓库**

创建 `apps/handcraft/lib/repo/handcraft_project_repo.dart`，必须包含：

- 抽象类 `HandcraftProjectRepo`。
- 方法 `createProject`、`getProject`、`listProjects`、`watchProjects`、`saveProject`。
- 类型别名 `HandcraftClock` 和 `HandcraftIdFactory`。
- 类 `InMemoryHandcraftProjectRepo`，内部使用 `Map<String, HandcraftProject>` 和 `StreamController<List<HandcraftProject>>.broadcast()`。
- `saveProject` 必须更新 `updatedAt` 并向 stream 发射新列表。
- `dispose` 必须关闭 stream controller。

- [ ] **步骤 3：运行仓库测试**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_project_repo_test.dart --no-pub
```

预期：测试通过。

- [ ] **步骤 4：先写提供器测试**

创建 `apps/handcraft/test/handcraft_providers_test.dart`：

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:puupee_handcraft/models/handcraft_project.dart';
import 'package:puupee_handcraft/providers/handcraft_providers.dart';
import 'package:puupee_handcraft/repo/handcraft_project_repo.dart';
import 'package:puupee_handcraft/services/handcraft_generation_service.dart';

void main() {
  test('工作台控制器可以创建草稿并完成模拟生成', () async {
    final repo = InMemoryHandcraftProjectRepo(
      clock: () => DateTime(2026, 5, 13, 9),
      idFactory: () => 'project-1',
    );
    final container = ProviderContainer(
      overrides: [
        handcraftProjectRepoProvider.overrideWithValue(repo),
        handcraftGenerationServiceProvider.overrideWithValue(
          const MockHandcraftGenerationService(delay: Duration.zero),
        ),
      ],
    );
    addTearDown(container.dispose);
    addTearDown(repo.dispose);

    final controller = container.read(handcraftStudioControllerProvider.notifier);
    final draft = await container.read(handcraftStudioControllerProvider.future);

    expect(draft.status, HandcraftProjectStatus.draft);

    await controller.updateInputs(
      styleImageLocalPath: '/tmp/bag.jpg',
      fabricImageLocalPath: '/tmp/canvas.jpg',
    );
    await controller.generate();

    final completed = await container.read(handcraftStudioControllerProvider.future);

    expect(completed.status, HandcraftProjectStatus.completed);
    expect(completed.previewImageLocalPath, isNotNull);
    expect(await repo.getProject('project-1'), completed);
  });
}
```

- [ ] **步骤 5：实现提供器**

创建 `apps/handcraft/lib/providers/handcraft_providers.dart`，必须包含：

- `@Riverpod(keepAlive: true) HandcraftProjectRepo handcraftProjectRepo(Ref ref)`。
- `@riverpod HandcraftGenerationService handcraftGenerationService(Ref ref)`。
- `@riverpod Stream<List<HandcraftProject>> handcraftProjects(Ref ref, {HandcraftProjectStatus? status})`。
- `@riverpod class HandcraftStudioController extends _$HandcraftStudioController`。
- 控制器方法 `updateInputs`、`updateDetails`、`generate`。
- `generate` 需要处理 `HandcraftGenerationException`，失败时保存 `status: failed`、`currentStage` 和 `errorMessage`。

- [ ] **步骤 6：生成 Riverpod 代码**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
dart run build_runner build --delete-conflicting-outputs
```

预期：生成 `lib/providers/handcraft_providers.g.dart`。

- [ ] **步骤 7：运行提供器测试**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_providers_test.dart --no-pub
```

预期：测试通过。

- [ ] **步骤 8：提交任务 4**

运行：

```bash
git add apps/handcraft/lib/repo/handcraft_project_repo.dart apps/handcraft/lib/providers/handcraft_providers.dart apps/handcraft/lib/providers/handcraft_providers.g.dart apps/handcraft/test/handcraft_project_repo_test.dart apps/handcraft/test/handcraft_providers_test.dart
git commit -m "feat(handcraft): 添加作品仓库和状态管理"
```

---

### 任务 5：实现路由、页面和组件

**文件：**
- 创建：`apps/handcraft/lib/router.dart`
- 生成：`apps/handcraft/lib/router.g.dart`
- 创建：`apps/handcraft/lib/components/adaptive_handcraft_shell.dart`
- 创建：`apps/handcraft/lib/components/handcraft_project_card.dart`
- 创建：`apps/handcraft/lib/components/handcraft_media_input_card.dart`
- 创建：`apps/handcraft/lib/components/handcraft_dimension_editor.dart`
- 创建：`apps/handcraft/lib/components/handcraft_generation_panel.dart`
- 创建：`apps/handcraft/lib/pages/handcraft_studio_page.dart`
- 创建：`apps/handcraft/lib/pages/handcraft_projects_page.dart`
- 创建：`apps/handcraft/lib/pages/handcraft_assets_page.dart`
- 测试：`apps/handcraft/test/handcraft_page_smoke_test.dart`

- [ ] **步骤 1：先写页面冒烟测试**

创建 `apps/handcraft/test/handcraft_page_smoke_test.dart`：

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:puupee_handcraft/pages/handcraft_studio_page.dart';
import 'package:shadcn_flutter/shadcn_flutter.dart';

void main() {
  testWidgets('创作页显示核心工作流', (tester) async {
    await tester.pumpWidget(
      const ProviderScope(
        child: ShadcnApp(home: HandcraftStudioPage()),
      ),
    );

    await tester.pumpAndSettle();

    expect(find.text('小汪手工'), findsOneWidget);
    expect(find.text('款式图'), findsWidgets);
    expect(find.text('布料图'), findsWidgets);
    expect(find.text('生成效果'), findsOneWidget);
  });
}
```

- [ ] **步骤 2：运行冒烟测试确认失败**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_page_smoke_test.dart --no-pub
```

预期：页面文件不存在，测试失败。

- [ ] **步骤 3：实现自适应 Shell**

创建 `apps/handcraft/lib/components/adaptive_handcraft_shell.dart`，结构参考 `apps/inventory/lib/components/adaptive_inventory_shell.dart`。

桌面端使用：

- `AdaptiveLayout`
- `HomeIcon(appId: 'handcraft')`
- `appTitle: '小汪手工'`

菜单项固定为：

```dart
static const List<AdaptiveMenuItem> _menuItems = [
  AdaptiveMenuItem(label: '创作', icon: Icons.auto_fix_high),
  AdaptiveMenuItem(label: '作品', icon: Icons.collections_bookmark_outlined),
  AdaptiveMenuItem(label: '素材', icon: Icons.perm_media_outlined),
  AdaptiveMenuItem(
    label: '我的',
    icon: Icons.person_outline,
    position: MenuItemPosition.bottom,
  ),
];
```

路由跳转固定为：

```dart
case 0:
  context.go('/handcraft');
case 1:
  context.go('/handcraft/projects');
case 2:
  context.go('/handcraft/assets');
case 3:
  context.go('/handcraft/settings');
```

- [ ] **步骤 4：实现组件**

创建以下组件：

- `HandcraftProjectCard`：显示作品标题、物品类型、状态，可点击。
- `HandcraftMediaInputCard`：显示标题和本地路径输入框；`TextField.placeholder` 使用 `const Text('输入本地图片路径')`。
- `HandcraftDimensionEditor`：显示 `project.effectiveDimensions` 的所有字段和值。
- `HandcraftGenerationPanel`：显示当前阶段、生成按钮、效果图、裁剪图、过程视频、错误信息。

生成按钮规则：

```dart
final isGenerating = project.status == HandcraftProjectStatus.generating;
final canGenerate = project.hasRequiredInputs && !isGenerating;
```

按钮禁用规则：

```dart
onPressed: canGenerate ? onGenerate : null
```

- [ ] **步骤 5：实现页面**

创建 `HandcraftStudioPage`：

- `ConsumerWidget`
- 使用 `AdaptiveScaffold(title: '小汪手工')`
- 读取 `handcraftStudioControllerProvider`
- 移动端宽度 `< 700` 时用单列向导布局
- 桌面端宽度 `>= 700` 时用左右两列布局
- 必须出现 `款式图`、`布料图`、`尺寸`、`生成效果`

创建 `HandcraftProjectsPage`：

- 读取 `handcraftProjectsProvider()`
- 无项目显示 `还没有手工作品`
- 有项目显示 `HandcraftProjectCard`

创建 `HandcraftAssetsPage`：

- 读取所有项目
- 按 `输入素材` 和 `生成结果` 两组列出非空媒体路径或 URL
- 无素材时显示 `还没有素材引用`

- [ ] **步骤 6：实现类型化路由和设置页路由**

创建 `apps/handcraft/lib/router.dart`：

- `@TypedStatefulShellRoute<HandcraftMainStatefulShellRoute>`
- 四个分支：`/handcraft`、`/handcraft/projects`、`/handcraft/assets`、`/handcraft/settings`
- 设置页复用 `SettingsShellPage`、`SettingsHomePage`、`AccountPage`、`StoragePage`、`SyncSettingsPage`、`DataPage`、`DevicesPage`、`DeveloperPage`、`AboutPage`、`FeedbackPage`、`FeedbackSuccessPage`、`FeedbackListPage`、`FeedbackDetailPage`、`RuntimeInfoPage`、`RecycleBinPage`、`SubscriptionPage`
- `/handcraft/settings` 在移动端返回 `SettingsHomePage`，桌面端返回 `SizedBox.shrink()`

- [ ] **步骤 7：生成路由和提供器代码**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
dart run build_runner build --delete-conflicting-outputs
```

预期：生成或更新 `lib/router.g.dart` 和 `lib/providers/handcraft_providers.g.dart`。

- [ ] **步骤 8：运行页面冒烟测试**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test test/handcraft_page_smoke_test.dart --no-pub
```

预期：测试通过。

- [ ] **步骤 9：提交任务 5**

运行：

```bash
git add apps/handcraft/lib/router.dart apps/handcraft/lib/router.g.dart apps/handcraft/lib/components apps/handcraft/lib/pages apps/handcraft/lib/models/handcraft_project.dart apps/handcraft/test/handcraft_page_smoke_test.dart
git commit -m "feat(handcraft): 添加创作工作台和导航页面"
```

---

### 任务 6：添加 Android 平台工程

**文件：**
- 创建：`apps/handcraft/android/.gitignore`
- 创建：`apps/handcraft/android/app/build.gradle.kts`
- 创建：`apps/handcraft/android/app/proguard-rules.pro`
- 创建：`apps/handcraft/android/app/src/debug/AndroidManifest.xml`
- 创建：`apps/handcraft/android/app/src/main/AndroidManifest.xml`
- 创建：`apps/handcraft/android/app/src/main/kotlin/com/puupee/handcraft/MainActivity.kt`
- 创建：`apps/handcraft/android/app/src/main/res/values/strings.xml`
- 创建：`apps/handcraft/android/app/src/main/res/values-zh/strings.xml`
- 创建：`apps/handcraft/android/app/src/main/res/values/styles.xml`
- 创建：`apps/handcraft/android/app/src/main/res/values-night/styles.xml`
- 创建：`apps/handcraft/android/app/src/main/res/drawable/launch_background.xml`
- 创建：`apps/handcraft/android/app/src/main/res/drawable-v21/launch_background.xml`
- 创建：`apps/handcraft/android/app/src/main/res/xml/file_paths.xml`
- 创建：`apps/handcraft/android/app/src/profile/AndroidManifest.xml`
- 创建：`apps/handcraft/android/build.gradle.kts`
- 创建：`apps/handcraft/android/gradle.properties`
- 创建：`apps/handcraft/android/settings.gradle.kts`
- 创建：`apps/handcraft/android/gradle/wrapper/gradle-wrapper.properties`
- 复制：`apps/inventory/android/app/src/main/res/mipmap-*` 下的启动图标资源。

- [ ] **步骤 1：从 inventory 复制 Android 模板**

运行：

```bash
mkdir -p apps/handcraft
rsync -a \
  --exclude='.gradle' \
  --exclude='local.properties' \
  --exclude='gradlew' \
  --exclude='gradlew.bat' \
  --exclude='gradle/wrapper/gradle-wrapper.jar' \
  --exclude='app/src/main/java' \
  apps/inventory/android/ \
  apps/handcraft/android/
```

预期：`apps/handcraft/android/app/src/main/AndroidManifest.xml` 存在。

- [ ] **步骤 2：修改 Kotlin 包路径**

运行：

```bash
mkdir -p apps/handcraft/android/app/src/main/kotlin/com/puupee/handcraft
mv apps/handcraft/android/app/src/main/kotlin/com/puupee/inventory/MainActivity.kt \
  apps/handcraft/android/app/src/main/kotlin/com/puupee/handcraft/MainActivity.kt
find apps/handcraft/android -type d -empty -delete
```

替换 `MainActivity.kt` 内容：

```kotlin
package com.puupee.handcraft

import io.flutter.embedding.android.FlutterActivity

class MainActivity : FlutterActivity()
```

- [ ] **步骤 3：修改 Gradle 包名和应用 ID**

在 `apps/handcraft/android/app/build.gradle.kts` 中替换：

```kotlin
namespace = "com.puupee.inventory"
applicationId = "com.puupee.inventory"
```

为：

```kotlin
namespace = "com.puupee.handcraft"
applicationId = "com.puupee.handcraft"
```

确认文件仍包含：

```kotlin
kotlin {
    jvmToolchain(17)
}

compileOptions {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
    isCoreLibraryDesugaringEnabled = true
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.1.5")
}
```

- [ ] **步骤 4：修改应用名称**

替换 `apps/handcraft/android/app/src/main/res/values/strings.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Puupee Handcraft</string>
</resources>
```

替换 `apps/handcraft/android/app/src/main/res/values-zh/strings.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">小汪手工</string>
</resources>
```

- [ ] **步骤 5：确认 Manifest 保留关键配置**

确认 `AndroidManifest.xml` 使用：

```xml
android:label="@string/app_name"
```

确认保留 FileProvider：

```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

- [ ] **步骤 6：删除本地平台垃圾文件**

运行：

```bash
rm -rf apps/handcraft/android/.gradle \
  apps/handcraft/android/local.properties \
  apps/handcraft/android/app/src/main/java \
  apps/handcraft/android/gradle/wrapper/gradle-wrapper.jar \
  apps/handcraft/android/gradlew \
  apps/handcraft/android/gradlew.bat
```

- [ ] **步骤 7：运行 Android 文件存在性检查**

运行：

```bash
test -f apps/handcraft/android/app/src/main/AndroidManifest.xml
test -f apps/handcraft/android/app/src/main/kotlin/com/puupee/handcraft/MainActivity.kt
test -f apps/handcraft/android/app/src/main/res/values/strings.xml
test -f apps/handcraft/android/app/src/main/res/values-zh/strings.xml
```

预期：全部退出码为 0。

- [ ] **步骤 8：提交任务 6**

运行：

```bash
git add apps/handcraft/android
git commit -m "build(handcraft): 添加 Android 平台配置"
```

---

### 任务 7：添加 README 并完成最终验证

**文件：**
- 创建：`apps/handcraft/README.md`
- 验证：`apps/handcraft/lib/router.g.dart`
- 验证：`apps/handcraft/lib/providers/handcraft_providers.g.dart`

- [ ] **步骤 1：创建 README**

创建 `apps/handcraft/README.md`：

```markdown
# 小汪手工

小汪手工是布料成品效果、尺寸裁剪图和制作过程视频的模拟生成工具。

## 第一版范围

- 上传款式图和布料图的本地路径。
- 选择包、衣服或其他物品类型。
- 填写目标尺寸；未填写时使用模拟估算尺寸。
- 使用本地模拟服务生成成品效果图、裁剪图和过程视频引用。
- 保存同步型作品历史的数据形状，预留远程媒体 URL 和上传状态字段。

## 开发

```bash
cd apps/handcraft
dart run build_runner build --delete-conflicting-outputs
dart analyze
flutter test
flutter build apk --debug --target-platform android-arm64 --no-pub
```
```

- [ ] **步骤 2：运行代码生成**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
dart run build_runner build --delete-conflicting-outputs
```

预期：生成文件保持最新。

- [ ] **步骤 3：运行 EnvConfig 相关 analyze**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps
dart analyze apps/handcraft/lib/env.dart apps/handcraft/lib/main.dart apps/handcraft/test/env_test.dart
```

预期：输出 `No issues found!`。

- [ ] **步骤 4：运行 handcraft 全部测试**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter test --no-pub
```

预期：全部测试通过。

- [ ] **步骤 5：构建 Android debug APK**

运行：

```bash
cd /Users/j/repos/puupees/puupee-apps/apps/handcraft
flutter build apk --debug --target-platform android-arm64 --no-pub
```

预期：debug APK 构建成功。

- [ ] **步骤 6：确认没有平台本地文件进入暂存区**

运行：

```bash
git status --short
```

预期：输出中不包含：

```text
apps/handcraft/android/.gradle
apps/handcraft/android/local.properties
apps/handcraft/android/gradle/wrapper/gradle-wrapper.jar
apps/handcraft/android/gradlew
apps/handcraft/android/gradlew.bat
```

- [ ] **步骤 7：提交任务 7**

如果 generated 文件有变化，运行：

```bash
git add apps/handcraft/README.md apps/handcraft/lib/router.g.dart apps/handcraft/lib/providers/handcraft_providers.g.dart
git commit -m "docs(handcraft): 添加应用说明和生成文件"
```

如果只有 README 有变化，运行：

```bash
git add apps/handcraft/README.md
git commit -m "docs(handcraft): 添加应用说明"
```

---

## 自审记录

- 规格覆盖：本计划覆盖应用骨架、工作区注册、EnvConfig、Android 配置、作品模型、两张图片输入、本地和远程媒体字段、上传状态、手动或模拟尺寸、模拟效果图/裁剪图/视频生成、作品历史仓库、Riverpod 状态、创作/作品/素材/我的导航、测试、analyze 和 APK 构建。
- 范围边界：真实 AI 生成、对象存储上传、精确制版算法、完整跨设备媒体同步不进入第一版。
- 类型一致性：测试、模型、服务、仓库、提供器、页面、路由统一使用 `Handcraft` 前缀。
- 语言一致性：正文、任务名、步骤说明、验收说明均为中文；路径、命令、Dart/Kotlin/XML 标识符按工程需要保留英文。
