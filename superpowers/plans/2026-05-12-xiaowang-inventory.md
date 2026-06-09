# 小汪记物 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the new `inventory` Flutter sub-application for personal physical and virtual asset inventory, backed by a Puupee/Sync `InventoryAsset` feature model.

**Architecture:** Add a first-class `InventoryAsset` content type in `packages/puupee`, then scaffold `apps/puupee/inventory` with the same `runMyApp`, GoRouter, Riverpod, ProductRepoBase, and shadcn Flutter conventions used by existing sub-apps. Image and screenshot capture produce editable `InventoryAssetDraft` values through a mock recognition service so the UI and Sync save path are complete before real OCR/AI exists.

**Tech Stack:** Dart 3.8, Flutter, shadcn_flutter, go_router with go_router_builder, Riverpod annotation, Puupee/Sync, drift query APIs via repo classes, build_runner.

---

## File Structure

- Modify `pubspec.yaml`: add `apps/puupee/inventory` to the workspace list near the other Puupee apps.
- Modify `packages/puupee_utilities/lib/products.dart`: register `Products.inventory` for app root creation.
- Modify `packages/puupee_utilities/test/products_test.dart`: cover `inventory` product lookup.
- Create `packages/puupee/lib/src/features/inventory_asset.dart`: declares enums, `@PuupeeFeature`, and convenience extensions for `InventoryAsset`.
- Modify `packages/puupee/lib/puupee.dart`: export `src/features/inventory_asset.dart`.
- Generate `packages/puupee/lib/src/features/inventory_asset.puupee.dart`, `packages/puupee/lib/src/generated/puupee_content_types.g.dart`, and `packages/puupee/lib/src/generated/feature_cli_schemas.g.dart`.
- Create `packages/puupee/test/features/inventory_asset_test.dart`: verifies contentType, key fields, and conversion helpers.
- Create `apps/puupee/inventory/pubspec.yaml`: workspace package definition.
- Create `apps/puupee/inventory/lib/env.dart`: `InventoryEnvConfig`.
- Create `apps/puupee/inventory/lib/main.dart`: app entry using `runMyApp`.
- Create `apps/puupee/inventory/lib/router.dart`: typed stateful shell routes and shared settings routes.
- Generate `apps/puupee/inventory/lib/router.g.dart`.
- Create `apps/puupee/inventory/lib/components/adaptive_inventory_shell.dart`: desktop sidebar and mobile bottom nav.
- Create `apps/puupee/inventory/lib/models/inventory_asset_draft.dart`: draft and field confidence model.
- Create `apps/puupee/inventory/lib/models/inventory_filter.dart`: list/search/reminder filter state.
- Create `apps/puupee/inventory/lib/services/inventory_recognition_service.dart`: mock physical/screenshot recognition.
- Create `apps/puupee/inventory/lib/repo/inventory_asset_repo.dart`: Sync-backed CRUD and stream queries.
- Create `apps/puupee/inventory/lib/providers/inventory_providers.dart`: repo, stream, filtering, dashboard, reminder providers.
- Generate `apps/puupee/inventory/lib/providers/inventory_providers.g.dart`.
- Create page files under `apps/puupee/inventory/lib/pages/`: dashboard, asset list, detail/edit, recognition confirm, reminders, checklist, settings shell handling through router.
- Create focused widgets under `apps/puupee/inventory/lib/components/`: asset card, stat card, quick add panel, filter bar, draft field editor.
- Create tests under `apps/puupee/inventory/test/`: draft/filter/recognition unit tests and at least one widget smoke test for the dashboard.
- Create `apps/puupee/inventory/README.md`: usage, first-version scope, and verification commands.

---

### Task 1: Add `InventoryAsset` Feature Model

**Files:**
- Create: `packages/puupee/test/features/inventory_asset_test.dart`
- Create: `packages/puupee/lib/src/features/inventory_asset.dart`
- Modify: `packages/puupee/lib/puupee.dart`
- Generated: `packages/puupee/lib/src/features/inventory_asset.puupee.dart`
- Generated: `packages/puupee/lib/src/generated/puupee_content_types.g.dart`
- Generated: `packages/puupee/lib/src/generated/feature_cli_schemas.g.dart`

- [ ] **Step 1: Write the failing model test**

