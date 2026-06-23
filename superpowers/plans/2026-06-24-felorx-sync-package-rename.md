# Felorx Sync Package Rename Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rename the sync package identities from `puupee_sync` / `puupee_sync_api` to `felorx_sync` / `felorx_sync_api`.

**Architecture:** This is the next Felorx migration slice after the Todo app identity rename. It changes Dart package names, directory paths, package imports, dependency keys, generated-client package library exports, and current runtime/documentation references that point to the sync packages. It deliberately does not rename API schema/model classes such as `Puupee`, `SyncApiPuupee`, request DTOs, content types, or service classes such as `PuupeeSyncServer`; those belong to the later model/class migration slice.

**Tech Stack:** Dart workspace, Melos/Dart pub workspaces, generated OpenAPI Dart client, Flutter apps and shared packages consuming sync packages.

---

### Task 1: Move Sync Package Directories And Workspace Entries

**Files:**
- Move: `packages/sync/puupee_sync` -> `packages/sync/felorx_sync`
- Move: `packages/api/puupee_sync_api` -> `packages/api/felorx_sync_api`
- Modify: `pubspec.yaml`
- Modify: `AGENTS_packages.md`
- Modify: `docs/packages.mdx`

- [ ] **Step 1: Move package directories**

Run:
```bash
git mv packages/sync/puupee_sync packages/sync/felorx_sync
git mv packages/api/puupee_sync_api packages/api/felorx_sync_api
```

Expected: `git status --short` shows both package directories as renames.

- [ ] **Step 2: Update workspace paths**

Apply these mappings:
```text
packages/sync/puupee_sync -> packages/sync/felorx_sync
packages/api/puupee_sync_api -> packages/api/felorx_sync_api
```

Update current package index docs (`AGENTS_packages.md`, `docs/packages.mdx`) with the new package paths and names. Do not rewrite historical release notes, old design specs, or migration summaries in this slice.

- [ ] **Step 3: Validate workspace discovery**

Run:
```bash
dart pub workspace list | rg 'packages/sync/felorx_sync|packages/api/felorx_sync_api|felorx_sync'
```

Expected: output includes both `felorx_sync` and `felorx_sync_api` package entries.

### Task 2: Rename Package Metadata And Library Entry Points

**Files:**
- Modify: `packages/sync/felorx_sync/pubspec.yaml`
- Move: `packages/sync/felorx_sync/lib/puupee_sync.dart` -> `packages/sync/felorx_sync/lib/felorx_sync.dart`
- Modify: `packages/api/felorx_sync_api/pubspec.yaml`
- Move: `packages/api/felorx_sync_api/lib/puupee_sync_api.dart` -> `packages/api/felorx_sync_api/lib/felorx_sync_api.dart`
- Modify: `packages/api/felorx_sync_api/.openapi-generator/FILES`
- Modify: `packages/sync/felorx_sync/sync-api-config.json`
- Modify: `packages/sync/felorx_sync/scripts/generate_api_client.dart`

- [ ] **Step 1: Update package names**

Set:
```yaml
# packages/sync/felorx_sync/pubspec.yaml
name: felorx_sync
dependencies:
  felorx_sync_api: ^1.10.1
```

Set:
```yaml
# packages/api/felorx_sync_api/pubspec.yaml
name: felorx_sync_api
description: Api for Felorx sync service.
homepage: https://github.com/felorx/felorx-sync
```

- [ ] **Step 2: Rename library barrels**

Run:
```bash
git mv packages/sync/felorx_sync/lib/puupee_sync.dart packages/sync/felorx_sync/lib/felorx_sync.dart
git mv packages/api/felorx_sync_api/lib/puupee_sync_api.dart packages/api/felorx_sync_api/lib/felorx_sync_api.dart
```

In the renamed API barrel, replace exports:
```text
package:puupee_sync_api/ -> package:felorx_sync_api/
```

- [ ] **Step 3: Update generator metadata**

Apply these mappings:
```text
"pubLibrary": "puupee_sync_api" -> "pubLibrary": "felorx_sync_api"
"pubName": "puupee_sync_api" -> "pubName": "felorx_sync_api"
"pubDescription": "Api for puupee sync service." -> "pubDescription": "Api for Felorx sync service."
"pubHomepage": "https://github.com/puupee/puupee-sync" -> "pubHomepage": "https://github.com/felorx/felorx-sync"
packages/puupee_sync_api -> packages/felorx_sync_api
--git-user-id puupee -> --git-user-id felorx
--git-repo-id puupee-sync-api -> --git-repo-id felorx-sync-api
lib/puupee_sync_api.dart -> lib/felorx_sync_api.dart
```

