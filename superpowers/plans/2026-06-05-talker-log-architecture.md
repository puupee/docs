# Talker Log Architecture Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Talker-based custom logging layer so `sync_server.dart` writes logs under a filterable `sync-server` key with searchable scene labels.

**Architecture:** Add a focused logging model and logger facade to `felorx_utilities`, then migrate only `packages/sync/felorx_sync/lib/src/sync_server.dart` to use it. The global `talker` remains the single logging backend, and the existing developer `TalkerScreen(talker: talker)` remains the UI.

**Tech Stack:** Dart 3.8, `talker` 5.1.17, Dart `test`, `felorx_utilities`, `felorx_sync`.

---

## File Structure

- Create `packages/core/felorx_utilities/test/talker_test.dart`
  - Unit tests for `FelorxTalkerLog`, `FelorxLogger`, key registration, and `createTalker(settings:)`.
- Modify `packages/core/felorx_utilities/lib/talker.dart`
  - Define module/scene constants, custom log model, logger facade, key registration, and fix `createTalker`.
- Modify `packages/core/felorx_utilities/lib/felorx_utilities.dart`
  - Already exports `talker.dart`; verify no extra export is needed.
- Modify `packages/sync/felorx_sync/lib/src/sync_server.dart`
  - Replace direct `Talker` usage with `FelorxLogger.syncServer()`.
  - Migrate existing `_logger.info/warning/error` calls to named parameters and scene labels.
- Create `packages/sync/felorx_sync/test/sync_server_logging_test.dart`
  - Lightweight tests proving sync-server logs use key `sync-server` and scene-prefixed messages.

---

### Task 1: Add Felorx Talker Infrastructure Tests

**Files:**
- Create: `packages/core/felorx_utilities/test/talker_test.dart`
- Modify: `packages/core/felorx_utilities/lib/talker.dart`

- [ ] **Step 1: Write the failing infrastructure tests**

Create `packages/core/felorx_utilities/test/talker_test.dart`:

```dart
import 'package:felorx_utilities/talker.dart';
import 'package:talker/talker.dart';
import 'package:test/test.dart';

void main() {
  group('FelorxTalkerLog', () {
    test('应该写入模块 key、等级和场景前缀', () {
      final stackTrace = StackTrace.current;
      final exception = StateError('boom');

      final log = FelorxTalkerLog(
        '处理配对请求失败',
        module: FelorxLogModule.syncServer,
        scene: FelorxLogScene.pairing,
        logLevel: LogLevel.error,
        exception: exception,
        stackTrace: stackTrace,
      );

      expect(log.key, equals(FelorxLogModule.syncServer));
      expect(log.title, equals(FelorxLogModule.syncServer));
      expect(log.logLevel, equals(LogLevel.error));
      expect(log.message, equals('[pairing] 处理配对请求失败'));
      expect(log.exception, same(exception));
      expect(log.stackTrace, same(stackTrace));
    });

    test('没有场景时不添加空前缀', () {
      final log = FelorxTalkerLog(
        '同步服务器已停止',
        module: FelorxLogModule.syncServer,
        logLevel: LogLevel.info,
      );

      expect(log.message, equals('同步服务器已停止'));
    });
  });

  group('FelorxLogger', () {
    test('应该通过 logCustom 写入 Talker history', () {
      final loggerBackend = createTalker(
        settings: TalkerSettings(useConsoleLogs: false),
      );
      final logger = FelorxLogger.syncServer(backend: loggerBackend);

      logger.warning(
        'API Key 验证失败',
        scene: FelorxLogScene.auth,
        exception: const FormatException('bad api key'),
        stackTrace: StackTrace.empty,
      );

      expect(loggerBackend.history, hasLength(1));
      final log = loggerBackend.history.single as FelorxTalkerLog;
      expect(log.key, equals(FelorxLogModule.syncServer));
      expect(log.logLevel, equals(LogLevel.warning));
      expect(log.message, equals('[auth] API Key 验证失败'));
      expect(log.exception, isA<FormatException>());
      expect(log.stackTrace, same(StackTrace.empty));
    });

    test('createTalker 应该保留传入 settings', () {
      final loggerBackend = createTalker(
        settings: TalkerSettings(
          useConsoleLogs: false,
          titles: const {FelorxLogModule.syncServer: 'Sync Server'},
        ),
      );

      expect(loggerBackend.settings.useConsoleLogs, isFalse);
      expect(
        loggerBackend.settings.getTitleByKey(FelorxLogModule.syncServer),
        equals('Sync Server'),
      );
    });

    test('registerFelorxTalkerKeys 应该注册 sync-server key', () {
      final loggerBackend = Talker(
        settings: TalkerSettings(
          useConsoleLogs: false,
          titles: const {},
        ),
      );

      registerFelorxTalkerKeys(loggerBackend);

      expect(
        loggerBackend.settings.registeredKeys,
        contains(FelorxLogModule.syncServer),
      );
      expect(
        loggerBackend.settings.getTitleByKey(FelorxLogModule.syncServer),
        equals(FelorxLogModule.syncServer),
      );
    });
  });
}
```