Create `packages/puupee/test/features/inventory_asset_test.dart`:

```dart
import 'package:decimal/decimal.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:puupee/puupee.dart';

void main() {
  test('InventoryAsset exposes stable content type and slot-backed fields', () {
    final asset = InventoryAsset.create(
      id: 'asset-1',
      creatorId: 'user-1',
      name: 'iPhone 15 Pro',
      assetKind: InventoryAssetKind.physical,
      category: '手机',
      status: InventoryAssetStatus.inUse,
      source: InventoryAssetSource.camera,
    )
      ..brand = 'Apple'
      ..model = 'A3108'
      ..serialNumber = 'SN-001'
      ..location = '卧室'
      ..purchasePrice = Decimal.parse('7999')
      ..currentValue = Decimal.parse('6200')
      ..currency = 'CNY'
      ..warrantyEndDate = DateTime(2027, 5, 12)
      ..reminderEnabled = true
      ..importance = 4;

    expect(asset.contentType, kInventoryAssetContentType);
    expect(asset.contentType, 'application/vnd.puupee.inventory.asset');
    expect(asset.assetKind, InventoryAssetKind.physical);
    expect(asset.status, InventoryAssetStatus.inUse);
    expect(asset.source, InventoryAssetSource.camera);
    expect(asset.brand, 'Apple');
    expect(asset.location, '卧室');
    expect(asset.purchasePrice, Decimal.parse('7999'));
    expect(asset.currentValue, Decimal.parse('6200'));
    expect(asset.warrantyEndDate, DateTime(2027, 5, 12));
    expect(asset.reminderEnabled, isTrue);
    expect(asset.importance, 4);
  });
}
```

- [ ] **Step 2: Run the test to verify it fails before implementation**

Run:

```bash
cd packages/puupee
flutter test test/features/inventory_asset_test.dart
```

Expected: FAIL because `InventoryAsset`, `InventoryAssetKind`, and `kInventoryAssetContentType` do not exist.

- [ ] **Step 3: Add the feature model source**

Create `packages/puupee/lib/src/features/inventory_asset.dart`:

```dart
import 'package:decimal/decimal.dart';
import 'package:drift_postgres/drift_postgres.dart';
import 'package:puupee_annotations/puupee_annotations.dart';

import '../database.dart';
import '../generated/puupee_content_types.g.dart';

part 'inventory_asset.puupee.dart';

enum InventoryAssetKind {
  physical,
  virtual,
  document,
  license,
  service,
  entitlement,
}

enum InventoryAssetStatus {
  inUse,
  idle,
  stored,
  missing,
  disposed,
  expired,
}

enum InventoryAssetSource {
  manual,
  camera,
  screenshot,
  import,
}

enum InventoryBillingCycle {
  none,
  daily,
  weekly,
  monthly,
  yearly,
  custom,
}

@PuupeeFeature(
  contentTypeOverride: 'application/vnd.puupee.inventory.asset',
)
// ignore: unused_element
abstract class _InventoryAsset {
  InventoryAssetKind? get assetKind;
  InventoryAssetStatus? get status;
  InventoryAssetSource? get source;
  String? get category;
  String? get brand;
  String? get model;
  String? get serialNumber;
  String? get location;
  String? get accountHint;
  String? get planName;
  String? get currency;

  @PuupeeField(slotType: SlotType.real)
  Decimal? get purchasePrice;

  @PuupeeField(slotType: SlotType.real)
  Decimal? get currentValue;

  @PuupeeField(slotType: SlotType.datetime)
  DateTime? get purchaseDate;

  @PuupeeField(slotType: SlotType.datetime)
  DateTime? get warrantyEndDate;

  @PuupeeField(slotType: SlotType.datetime)
  DateTime? get renewalDate;

  InventoryBillingCycle? get billingCycle;
  int? get importance;

  @PuupeeDefault(false)
  bool get reminderEnabled;

  @PuupeeTransient()
  List<String>? get tags;
}

extension InventoryAssetPuupeeX on Puupee {
  InventoryAsset toInventoryAssetWithTags({List<String>? tags}) {
    final asset = toInventoryAsset(tags: tags);
    asset.tags = tags;
    return asset;
  }
}

extension InventoryAssetX on InventoryAsset {
  bool get isVirtualLike =>
      assetKind == InventoryAssetKind.virtual ||
      assetKind == InventoryAssetKind.license ||
      assetKind == InventoryAssetKind.service ||
      assetKind == InventoryAssetKind.entitlement;

  bool get hasUpcomingReminder =>
      reminderEnabled && (renewalDate != null || warrantyEndDate != null);

  DateTime? get nextImportantDate {
    final dates = <DateTime>[
      if (renewalDate != null) renewalDate!,
      if (warrantyEndDate != null) warrantyEndDate!,
    ]..sort();
    return dates.isEmpty ? null : dates.first;
  }
}
```

