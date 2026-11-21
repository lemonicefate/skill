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
2. 在寫測試或程式前，透過 **Codex 原生 Skill 機制**載入 Matt Pocock `tdd`：優先在本 Worker Session 的第一則 user/task 訊息明確加入 `$mattpocock-skills:tdd`（必要時由 Lead 用 `hc send` 傳送），也接受已能核實的 model-invoked 載入。檢查 Session 中確實出現對應 `<skill>` 注入區塊、名稱與來源路徑，才在已核准的 public-interface seam 按原版 `tdd` 進行 Red → Green。Codex 不需要獨立的 `Skill tool`；缺少此工具不是阻塞。若原生載入經檢查仍失敗，才回報 `BLOCKED_SKILL_INVOCATION`，停止本票實作。
3. 只做本 Issue。不要修改 Lead worktree、其他 repo，也不要自行派子 Agent。
4. 每個 commit 使用明確檔案 stage；若授權不包含 commit，僅提交 diff / 測試證據供 Lead 處理。不得自行 push、merge 或擴大範圍。
5. 完成或真正 blocked 時，**在自己的 Session 最終回覆**以 `READY_FOR_INTEGRATION` 或 `BLOCKED` 開頭，簡要列出以下欄位；長篇日誌、測試／GUI 證據寫入本票可讀的檔案並附絕對路徑。**不要**例行執行 `hc send <LEAD_TERMINAL_ID>`，Lead 會以 `hc wait --children any`、`hc get`、`hc read` 主動收件。僅有不能等待收件的緊急安全／授權問題，才允許一則簡短 `hc send`，並保留自己的完整報告。保持 Session 可讀，待 Lead 核對後清理。

自己的 Session 最終報告必須包含：
- Issue / branch / worktree / base SHA / head SHA；
- 修改檔案、commit SHA（如已授權）與 dirty status；
- `tdd` Skill 正式呼叫／載入紀錄（Session ID 與證據）、RED 的實際命令與失敗證據、GREEN 與完整測試結果；
- 安全／資料來源驗證及剩餘未驗證內容；
- status = READY_FOR_INTEGRATION 或 BLOCKED；若 BLOCKED，最小必要人工動作。
```

## 派工檢查

- 先閱讀當下 `hc --skill` 與 `hc run --help`；以 `hc run --bg --no-notify --parent <LEAD> --cwd <WORKTREE> --name <NAME> ...` 啟動新的互動式 Worker。Merger／Reviewer 同樣使用 `--no-notify` 和自己的 Session 最終報告。
- 核對 `hc get <worker> --json` 的 `parent`、worktree 和 `first_message.sent`。若送達不確定，先讀螢幕再判斷，避免重複發送；完成後 Lead 用 `hc wait --children any`、`hc get`、`hc read` 主動收件。
- 同時活躍 Worker 預設上限為 2；Reviewer 和 demo terminal 也需要各自 owner 和結束條件。