- [ ] **Step 2: Run the new test to verify it fails**

Run:

```bash
cd packages/core/felorx_utilities
dart test test/talker_test.dart
```

Expected: FAIL with missing symbols such as `FelorxTalkerLog`, `FelorxLogModule`, `FelorxLogScene`, `FelorxLogger`, or `registerFelorxTalkerKeys`.

- [ ] **Step 3: Implement the minimal logging infrastructure**

Replace `packages/core/felorx_utilities/lib/talker.dart` with:

```dart
import 'package:talker/talker.dart';

abstract final class FelorxLogModule {
  static const syncServer = 'sync-server';
  static const syncClient = 'sync-client';
  static const syncStorage = 'sync-storage';
  static const syncAuth = 'sync-auth';
  static const syncMcp = 'sync-mcp';
}

abstract final class FelorxLogScene {
  static const lifecycle = 'lifecycle';
  static const db = 'db';
  static const mq = 'mq';
  static const http = 'http';
  static const storage = 'storage';
  static const pairing = 'pairing';
  static const auth = 'auth';
  static const mcp = 'mcp';
  static const cache = 'cache';
  static const crdt = 'crdt';
  static const contentEvent = 'content-event';
  static const version = 'version';
  static const appInfo = 'app-info';
}

class FelorxTalkerLog extends TalkerLog {
  FelorxTalkerLog(
    String message, {
    required String module,
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
    LogLevel? logLevel,
  }) : super(
         _formatMessage(message, scene),
         key: module,
         title: module,
         exception: exception,
         stackTrace: stackTrace,
         logLevel: logLevel,
       );

  static String _formatMessage(String message, String? scene) {
    final normalizedScene = scene?.trim();
    if (normalizedScene == null || normalizedScene.isEmpty) {
      return message;
    }
    return '[$normalizedScene] $message';
  }
}

class FelorxLogger {
  const FelorxLogger({
    required this.talker,
    required this.module,
  });

  factory FelorxLogger.syncServer({Talker? backend}) {
    return FelorxLogger(
      talker: backend ?? talker,
      module: FelorxLogModule.syncServer,
    );
  }

  final Talker talker;
  final String module;

  void critical(
    Object? message, {
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
  }) {
    _log(
      message,
      scene: scene,
      exception: exception,
      stackTrace: stackTrace,
      logLevel: LogLevel.critical,
    );
  }

  void debug(
    Object? message, {
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
  }) {
    _log(
      message,
      scene: scene,
      exception: exception,
      stackTrace: stackTrace,
      logLevel: LogLevel.debug,
    );
  }

  void error(
    Object? message, {
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
  }) {
    _log(
      message,
      scene: scene,
      exception: exception,
      stackTrace: stackTrace,
      logLevel: LogLevel.error,
    );
  }

  void info(
    Object? message, {
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
  }) {
    _log(
      message,
      scene: scene,
      exception: exception,
      stackTrace: stackTrace,
      logLevel: LogLevel.info,
    );
  }

  void verbose(
    Object? message, {
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
  }) {
    _log(
      message,
      scene: scene,
      exception: exception,
      stackTrace: stackTrace,
      logLevel: LogLevel.verbose,
    );
  }

  void warning(
    Object? message, {
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
  }) {
    _log(
      message,
      scene: scene,
      exception: exception,
      stackTrace: stackTrace,
      logLevel: LogLevel.warning,
    );
  }

  void _log(
    Object? message, {
    required LogLevel logLevel,
    String? scene,
    Object? exception,
    StackTrace? stackTrace,
  }) {
    talker.logCustom(
      FelorxTalkerLog(
        message?.toString() ?? '',
        module: module,
        scene: scene,
        exception: exception,
        stackTrace: stackTrace,
        logLevel: logLevel,
      ),
    );
  }
}

Talker createTalker({TalkerSettings? settings}) {
  final instance = Talker(settings: settings);
  registerFelorxTalkerKeys(instance);
  return instance;
}

void registerFelorxTalkerKeys(Talker talker) {
  talker.settings.registerKeys(const [FelorxLogModule.syncServer]);
  talker.settings.titles[FelorxLogModule.syncServer] =
      FelorxLogModule.syncServer;
}

Talker talker = createTalker();
```