- [ ] **Step 4: Export the feature model**

Modify `packages/puupee/lib/puupee.dart` and add this export near the other feature exports:

```dart
export 'src/features/inventory_asset.dart';
```

- [ ] **Step 5: Run generator for `packages/puupee`**

Run:

```bash
cd packages/puupee
dart run build_runner build --delete-conflicting-outputs
```

Expected: generated files include `inventory_asset.puupee.dart`, `kInventoryAssetContentType`, and the CLI schema entry for `application/vnd.puupee.inventory.asset`.

- [ ] **Step 6: Run the feature test**

Run:

```bash
cd packages/puupee
flutter test test/features/inventory_asset_test.dart
```

Expected: PASS.

- [ ] **Step 7: Commit the model work**

```bash
git add packages/puupee/lib/puupee.dart \
  packages/puupee/lib/src/features/inventory_asset.dart \
  packages/puupee/lib/src/features/inventory_asset.puupee.dart \
  packages/puupee/lib/src/generated/puupee_content_types.g.dart \
  packages/puupee/lib/src/generated/feature_cli_schemas.g.dart \
  packages/puupee/test/features/inventory_asset_test.dart
git commit -m "feat(inventory): 添加资产盘点模型"
```

---

### Task 2: Scaffold the Inventory App Shell

**Files:**
- Modify: `pubspec.yaml`
- Create: `apps/puupee/inventory/pubspec.yaml`
- Create: `apps/puupee/inventory/lib/env.dart`
- Create: `apps/puupee/inventory/lib/main.dart`
- Create: `apps/puupee/inventory/lib/router.dart`
- Create: `apps/puupee/inventory/lib/components/adaptive_inventory_shell.dart`
- Create: temporary page placeholders under `apps/puupee/inventory/lib/pages/`
- Generated: `apps/puupee/inventory/lib/router.g.dart`

- [ ] **Step 1: Add the app to the workspace**

In root `pubspec.yaml`, add the workspace entry near the other `apps/puupee/*` entries:

```yaml
  - apps/puupee/inventory
```

- [ ] **Step 2: Create the app pubspec**

Create `apps/puupee/inventory/pubspec.yaml`:

```yaml
name: puupee_inventory
resolution: workspace
version: 0.1.0
publish_to: none
description: 小汪记物 - 个人实物资产与虚拟资产盘点工具

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
  decimal: ^3.0.2
  drift: ^2.23.1
  drift_postgres: ^1.3.0

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

- [ ] **Step 3: Create env and main**

Create `apps/puupee/inventory/lib/env.dart`:

```dart
import 'package:puupee_shared/env.dart';

class InventoryEnvConfig extends EnvConfig {
  InventoryEnvConfig()
    : super(
        env: 'production',
        appId: 'inventory',
        appTitle: '小汪记物',
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
      );
}
```

Create `apps/puupee/inventory/lib/main.dart`:

```dart
import 'package:go_router/go_router.dart';
import 'package:puupee_inventory/env.dart';
import 'package:puupee_inventory/router.dart';
import 'package:puupee_shared/app/startup.dart';

void main() async {
  await runMyApp(
    env: InventoryEnvConfig(),
    createRouter: (ref) => GoRouter(
      initialLocation: '/inventory',
      routes: $appRoutes,
    ),
  );
}
```

- [ ] **Step 4: Add placeholder pages for route compilation**

Create `apps/puupee/inventory/lib/pages/dashboard_page.dart`:

```dart
import 'package:shadcn_flutter/shadcn_flutter.dart';

