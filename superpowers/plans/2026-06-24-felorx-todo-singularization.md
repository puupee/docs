# Felorx Todo Singularization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rename the Todos sub-application identity from `todos` / `Puupee Todos` / `puupee_todos` / `com.puupee.todos` to `todo` / `Felorx Todo` / `felorx_todo` / `com.felorx.todo`.

**Architecture:** This is the first Phase 2 slice. It only changes the Todo app identity, workspace paths, package imports, bundle IDs, website seed data, and build/install scripts that explicitly reference the Todos app. It deliberately keeps the `Todo` feature model and `application/vnd.puupee.todo` content type unchanged until the later model/class-name migration slice.

**Tech Stack:** Flutter/Dart workspace, iOS/macOS Xcode configs, Android/OHOS bundle metadata, Windows/Linux CMake/installer files, Next.js website seed data, repository build scripts.

---

### Task 1: Move App Directory And Workspace Entries

**Files:**
- Move: `apps/todos` -> `apps/todo`
- Modify: `pubspec.yaml`
- Modify: `.puupee/builder.yaml`
- Modify: `.vscode/launch.json`

- [ ] **Step 1: Move the directory**

Run:
```bash
git mv apps/todos apps/todo
```

Expected: `git status --short` shows a rename from `apps/todos` to `apps/todo`.

- [ ] **Step 2: Update workspace and builder references**

Apply these exact mapping rules:
```text
apps/todos -> apps/todo
todos: -> todo:
cwd": "apps/todos" -> cwd": "apps/todo"
Todos Dev -> Todo Dev
Todos Prod -> Todo Prod
Todos -> Todo
```

Do not replace business variables such as `todosShowCompletedSettingProvider`, `List<Todo> todos`, or prose that describes a collection of todo items.

- [ ] **Step 3: Validate JSON and workspace lookup**

Run:
```bash
python3 -m json.tool .vscode/launch.json >/tmp/felorx-todo-launch-json-check.txt
dart pub workspace list | rg 'apps/todo|felorx_todo'
```

Expected: JSON parse succeeds and workspace output contains `apps/todo`.

### Task 2: Rename Dart Package And Imports

**Files:**
- Modify: `apps/todo/pubspec.yaml`
- Modify: all Dart imports under `apps/todo/lib` and `apps/todo/test`
- Modify: `apps/todo/README.md`

- [ ] **Step 1: Change package metadata**

Set:
```yaml
name: felorx_todo
tobias:
  url_scheme: felorxtodo
description: Felorx Todo - 待办事项管理应用
```

- [ ] **Step 2: Rewrite package imports**

Apply:
```text
package:puupee_todos/ -> package:felorx_todo/
```

Do not rename Dart symbols named `Todo`, `todo`, or `todos` unless they are package/import identity.

- [ ] **Step 3: Validate package tests**

Run:
```bash
cd apps/todo
dart format --output=none --set-exit-if-changed lib test
dart analyze
flutter test test/provider_dependencies_test.dart test/calendar/todo_event_converter_test.dart test/components/todo_card_test.dart test/pages/todo_detail_title_test.dart
```

Expected: format exits 0, analyze has no new errors, and targeted tests pass. If existing unrelated info-level lints remain, record them before committing.

### Task 3: Update Platform Bundle IDs And Display Names

**Files:**
- Modify: `apps/todo/flavorizr.yaml`
- Modify: `apps/todo/android/**`
- Modify: `apps/todo/ios/**`
- Modify: `apps/todo/macos/**`
- Modify: `apps/todo/ohos/**`
- Modify: `apps/todo/windows/**`
- Modify: `apps/todo/linux/**`
- Modify: `apps/todo/web/**`

- [ ] **Step 1: Apply identity mappings**

Apply these exact mappings in platform/config files:
```text
com.puupee.todos -> com.felorx.todo
com.puupee.todos.dev -> com.felorx.todo.dev
com.puupee.todos.RunnerTests -> com.felorx.todo.RunnerTests
Puupee Todos Dev -> Felorx Todo Dev
Puupee Todos -> Felorx Todo
puupee_todos_dev -> felorx_todo_dev
puupee_todos -> felorx_todo
puupeetodos -> felorxtodo
todos_setup -> todo_setup
```