Do not rename generated model files such as `sync_api_puupee.dart` in this slice.

### Task 3: Rewrite Current Package Consumers

**Files:**
- Modify: current app/package/demo `pubspec.yaml` files that depend on `puupee_sync` or `puupee_sync_api`
- Modify: Dart imports in `apps/`, `demos/`, `packages/common`, `packages/tools`, `packages/sync/felorx_sync`, and `packages/api/felorx_sync_api`
- Modify: current runtime-facing strings that say to deploy `puupee_sync`

- [ ] **Step 1: Rewrite dependency keys**

Apply these exact key mappings in `pubspec.yaml` files:
```text
puupee_sync_api: -> felorx_sync_api:
puupee_sync: -> felorx_sync:
```

Do not rename `puupee_sync_node` in this slice.

- [ ] **Step 2: Rewrite package imports**

Apply:
```text
package:puupee_sync_api/ -> package:felorx_sync_api/
package:puupee_sync/ -> package:felorx_sync/
```

Do not replace `package:puupee_sync_node/`.

- [ ] **Step 3: Rewrite active runtime wording**

Replace active user-facing/runtime references that name the package as infrastructure:
```text
puupee_sync -> felorx_sync
puupee_sync_api -> felorx_sync_api
```

Keep historical changelog entries, old superpowers plans/specs, and generated API model names unchanged.

### Task 4: Validate And Commit

**Files:**
- All files changed by Tasks 1-3.

- [ ] **Step 1: Format and analyze renamed packages**

Run:
```bash
dart format packages/sync/felorx_sync/lib packages/sync/felorx_sync/test packages/api/felorx_sync_api/lib packages/api/felorx_sync_api/test
dart analyze packages/sync/felorx_sync packages/api/felorx_sync_api
```

Expected: format exits 0 and analyzer has no new errors.

- [ ] **Step 2: Run focused tests**

Run:
```bash
cd packages/api/felorx_sync_api && dart test test/default_api_test.dart test/health_check200_response_test.dart test/get_node_info200_response_test.dart
cd packages/sync/felorx_sync && dart test test/core_models_test.dart test/pairing_auth_test.dart test/sync_server_route_naming_test.dart test/lan_discovery_service_entry_test.dart
```

Expected: targeted tests pass.

- [ ] **Step 3: Validate representative consumers**

Run:
```bash
dart analyze apps/todo/lib/provider.dart apps/drop/lib/provider.dart apps/sync_node/lib/sync_node_runner.dart packages/common/puupee_shared/lib/providers/sync/syncer.dart packages/tools/puupee_cli/lib/src/commands/discover/discover_command.dart packages/tools/puupee_builder_cli/lib/src/build_command.dart
```

Expected: analyzer has no import/package resolution errors for `felorx_sync` or `felorx_sync_api`.

- [ ] **Step 4: Run residual scans**

Run:
```bash
rg -n 'package:puupee_sync(_api)?/|^\s*puupee_sync(_api)?:|packages/(sync/puupee_sync|api/puupee_sync_api)\b' apps demos packages pubspec.yaml AGENTS_packages.md docs/packages.mdx --hidden --glob '!**/.git/**' --glob '!**/build/**' --glob '!**/.dart_tool/**' --glob '!**/node_modules/**' || true
rg -n 'package:felorx_sync(_api)?/|^\s*felorx_sync(_api)?:|packages/(sync/felorx_sync|api/felorx_sync_api)\b' apps demos packages pubspec.yaml AGENTS_packages.md docs/packages.mdx --hidden --glob '!**/.git/**' --glob '!**/build/**' --glob '!**/.dart_tool/**' --glob '!**/node_modules/**'
git diff --check
```

Expected: first scan has no current code/config/package-path hits except intentionally retained historical docs if the scan scope is expanded; second scan shows the new package identities; diff check exits 0.

- [ ] **Step 5: Commit**

Run:
```bash
git add -A packages/sync packages/api/felorx_sync_api packages/api/puupee_sync_api apps demos packages/common packages/tools pubspec.yaml AGENTS_packages.md docs
git commit -m "refactor(felorx_sync): 迁移同步包命名"
```

Expected: commit succeeds and `git status --short` is clean.