class InventoryDashboardPage extends StatelessWidget {
  const InventoryDashboardPage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Center(child: Text('小汪记物'));
  }
}
```

Create `asset_list_page.dart`, `reminders_page.dart`, and `checklist_page.dart` in the same folder with classes `InventoryAssetListPage`, `InventoryRemindersPage`, and `InventoryChecklistPage`; each returns `const Center(child: Text('<页面名>'))`.

- [ ] **Step 5: Create shell and router**

Create `apps/puupee/inventory/lib/components/adaptive_inventory_shell.dart` using `AdaptiveLayout`, `HomeIcon(appId: 'inventory')`, shadcn `Scaffold`, and five menu items:

```dart
import 'package:go_router/go_router.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:puupee_shared/components/button/home_icon_button.dart';
import 'package:puupee_ui/components/adaptive_layout.dart';
import 'package:shadcn_flutter/shadcn_flutter.dart';

class AdaptiveInventoryShell extends ConsumerWidget {
  const AdaptiveInventoryShell({super.key, required this.navigationShell});

  final StatefulNavigationShell navigationShell;

  static const _items = [
    AdaptiveMenuItem(label: '总览', icon: Icons.dashboard_outlined),
    AdaptiveMenuItem(label: '资产', icon: Icons.inventory_2_outlined),
    AdaptiveMenuItem(label: '提醒', icon: Icons.notifications_outlined),
    AdaptiveMenuItem(label: '盘点', icon: Icons.fact_check_outlined),
    AdaptiveMenuItem(
      label: '我的',
      icon: Icons.person_outline,
      position: MenuItemPosition.bottom,
    ),
  ];

  int _currentIndex(BuildContext context) {
    final path = GoRouterState.of(context).uri.path;
    if (path.startsWith('/inventory/assets')) return 1;
    if (path.startsWith('/inventory/reminders')) return 2;
    if (path.startsWith('/inventory/checklist')) return 3;
    if (path.startsWith('/inventory/settings')) return 4;
    return 0;
  }

  void _go(BuildContext context, int index) {
    switch (index) {
      case 1:
        context.go('/inventory/assets');
        break;
      case 2:
        context.go('/inventory/reminders');
        break;
      case 3:
        context.go('/inventory/checklist');
        break;
      case 4:
        context.go('/inventory/settings');
        break;
      default:
        context.go('/inventory');
    }
  }

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isDesktop = MediaQuery.of(context).size.width >= 700;
    final current = _currentIndex(context);
    if (isDesktop) {
      return AdaptiveLayout(
        appIcon: const HomeIcon(appId: 'inventory'),
        appTitle: '小汪记物',
        menuItems: _items,
        selectedIndex: current,
        onMenuItemTap: (index) => _go(context, index),
        child: navigationShell,
      );
    }