- [ ] **Step 2: Preserve user-facing Chinese names**

Keep existing Chinese display names such as `小汪待办` unless a file also contains the English product name being migrated.

- [ ] **Step 3: Validate generated metadata surfaces**

Run:
```bash
rg -n 'com\.puupee\.todos|Puupee Todos|puupee_todos|puupeetodos|todos_setup' apps/todo || true
rg -n 'com\.felorx\.todo|Felorx Todo|felorx_todo|felorxtodo|todo_setup' apps/todo
```

Expected: first command has no output, second command shows the updated app identity surfaces.

### Task 4: Update Website, Scripts, And Seed References

**Files:**
- Modify: `website/src/lib/data/apps.ts`
- Modify: website tests that assert the Todo app id/path/name
- Modify: `scripts/apply_todos_pricing_seed.dart`
- Modify: `scripts/sync_todos_pricing_backlog_puupee.dart`
- Modify: `scripts/generate_inno_setup_configs.sh`
- Modify: `scripts/create_app_configs.sh`
- Modify: `scripts/update_to_new_startup.sh`
- Modify: `scripts/build.dart`, `scripts/build.sh`, `scripts/melos-build.sh`, `scripts/build_windows_installer.sh`
- Modify: `packages/tools/puupee_cli/bin/seed_todos_app_features.dart`
- Modify: `packages/tools/puupee_builder_cli` tests and examples that hard-code `todos`

- [ ] **Step 1: Apply application identity mappings**

Apply these mappings only to app identity, path, package, artifact, and docs references:
```text
apps/todos -> apps/todo
/apps/todos -> /apps/todo
id: 'todos' -> id: 'todo'
contentType: 'todos' -> contentType: 'todo'
name: 'puupee_todos' -> name: 'felorx_todo'
displayName: 'Puupee Todos' -> displayName: 'Felorx Todo'
Puupee Todos -> Felorx Todo
Todos -> Todo
todos app/features/pricing script identifiers -> todo app/features/pricing script identifiers when they describe the app
PUUPEE_TODOS_APP_ID -> FELORX_TODO_APP_ID
puupee_todos -> felorx_todo
```

Do not rename variables that clearly mean a collection of todo items, such as `todosAppId` only after first renaming it to `todoAppId` in the same file.

- [ ] **Step 2: Validate website tests**

Run:
```bash
cd website
npm test -- --runInBand
npm run type-check
```

Expected: tests and type-check pass.

- [ ] **Step 3: Validate builder CLI targeted tests**

Run:
```bash
cd packages/tools/puupee_builder_cli
dart test test/build_workspace_service_test.dart test/artifact_archive_service_test.dart test/artifact_path_resolver_test.dart test/build_unit_planner_test.dart test/build_all_units_test.dart
```

Expected: targeted tests pass with updated `todo` expectations.

### Task 5: Commit And Residual Audit

**Files:**
- All files changed by Tasks 1-4.

- [ ] **Step 1: Run final residual scans**

Run:
```bash
rg -n 'apps/todos|/apps/todos|Puupee Todos|puupee_todos|com\.puupee\.todos|puupeetodos|todos_setup|PUUPEE_TODOS_APP_ID' . --hidden --glob '!**/.git/**' --glob '!**/build/**' --glob '!**/.dart_tool/**' --glob '!**/node_modules/**' || true
rg -n 'apps/todo|/apps/todo|Felorx Todo|felorx_todo|com\.felorx\.todo|felorxtodo|todo_setup|FELORX_TODO_APP_ID' . --hidden --glob '!**/.git/**' --glob '!**/build/**' --glob '!**/.dart_tool/**' --glob '!**/node_modules/**'
git diff --check
```

Expected: first scan only shows intentional legacy docs/changelog entries if retained; second scan shows updated identities; diff check exits 0.

- [ ] **Step 2: Commit**

Run:
```bash
git add pubspec.yaml .puupee/builder.yaml .vscode/launch.json apps/todo website scripts packages/tools/puupee_cli packages/tools/puupee_builder_cli codemagic.yaml templates
git commit -m "refactor(todo): 迁移 Felorx Todo 应用标识"
```

Expected: commit succeeds and `git status --short` is clean.