- [ ] **Step 4: Run the test to verify it passes**

Run:

```bash
cd packages/core/felorx_utilities
dart test test/talker_test.dart
```

Expected: PASS.

- [ ] **Step 5: Run formatter**

Run:

```bash
dart format lib/talker.dart test/talker_test.dart
```

Expected: formatter exits with code 0.

- [ ] **Step 6: Commit infrastructure**

Run:

```bash
git add packages/core/felorx_utilities/lib/talker.dart packages/core/felorx_utilities/test/talker_test.dart
git commit -m "feat(felorx_utilities): 添加 Talker 自定义日志基础设施"
```

Expected: commit succeeds.

---

### Task 2: Add Sync Server Logging Test

**Files:**
- Create: `packages/sync/felorx_sync/test/sync_server_logging_test.dart`
- Modify: `packages/sync/felorx_sync/lib/src/sync_server.dart`

- [ ] **Step 1: Write a sync-server logger behavior test**

Create `packages/sync/felorx_sync/test/sync_server_logging_test.dart`:

```dart
import 'package:felorx_utilities/felorx_utilities.dart';
import 'package:talker/talker.dart';
import 'package:test/test.dart';

void main() {
  group('sync-server 日志', () {
    test('应该写入 sync-server key 和 lifecycle 场景', () {
      final loggerBackend = createTalker(
        settings: TalkerSettings(useConsoleLogs: false),
      );
      final logger = FelorxLogger.syncServer(backend: loggerBackend);

      logger.info(
        'Serving at http://0.0.0.0:8080',
        scene: FelorxLogScene.lifecycle,
      );

      expect(loggerBackend.history, hasLength(1));
      final log = loggerBackend.history.single;
      expect(log.key, equals(FelorxLogModule.syncServer));
      expect(log.logLevel, equals(LogLevel.info));
      expect(log.message, equals('[lifecycle] Serving at http://0.0.0.0:8080'));
    });

    test('应该传递错误对象和堆栈', () {
      final loggerBackend = createTalker(
        settings: TalkerSettings(useConsoleLogs: false),
      );
      final logger = FelorxLogger.syncServer(backend: loggerBackend);
      final exception = Exception('storage failed');

      logger.error(
        '本地上传失败',
        scene: FelorxLogScene.storage,
        exception: exception,
        stackTrace: StackTrace.empty,
      );

      final log = loggerBackend.history.single;
      expect(log.key, equals(FelorxLogModule.syncServer));
      expect(log.logLevel, equals(LogLevel.error));
      expect(log.message, equals('[storage] 本地上传失败'));
      expect(log.exception, same(exception));
      expect(log.stackTrace, same(StackTrace.empty));
    });
  });
}
```

- [ ] **Step 2: Run the test**

Run:

```bash
cd packages/sync/felorx_sync
dart test test/sync_server_logging_test.dart
```

Expected: PASS if Task 1 is complete. If it fails because `felorx_sync` does not see the local workspace version of `felorx_utilities`, run `melos bootstrap` from the repository root and rerun the test.

- [ ] **Step 3: Format the new test**

Run:

```bash
dart format test/sync_server_logging_test.dart
```

Expected: formatter exits with code 0.

- [ ] **Step 4: Commit the test**

Run:

```bash
git add packages/sync/felorx_sync/test/sync_server_logging_test.dart
git commit -m "test(felorx_sync): 覆盖 sync-server 自定义日志"
```

Expected: commit succeeds.

---

### Task 3: Migrate sync_server.dart Logger Field and Imports

**Files:**
- Modify: `packages/sync/felorx_sync/lib/src/sync_server.dart`

- [ ] **Step 1: Remove direct Talker import**