    final theme = Theme.of(context);
    return Scaffold(
      footers: [
        Container(
          decoration: BoxDecoration(
            color: theme.colorScheme.background,
            border: Border(
              top: BorderSide(color: theme.colorScheme.border, width: 1),
            ),
          ),
          child: SafeArea(
            top: false,
            child: Row(
              children: [
                for (var i = 0; i < _items.length; i++)
                  Expanded(
                    child: GestureDetector(
                      behavior: HitTestBehavior.opaque,
                      onTap: () => _go(context, i),
                      child: SizedBox(
                        height: 56,
                        child: Icon(
                          _items[i].icon,
                          size: 22,
                          color: i == current
                              ? theme.colorScheme.primary
                              : theme.colorScheme.mutedForeground,
                        ),
                      ),
                    ),
                  ),
              ],
            ),
          ),
        ),
      ],
      child: navigationShell,
    );
  }
}
```

Create `apps/puupee/inventory/lib/router.dart` by following the `ops` router pattern with branches for `/inventory`, `/inventory/assets`, `/inventory/reminders`, `/inventory/checklist`, and shared settings routes under `/inventory/settings`.

- [ ] **Step 6: Generate routes and verify app analysis**

Run:

```bash
cd apps/puupee/inventory
dart run build_runner build --delete-conflicting-outputs
dart analyze
```

Expected: `router.g.dart` is generated and `dart analyze` exits without errors for the new app.

- [ ] **Step 7: Commit the shell work**

```bash
git add pubspec.yaml apps/puupee/inventory
git commit -m "feat(inventory): 创建小汪记物应用骨架"
```

---

### Task 3: Add Draft Models, Recognition Service, and Unit Tests

**Files:**
- Create: `apps/puupee/inventory/lib/models/inventory_asset_draft.dart`
- Create: `apps/puupee/inventory/lib/models/inventory_filter.dart`
- Create: `apps/puupee/inventory/lib/services/inventory_recognition_service.dart`
- Create: `apps/puupee/inventory/test/inventory_recognition_service_test.dart`
- Create: `apps/puupee/inventory/test/inventory_filter_test.dart`

- [ ] **Step 1: Write recognition tests**

Create `apps/puupee/inventory/test/inventory_recognition_service_test.dart`:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:puupee/puupee.dart';
import 'package:puupee_inventory/services/inventory_recognition_service.dart';

void main() {
  test('mock physical recognition returns multiple editable drafts', () async {
    final service = MockInventoryRecognitionService();
    final drafts = await service.recognizePhysicalPhoto();

    expect(drafts, hasLength(3));
    expect(drafts.first.name, 'iPhone 15 Pro');
    expect(drafts.first.assetKind, InventoryAssetKind.physical);
    expect(drafts.first.source, InventoryAssetSource.camera);
    expect(drafts.first.fieldConfidence['name'], 0.96);
  });

  test('mock virtual recognition returns subscription draft with reminder', () async {
    final service = MockInventoryRecognitionService();
    final drafts = await service.recognizeVirtualScreenshot();

    expect(drafts, hasLength(1));
    expect(drafts.single.name, 'ChatGPT Plus');
    expect(drafts.single.assetKind, InventoryAssetKind.virtual);
    expect(drafts.single.source, InventoryAssetSource.screenshot);
    expect(drafts.single.reminderEnabled, isTrue);
    expect(drafts.single.billingCycle, InventoryBillingCycle.monthly);
  });
}
```

- [ ] **Step 2: Write filter tests**

Create `apps/puupee/inventory/test/inventory_filter_test.dart` with tests for search query, kind, status, and reminder windows using `InventoryAsset.create(...)`.

- [ ] **Step 3: Run tests and confirm failure**

Run:

```bash
cd apps/puupee/inventory
flutter test test/inventory_recognition_service_test.dart test/inventory_filter_test.dart
```

Expected: FAIL because the model and service files do not exist.

- [ ] **Step 4: Implement draft model**

Create `inventory_asset_draft.dart` with immutable `InventoryAssetDraft`, `copyWith`, `toAsset`, and `fromAsset`. Include `Map<String, double> fieldConfidence` and `Set<String> missingFields`.

- [ ] **Step 5: Implement filter model**

Create `inventory_filter.dart` with `InventoryFilter` and `InventoryFilterX.applyTo(List<InventoryAsset>)`. Search lowercases `name`, `description`, `category`, `brand`, `model`, `location`, `accountHint`, and tag strings.

- [ ] **Step 6: Implement mock recognition service**

Create `inventory_recognition_service.dart` with:

```dart
abstract class InventoryRecognitionService {
  Future<List<InventoryAssetDraft>> recognizePhysicalPhoto();
  Future<List<InventoryAssetDraft>> recognizeVirtualScreenshot();
}
```

`MockInventoryRecognitionService` returns stable drafts for iPhone, charger, USB-C cable, and ChatGPT Plus.

- [ ] **Step 7: Run tests**

Run:

```bash
cd apps/puupee/inventory
flutter test test/inventory_recognition_service_test.dart test/inventory_filter_test.dart
```

Expected: PASS.

- [ ] **Step 8: Commit the domain service work**

```bash
git add apps/puupee/inventory/lib/models \
  apps/puupee/inventory/lib/services \
  apps/puupee/inventory/test/inventory_recognition_service_test.dart \
  apps/puupee/inventory/test/inventory_filter_test.dart
git commit -m "feat(inventory): 添加资产草稿和识别服务"
```

---

### Task 4: Add Sync Repo and Riverpod Providers

**Files:**
- Modify: `packages/puupee_utilities/lib/products.dart`
- Modify: `packages/puupee_utilities/test/products_test.dart`
- Create: `apps/puupee/inventory/lib/repo/inventory_asset_repo.dart`
- Create: `apps/puupee/inventory/lib/providers/inventory_providers.dart`
- Generated: `apps/puupee/inventory/lib/providers/inventory_providers.g.dart`
- Create: `apps/puupee/inventory/test/inventory_provider_logic_test.dart`

