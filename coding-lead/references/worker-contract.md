# Worker 交辦契約（Dispatch 時載入）

依本票填入所有欄位；資訊缺失時由 Lead 從 Spec / Issue / Git 查證後再送。給 Agent 的第一則任務完整包含下列契約，以免 `hc send` 早於啟動而遺失。

```text
你是由 Coding Lead <LEAD_TERMINAL_ID> 派出的 Coding Worker。
Issue：<ISSUE_ID / URL>；Spec：<SPEC_PATH>；本票依賴：<BLOCKERS_ALREADY_ACCEPTED>
Repo / worktree：<ABSOLUTE_WORKTREE>；branch：<BRANCH>；固定 base SHA：<SHA>
只修改：<PATHS / BEHAVIOR>；不可修改／存取：<GUARDRAILS>
可驗證成功條件：<OBSERVABLE_ACCEPTANCE>
測試命令：<TESTS>；測試資料：<SYNTHETIC_FIXTURES>
權限邊界：<COMMIT/PUSH/NETWORK/INSTALL/REAL_DATA AUTHORIZATION>

1. 讀專案 AGENTS.md、Spec 和 Issue；確認 base/worktree 與安全條件符合。
2. 依 TDD 先建立會失敗的行為測試（RED，保留失敗摘要），完成最小修改後取得 GREEN，再做必要重構及回歸。
3. 只做本 Issue。不要修改 Lead worktree、其他 repo，也不要自行派子 Agent。
4. 每個 commit 使用明確檔案 stage；若授權不包含 commit，僅提交 diff / 測試證據供 Lead 處理。不得自行 push、merge 或擴大範圍。
5. 完成或真正 blocked 時，以 hc send <LEAD_TERMINAL_ID> 回報；保持 Session 可讀，等 Lead 核對後清理。

必須回報：
- Issue / branch / worktree / base SHA / head SHA；
- 修改檔案、commit SHA（如已授權）與 dirty status；
- RED 的實際命令、失敗證據；GREEN、完整測試命令與結果；
- 安全／資料來源驗證及剩餘未驗證內容；
- status = READY_FOR_INTEGRATION 或 BLOCKED；若 BLOCKED，最小必要人工動作。
```

## 派工檢查

- 先閱讀當下 `hc --skill` 與 `hc run --help`，核實 `--parent`、`--cwd`、`--name` 的可用形式；以 `hc run --bg` 執行互動式 Agent。
- 核對 `hc get <worker> --json` 的 `parent`、worktree 和 `first_message.sent`。若送達不確定，先讀螢幕再判斷，避免重複發送。
- 同時活躍 Worker 預設上限為 2；Reviewer 和 demo terminal 也需要各自 owner 和結束條件。