In `packages/sync/felorx_sync/lib/src/sync_server.dart`, remove:

```dart
import 'package:talker/talker.dart';
```

Keep:

```dart
import 'package:felorx_utilities/felorx_utilities.dart';
```

- [ ] **Step 2: Replace the logger field**

Change:

```dart
final Talker _logger = talker;
```

To:

```dart
final FelorxLogger _logger = FelorxLogger.syncServer();
```

- [ ] **Step 3: Run analyzer to see call-site errors**

Run:

```bash
cd packages/sync/felorx_sync
dart analyze lib/src/sync_server.dart
```

Expected: FAIL with errors for positional exception/stackTrace arguments on `_logger.warning` and `_logger.error`. These errors drive Task 4.

- [ ] **Step 4: Commit only if analyzer shows no unrelated issues after Task 4**

Do not commit this task alone if `sync_server.dart` does not compile. The commit for this field/import change happens together with Task 4.

---

### Task 4: Migrate sync_server.dart Log Calls by Scene

**Files:**
- Modify: `packages/sync/felorx_sync/lib/src/sync_server.dart`

- [ ] **Step 1: Convert app-info logs**

Change:

```dart
_logger.warning('解析 App 信息失败: $appName', e, stackTrace);
```

To:

```dart
_logger.warning(
  '解析 App 信息失败: $appName',
  scene: FelorxLogScene.appInfo,
  exception: e,
  stackTrace: stackTrace,
);
```

- [ ] **Step 2: Convert lifecycle logs**

Use `scene: FelorxLogScene.lifecycle` for these messages:

```dart
_logger.info('Sync server already started');
_logger.info('Serving at http://${server.address.host}:${server.port}');
_logger.info('同步服务器已停止');
_logger.info('关闭空闲连接: (${client.peerId?.short ?? "unknown"})');
_logger.info('已关闭 ${idleClients.length} 个空闲连接');
_logger.warning('关闭客户端连接失败: $e');
```

Converted examples:

```dart
_logger.info(
  'Sync server already started',
  scene: FelorxLogScene.lifecycle,
);
```

```dart
_logger.warning(
  '关闭客户端连接失败: $e',
  scene: FelorxLogScene.lifecycle,
);
```

- [ ] **Step 3: Convert database and CRDT logs**

Use `scene: FelorxLogScene.db` for:

```dart
_logger.info('Fields: $fields');
_logger.error('Failed to open Postgres database.', e, stackTrace);
```

Use `scene: FelorxLogScene.crdt` for:

```dart
_logger.info('[validateRecord] Record creator is local user');
```

Converted error:

```dart
_logger.error(
  'Failed to open Postgres database.',
  scene: FelorxLogScene.db,
  exception: e,
  stackTrace: stackTrace,
);
```

- [ ] **Step 4: Convert MQ and content event logs**

Use `scene: FelorxLogScene.mq` for:

```dart
_logger.warning('Failed to initialize AMQP exchange', e, stackTrace);
```

Use `scene: FelorxLogScene.contentEvent` for:

```dart
_logger.error('发布 Felorx content event 失败', e, stackTrace);
```

Converted examples:

```dart
_logger.warning(
  'Failed to initialize AMQP exchange',
  scene: FelorxLogScene.mq,
  exception: e,
  stackTrace: stackTrace,
);
```

```dart
_logger.error(
  '发布 Felorx content event 失败',
  scene: FelorxLogScene.contentEvent,
  exception: e,
  stackTrace: stackTrace,
);
```

- [ ] **Step 5: Convert MCP logs**

Use `scene: FelorxLogScene.mcp` for:

```dart
_logger.info('MCP 服务器端点已注册: GET/POST /mcp (需要认证)');
_logger.info('MCP 服务器端点: http://${server.address.host}:${server.port}/mcp');
```

Converted example:

```dart
_logger.info(
  'MCP 服务器端点已注册: GET/POST /mcp (需要认证)',
  scene: FelorxLogScene.mcp,
);
```

- [ ] **Step 6: Convert HTTP request/response logs**

Use `scene: FelorxLogScene.http` inside `_logNonSuccessRequests` and health/request handling for:

```dart
_logger.error('健康检查失败: $e');
_logger.error('读取请求体用于错误日志失败: ${request.method} ${request.requestedUri}', e, stackTrace);
_logger.warning(_formatNonSuccessLog(...));
_logger.error(_formatExceptionLog(...), e, stackTrace);
```