- [ ] **Step 1: Register the inventory product**

Modify `packages/puupee_utilities/lib/products.dart`:

```dart
static const Products inventory = Products._("inventory", "Inventory", "盘点");
```

Add `inventory` to `Products.values` after `billings`.

Modify `packages/puupee_utilities/test/products_test.dart`:

```dart
expect(Products.inventory.appId, equals('inventory'));
expect(Products.inventory.value, equals('Inventory'));
expect(Products.inventory.title, equals('盘点'));
expect(Products.values, contains(Products.inventory));
expect(Products.fromAppId('inventory'), equals(Products.inventory));
expect(Products.fromValue('Inventory'), equals(Products.inventory));
```

Run:

```bash
cd packages/puupee_utilities
dart test test/products_test.dart
```

Expected: PASS.

- [ ] **Step 2: Write provider logic tests**

Create pure tests for `InventoryDashboardSummary.fromAssets` and `InventoryReminderBucket.fromAssets`, using fixed dates so the output is stable.

- [ ] **Step 3: Implement `InventoryAssetRepo`**

Create repo extending `ProductRepoBase`; set `product => Products.inventory`. Implement:

```dart
Stream<List<InventoryAsset>> queryAssetStream();
Future<List<InventoryAsset>> getAllAssets();
Future<InventoryAsset?> getAssetById(String id);
Future<InventoryAsset> createAsset(InventoryAssetDraft draft);
Future<InventoryAsset> updateAsset(InventoryAsset asset);
Future<void> deleteAsset(InventoryAsset asset);
```

Queries filter `contentType == PuupeeContentTypes.inventoryAsset`, current `creatorId`, and `deletedAt.isNull()`. Use `queryStreamWithTags` and `queryWithTags`.

- [ ] **Step 4: Implement providers**

Create `getInventoryAssetRepoProvider`, `inventoryAssetStreamProvider`, `inventoryFilteredAssetsProvider`, `inventoryDashboardSummaryProvider`, `inventoryReminderAssetsProvider`, and `inventoryRecognitionServiceProvider`.

- [ ] **Step 5: Generate provider code**

Run:

```bash
cd apps/puupee/inventory
dart run build_runner build --delete-conflicting-outputs
```

Expected: `inventory_providers.g.dart` is generated.

- [ ] **Step 6: Run provider tests and analyze**

Run:

```bash
cd apps/puupee/inventory
flutter test test/inventory_provider_logic_test.dart
dart analyze
cd ../../../packages/puupee_utilities
dart test test/products_test.dart
```

Expected: PASS and no analyzer errors in the new app.

- [ ] **Step 7: Commit repo and provider work**

```bash
git add packages/puupee_utilities/lib/products.dart \
  packages/puupee_utilities/test/products_test.dart \
  apps/puupee/inventory/lib/repo \
  apps/puupee/inventory/lib/providers \
  apps/puupee/inventory/test/inventory_provider_logic_test.dart
git commit -m "feat(inventory): 接入资产同步数据层"
```

---

### Task 5: Build Core UI Pages

**Files:**
- Create/modify components under `apps/puupee/inventory/lib/components/`
- Replace placeholders in `apps/puupee/inventory/lib/pages/dashboard_page.dart`
- Replace placeholders in `asset_list_page.dart`, `reminders_page.dart`, `checklist_page.dart`
- Create: `apps/puupee/inventory/lib/pages/asset_detail_page.dart`
- Create: `apps/puupee/inventory/lib/pages/recognition_confirm_page.dart`
- Modify: `apps/puupee/inventory/lib/router.dart`
- Generated: `apps/puupee/inventory/lib/router.g.dart`
- Create: `apps/puupee/inventory/test/dashboard_page_test.dart`

- [ ] **Step 1: Write dashboard smoke test**

Create a widget test that pumps `InventoryDashboardPage` inside `ProviderScope` and verifies text for `小汪记物`, `拍照实物`, `导入截图`, and `手动添加`.

- [ ] **Step 2: Implement shared components**

Create:

```text
inventory_stat_card.dart
inventory_asset_card.dart
inventory_quick_add_panel.dart
inventory_filter_bar.dart
inventory_draft_field_editor.dart
```

