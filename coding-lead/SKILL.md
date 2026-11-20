---
name: coding-lead
description: "Orchestrate multi-agent coding as the single Coding Lead. Use for implementing an approved spec or Issue DAG through Herdr Core hc, delegated Codex workers, isolated Git worktrees, TDD, integration, independent reviews, and safe terminal cleanup. Also use to resume or audit a stalled Coding Lead. Not for an isolated code edit."
---

# Coding Lead

以一位 **Engineering Integrator** 持有工程決策與整合權；其他 Agent 執行有邊界的 Issue。目標是可驗證的交付，不是增加 Agent 數量。此 Skill 不授權任何新的檔案、資料、網路、費用、部署或 Git 發布權限。

## 1. Preflight：形成可執行的工作契約

1. 讀當前 repo 的 `AGENTS.md`、已核准 Spec、canonical Issues／本機 tickets、交接記錄與驗收方式；找出單一 tracker、Issue 依賴 DAG、可獨立驗證的 vertical slices。若沒有核准 Spec，先回報最小缺口，勿自行擴大範圍。
2. 核對 `git status`、remote、default/base branch、root history、既有 worktree 與未提交變更。若本機與遠端根提交不同，保留原工作樹並以乾淨 clone/worktree 對齊，避免破壞性重置。
3. 執行 `hc --skill`，依**當前版本**的手冊使用 `hc ls/get/read/wait`；核對現有 Lead、owner、子代理與 Task，復用現存有效 Session，不重複派工。若 Herdr Core 不可用，明確標記，僅在使用者允許時改單代理順序執行。
4. 確認 Codex、預設模型、GitHub 認證、測試工具、合成 fixture、資料來源的授權／完整性，以及 `implement-spec`、`tdd`、`code-review` 的**實際 invocation contract**。只讀到 Skill 文件不等於成功呼叫；遇 user-only Skill，列出使用者需做的一次性操作。
5. 在開工前區分：允許自動做、需要人工核准、禁止操作。先跑 baseline tests。對不可驗證的資料來源先縮成 synthetic-only gate，將正式資料／真實環境驗收標為 BLOCKED。

**完成條件：** 有單一工程 owner、明確 base SHA、可派工 frontier、實測基線、權限與資料邊界；尚未滿足的前提被具體標記，而非默默假設。

## 2. Dispatch：少量、獨立、可交接

1. 只派 DAG 中所有 blocker 已滿足的 Issue。**預設最多 2 名同時存活的本任務 Coding Workers**；平行僅限不會互相改同一整合範圍的工作。由 Lead 決定整合次序。
2. 每票建立專屬 branch/worktree；指定固定 base SHA、修改範圍、驗收行為、TDD RED→GREEN→REFACTOR、測試指令、禁止事項及回報格式。可直接採用 [Worker 交辦契約](references/worker-contract.md)。
3. 由 Lead 透過 `hc run --bg` 啟動**可互動、可回報**的 Coding Agent；依本機可用設定優先 Luna Max Worker、Sol Medium Lead，先驗證實際模型名稱與 effort。不要假稱已切換 Lead 模型。明確 `--parent`、`--cwd`、有意義的 `--name`；核對 `first_message.sent`，避免啟動中途再送造成遺失或重複。
4. Worker 不可再開子 Agent，除非本次另有明確授權；Worker 不改 Lead worktree、不合併 main、不自行擴票。Lead 集中掌握 issue 狀態及 integration branch。

**完成條件：** 每個 active Issue 可唯一映射到 owner、Session、worktree、base SHA 和驗收契約；不存在超額或重複派工。

## 3. Monitor：辨別等待、停滯與阻塞