Converted examples:

```dart
_logger.error(
  '健康检查失败: $e',
  scene: FelorxLogScene.http,
);
```

```dart
_logger.error(
  _formatExceptionLog(
    request: bufferedRequest.request,
    requestBody: bufferedRequest.body,
    elapsed: DateTime.now().difference(startedAt),
  ),
  scene: FelorxLogScene.http,
  exception: e,
  stackTrace: stackTrace,
);
```

- [ ] **Step 7: Convert storage logs**

Use `scene: FelorxLogScene.storage` for file upload/download/storage credential logs, including:

```dart
_logger.warning('rapidCode hash 计算异常', e, stackTrace);
_logger.warning('清理临时目录失败: ${dir.path} - $e');
_logger.error('onFileLocalUploaded 回调失败', e, stackTrace);
_logger.warning('onFileLocalProgress 回调失败', e, stackTrace);
_logger.error('本地上传失败', e, stackTrace);
_logger.error('读取文件失败: $e');
_logger.warning('cdnDomain 为 null 且没有默认值: usage=$usage');
_logger.warning('存储空间已满: $errorMessage');
_logger.warning('API 业务错误: $errorCode - $errorMessage');
_logger.warning('解析错误响应失败: $parseError');
_logger.error('获取文件凭证失败: ${e.message}', e, e.stackTrace);
_logger.error('获取文件凭证时发生未预期的错误', e, stackTrace);
_logger.error('Error getting user storage', e, stackTrace);
```

Converted example:

```dart
_logger.error(
  '获取文件凭证失败: ${e.message}',
  scene: FelorxLogScene.storage,
  exception: e,
  stackTrace: e.stackTrace,
);
```

- [ ] **Step 8: Convert auth and Felorx API logs**

Use `scene: FelorxLogScene.auth` for:

```dart
_logger.info('配对码认证，跳过权限检查: felorxId=$felorxId');
_logger.error('检查 Felorx 访问权限失败: $e', e, stackTrace);
_logger.warning('请求 User-Agent 版本校验失败: $userAgent', e);
```

Use `scene: FelorxLogScene.http` for Felorx CRUD endpoint failures:

```dart
_logger.error('获取 Felorx 失败: $e', e, stackTrace);
_logger.error('查询 Felorx 集合失败: $e', e, stackTrace);
_logger.error('更新 Felorx 失败: $e', e, stackTrace);
_logger.error('删除 Felorx 失败: $e', e, stackTrace);
_logger.error('统计 Felorx 失败: $e', e, stackTrace);
_logger.error('批量创建 Felorx 失败: $e', e, stackTrace);
_logger.error('批量更新 Felorx 失败: $e', e, stackTrace);
_logger.error('批量删除 Felorx 失败: $e', e, stackTrace);
```

Converted auth warning:

```dart
_logger.warning(
  '请求 User-Agent 版本校验失败: $userAgent',
  scene: FelorxLogScene.auth,
  exception: e,
);
```

- [ ] **Step 9: Convert pairing logs**

Use `scene: FelorxLogScene.pairing` for:

```dart
_logger.error('处理配对请求失败: $e', e, stackTrace);
_logger.warning('接收端 token 验证失败: ${verificationResult.error}');
_logger.info('接收端 token 验证成功，接收端 nodeId: $receiverNodeId');
_logger.warning('解析或验证接收端公钥失败: $e');
_logger.info('已保存接收端配对 token: $receiverNodeId');
_logger.info('已保存接收端公钥: $receiverNodeId');
_logger.error('处理配对确认失败: $e', e, stackTrace);
```

Converted example:

```dart
_logger.error(
  '处理配对确认失败: $e',
  scene: FelorxLogScene.pairing,
  exception: e,
  stackTrace: stackTrace,
);
```

- [ ] **Step 10: Convert cache logs**

Use `scene: FelorxLogScene.cache` for:

```dart
_logger.info('清理了 ${keysToRemove.length} 个过期缓存项');
```

Converted:

```dart
_logger.info(
  '清理了 ${keysToRemove.length} 个过期缓存项',
  scene: FelorxLogScene.cache,
);
```

- [ ] **Step 11: Find and convert every remaining call**

Run:

```bash
rg -n "_logger\\.(info|warning|error|debug|verbose|critical)\\([^\\n]*, [^\\n]*, [^\\n]*\\)" packages/sync/felorx_sync/lib/src/sync_server.dart
```

Expected: no output.

Run:

```bash
rg -n "_logger\\." packages/sync/felorx_sync/lib/src/sync_server.dart
```

Expected: every result has `scene:` in the call block.

- [ ] **Step 12: Format and analyze**

Run:

```bash
cd packages/sync/felorx_sync
dart format lib/src/sync_server.dart
dart analyze lib/src/sync_server.dart
```

Expected: formatter exits with code 0 and analyzer exits with code 0.

- [ ] **Step 13: Commit sync_server migration**

Run:

```bash
git add packages/sync/felorx_sync/lib/src/sync_server.dart
git commit -m "refactor(felorx_sync): 使用 sync-server 自定义日志"
```

Expected: commit succeeds.

---

### Task 5: Run Focused Tests and Package Analysis

**Files:**
- No source files expected.

- [ ] **Step 1: Run felorx_utilities tests**

Run:

```bash
cd packages/core/felorx_utilities
dart test test/talker_test.dart
dart analyze lib/talker.dart test/talker_test.dart
```

Expected: both commands exit with code 0.

- [ ] **Step 2: Run felorx_sync focused tests**

Run:

```bash
cd packages/sync/felorx_sync
dart test test/sync_server_logging_test.dart
dart test test/sync_server_test.dart
dart analyze lib/src/sync_server.dart test/sync_server_logging_test.dart
```

Expected: all commands exit with code 0.

- [ ] **Step 3: Run repository-level targeted search checks**

Run:

```bash
rg -n "final Talker _logger = talker" packages/sync/felorx_sync/lib/src/sync_server.dart
```

Expected: no output.

Run:

```bash
rg -n "import 'package:talker/talker.dart';" packages/sync/felorx_sync/lib/src/sync_server.dart
```

Expected: no output.

Run:

```bash
rg -n "FelorxLogger.syncServer" packages/sync/felorx_sync/lib/src/sync_server.dart
```

Expected: one output line for the `_logger` field.

- [ ] **Step 4: Commit verification notes only if files changed**

If no files changed during verification, do not commit. If formatter changed files, run:

```bash
git status --short
git add packages/core/felorx_utilities packages/sync/felorx_sync
git commit -m "chore(felorx_sync): 格式化 sync-server 日志迁移"
```

Expected: commit succeeds only when there are formatter changes.

---

### Task 6: Final Review Before PR or Handoff

**Files:**
- Review only.

- [ ] **Step 1: Inspect final diff**

Run:

```bash
git status --short
git log --oneline -n 5
```

Expected: working tree is clean, and recent commits include:

```text
feat(felorx_utilities): 添加 Talker 自定义日志基础设施
test(felorx_sync): 覆盖 sync-server 自定义日志
refactor(felorx_sync): 使用 sync-server 自定义日志
```

- [ ] **Step 2: Confirm spec coverage**

Open `docs/superpowers/specs/2026-06-05-talker-log-architecture-design.md` and confirm:

- `sync-server` is the Talker key.
- Scene labels stay in message prefixes.
- Existing `TalkerScreen(talker: talker)` remains unchanged.
- Only `sync_server.dart` is migrated in the first implementation pass.
- Sensitive request/body behavior is not expanded.

- [ ] **Step 3: Prepare handoff summary**

Use this summary shape:

```text
已完成 Talker 自定义日志第一阶段：
- 在 felorx_utilities 增加 FelorxTalkerLog / FelorxLogger / sync-server key 注册。
- 修正 createTalker(settings:) 传参。
- 将 sync_server.dart 迁移到 sync-server 模块日志，并按 scene 标注核心日志。
- 增加 felorx_utilities 与 felorx_sync 的聚焦测试。

验证：
- cd packages/core/felorx_utilities && dart test test/talker_test.dart
- cd packages/core/felorx_utilities && dart analyze lib/talker.dart test/talker_test.dart
- cd packages/sync/felorx_sync && dart test test/sync_server_logging_test.dart
- cd packages/sync/felorx_sync && dart test test/sync_server_test.dart
- cd packages/sync/felorx_sync && dart analyze lib/src/sync_server.dart test/sync_server_logging_test.dart
```
