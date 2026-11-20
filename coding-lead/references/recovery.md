# 故障恢復與 terminal 清理（僅遇異常或結束時載入）

狀態判斷以當前 `hc --skill`、`hc ls/get/read/wait` 及 repo / 測試證據為準。不要從單一 `working`、`done` 或 PID 存活猜測工程結果。

| 訊號 | 核實方式 | 可逆處理 |
|---|---|---|
| `hc wait` timeout | `hc get` 仍 working，螢幕顯示工具／子代理進度 | 正常等待或改做其他可獨立工作；不重新派工 |
| `blocked`（授權／詢問） | `hc read` 實際提示、誰有權核准 | 取得具體人工操作；不可用 `--force` 代替同意 |
| Worker 意外退出 | `hc get` finished/error、`hc read` 最後輸出、worktree status | 保留已完成變更；先恢復可用 Session 或交接後再派新任務，避免重做 |
| 首則任務可能未送達 | `first_message.sent`、`hc read`、啟動時 trust dialog | 只有確認未送達才補送，避免雙重執行 |
| 工程看似停止 | 比對當前 Issue DAG、最後 commit／測試時間、Lead 螢幕與有無活躍子代理 | 若仍有 ready Issue 而 Lead done，先回報或向原 Lead 送單次明確續作指示 |
| Git base／root 分歧 | `git status`、`git log --all --graph`、`git worktree list` | 保留 dirty worktree，以乾淨 clone/branch 整合；停止破壞性操作 |
| 來源資料不可靠 | 檢查來源授權、只讀快照、一致性、manifest、hash | 僅測合成資料，對正式資料標 BLOCKED |
| Windows 權限／ACL、browser automation 失敗 | 隔離測試路徑、實際測試輸出、CUA 權限 | 修正最小受控環境或標 NOT_VERIFIED，不繞過 OS 隔離 |
| Review 發現 blocking finding | 核對 fixed SHA、可重現反例 | 修正、重跑測試、對新 SHA 重審；舊 PASS 不沿用 |

## Terminal 清理必要條件

對每個 terminal 先檢查：

1. `hc get <id> --json` 的 `parent` 必須是**目前 Lead**，或已取得其他 owner 的明確確認；不可因 terminal 看似舊就關閉。
2. `state/status` 顯示非工作中，且 `hc read`、commit、test、review/result 均已收齊。
3. 該 Session 已無後續交辦或重新檢查需求；能由 Git / PR / 測試重現其成果。
4. 對仍 live 而可清理者先 `hc kill <id>`，確認 finished 再 `hc rm <id>`。已 finished 者依目前 `hc --skill` 指示直接移除。
5. 保留 Lead Session、commit、PR、Issue 與 handoff；已結束的外部停滯監控同步停用。

## 最終交接的三層結果

- 工程通過：commit + RED/GREEN + Lead 回歸 + 獨立 Review + 合成 E2E。
- 實際 UI／設備通過：有瀏覽器／剪貼簿／Windows 實測證據；沒做就標 NOT_VERIFIED。
- 正式或人工授權通過：由有權人實際核准；工程 PASS 不能代替。