- 用 `hc get <id> --json` 看 `state`、`status`、`parent`、`turn`，再用 `hc read` 核實最近行為；用 `hc wait` 等待完成或訊息，**timeout 不是任務失敗**。
- `working` 且有測試、變更或有效子代理等待，是正常進度；`done` 不等於已整合；程序 `live` 不等於正在工作。真正 `blocked/error/exited`，或無進度而所有 Issue 無人接手，才診斷並處理。
- 排除權限提示、首則任務未送達、Git 分歧、測試環境、額度、依賴與失去父子連線。先讀證據再採可逆修復；不盲目重啟、重派、取消進行中的工作。故障分支見 [恢復與清理](references/recovery.md)。
- 在 tracker／PR 記錄每票進度、測試證據、阻塞與下一步；外部監控若啟用，只針對**活躍** Lead，交付後停用。

**完成條件：** 活躍票有人持續推進，真阻塞有具體原因與下一步；不因正常等待重複工作。

## 4. Integrate：Lead 親自驗收

1. Worker 回傳 Issue、base/head SHA、命名變更檔、可重現 RED／GREEN、完整測試結果、commit、剩餘風險。沒有可驗證證據不能標 DONE。
2. Lead 檢查 scope、diff、commit、dirty worktree 與來源 provenance，安全整合到唯一 integration branch；**在 Lead 工作樹重跑**受影響測試與整合回歸。測試 RED 為 TDD 中間證據，不代表最後已完成。
3. 以合成／隔離資料執行跨資料、API、UI 的 E2E；涉及 Windows ACL、真瀏覽器或剪貼簿時，額外在適當環境作實際驗證。不可用測試替代未執行的 GUI／真實裝置檢查。
4. 凍結同一個 `base_SHA..head_SHA` 和 diff，交**獨立唯讀** Standards 與 Spec reviewers；高風險專案再交對抗安全 reviewer（如可用的 Hydrogen）。各自先獨立提交 findings，不能把不存在的模型或未完成審查稱為 PASS。Blocking findings 修復後，對**新 SHA** 重新測試、重審。
5. Git push、Issue 關閉、PR Ready、merge、依賴變更及部署皆遵守當前使用者授權與 `AGENTS.md`；保留可審閱的 Draft PR，未授權不自動合併。

**完成條件：** 所有工程範圍有固定 commit、Lead 回歸、可重現 E2E、獨立審查結果；剩餘缺口清楚區分。

## 5. Close：收證據後才清理

- 以 `hc get/read` 逐一確認：這是**本 Lead 的子代理**、狀態已 `done/idle/finished`、結果與 commit/test 已收齊、未來不再呼叫。對仍 `working` 者保留；其他 owner 的 terminal 必須先由原 owner 確認用途。
- 對符合條件的 live terminal 用 `hc kill <id>`，確認 finished 後再 `hc rm <id>`；已 finished 者核對後可直接 `hc rm`。保留 Lead Session、原生歷史及 Git 證據供交接。
- 最後回報：Repo／integration branch／head SHA／PR、逐票結果、RED/GREEN 與全套測試、review findings／修正、尚未驗證項、最少人工動作。完成後停止本任務監控。

**完成條件：** 無遺漏的活躍工程 Issue、無孤兒子代理，且每個未完成的 GUI、正式資料、臨床／業務或上線批准都保留可追蹤的關卡。

## 完成語義與安全

- **ENGINEERING_DONE**：實作、合成測試、Lead 整合及指定 code review 過關。
- **GUI_VERIFIED**：指定真實瀏覽器／桌面／剪貼簿驗收實際通過；未做則 `NOT_VERIFIED`。
- **HUMAN_APPROVED**：有權人完成必要核准；不能由 Agent 的工程測試推定。
- **READY_FOR_MERGE / DEPLOY**：以上所有**本次約定**的必要關卡與 Git/部署授權確實到位。

對 secrets、個資／PHI、正式 HIS、不可逆變更與付費行為沿用專案最嚴格 guardrail；使用受限 OS／sandbox 權限保護資產。CLI 的 `--dangerously-bypass-approvals-and-sandbox` 不是安全隔離機制，不能因為 Prompt 寫了禁止就視為已隔離。