Use only `package:shadcn_flutter/shadcn_flutter.dart` plus project shared components. Do not import `package:flutter/material.dart`.

- [ ] **Step 3: Implement dashboard page**

Dashboard shows four stats, quick add panel, soon reminders list, and latest assets list. Buttons route to `/inventory/capture/physical`, `/inventory/capture/virtual`, and `/inventory/assets/new`.

- [ ] **Step 4: Implement asset list page**

List page watches filtered assets, renders filter bar, and renders `InventoryAssetCard` for each item. Empty state text is `还没有资产`.

- [ ] **Step 5: Implement detail/edit page**

Detail page accepts optional asset id. New mode starts from an empty `InventoryAssetDraft`; edit mode loads an existing asset. Save calls repo create or update and shows `showMessage('资产已保存')`.

- [ ] **Step 6: Implement recognition confirmation page**

Page accepts mode `physical` or `virtual`, calls the mock recognition service, renders all drafts, and saves selected drafts through repo. Save success message is `资产已添加`.

- [ ] **Step 7: Implement reminders and checklist pages**

Reminders page groups upcoming renewal, expiry, and warranty dates. Checklist page groups assets by location and provides quick status updates for confirmed, needs attention, and missing.

- [ ] **Step 8: Update router and regenerate**

Add routes:

```text
/inventory/assets/new
/inventory/assets/:id
/inventory/capture/physical
/inventory/capture/virtual
```

Run:

```bash
cd apps/puupee/inventory
dart run build_runner build --delete-conflicting-outputs
flutter test test/dashboard_page_test.dart
dart analyze
```

Expected: PASS and no analyzer errors in the new app.

- [ ] **Step 9: Commit UI work**

```bash
git add apps/puupee/inventory/lib apps/puupee/inventory/test/dashboard_page_test.dart
git commit -m "feat(inventory): 完成资产盘点核心界面"
```

---

### Task 6: Documentation and Final Verification

**Files:**
- Create: `apps/puupee/inventory/README.md`
- Modify: generated files only when verification commands produce intended app-specific changes

- [ ] **Step 1: Create README**

Create `apps/puupee/inventory/README.md` with sections:

```markdown
# 小汪记物

个人实物资产与虚拟资产盘点工具。

## 第一版范围

- 统一管理实物和虚拟资产。
- 手动添加、拍照实物、截图虚拟资产三个入口。
- 拍照/截图使用模拟识别服务生成可编辑草稿。
- 资产数据保存到 Puupee/Sync。
- 提供总览、资产列表、提醒和轻量盘点。

## 开发

```bash
cd apps/puupee/inventory
dart run build_runner build --delete-conflicting-outputs
dart analyze
flutter test
```
```

- [ ] **Step 2: Run package tests**

Run:

```bash
cd packages/puupee
flutter test test/features/inventory_asset_test.dart
```

Expected: PASS.

- [ ] **Step 3: Run app tests**

Run:

```bash
cd apps/puupee/inventory
flutter test
dart analyze
```

Expected: PASS and no analyzer errors in the inventory app.

- [ ] **Step 4: Check git diff scope**

Run:

```bash
git status --short
git diff --stat
```

Expected: changed files are limited to `packages/puupee`, `apps/puupee/inventory`, root `pubspec.yaml`, and generated files caused by the inventory model/app. Existing unrelated `GeneratedPluginRegistrant` and `untranslated_messages.json` changes remain unstaged.

- [ ] **Step 5: Commit final docs**

```bash
git add apps/puupee/inventory/README.md
git commit -m "docs(inventory): 添加小汪记物说明"
```

---

## Self-Review

- Spec coverage: The plan creates the `inventory` app, adds `InventoryAsset`, supports physical and virtual assets, uses mock recognition, includes Sync repo/providers, implements dashboard/list/detail/confirm/reminder/checklist pages, and adds tests and docs.
- Placeholder scan: The plan has no incomplete sections or unnamed files.
- Type consistency: `InventoryAssetKind`, `InventoryAssetStatus`, `InventoryAssetSource`, `InventoryBillingCycle`, `InventoryAssetDraft`, `InventoryRecognitionService`, `InventoryAssetRepo`, and provider names are consistent across tasks.
