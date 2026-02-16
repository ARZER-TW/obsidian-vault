---
tags: [claude-code, configuration, reference]
aliases: [Claude Code 設定, Claude Code Setup]
---

# Claude Code 完整配置

最後更新：2026-02-16

---

## 記憶系統

三層架構，位於 `~/.claude/memory/`。每次 session 自動載入前兩層，按需載入第三層。

```
~/.claude/memory/
├── identity.md          # Layer 1: 永久事實（身份、偏好、環境）
├── working.md           # Layer 2: 活躍上下文（進行中專案、近期決策）
├── index.md             # Layer 3 索引
└── archive/
    ├── sessions/        # 每日 session 日記，月底壓縮
    ├── projects/        # 專案記憶
    ├── decisions/       # 技術決策（附理由）
    ├── people/          # 提到的人物
    └── patterns/        # 行為模式（由 /reflect 辨識）
```

### 運作機制

| 時機 | 動作 |
|------|------|
| Session 開始 | 讀取 identity.md + working.md + index.md，按 cwd 載入相關 archive |
| 聊天過程中 | 通過 Write Gate（5 條標準）過濾後，自動寫入對應層級 |
| PreCompact | hook 發出緊急信號，立刻存檔未儲存的觀察 |
| Session 結束 | hook 自動同步記憶到 Obsidian vault |

### Write Gate（5 條標準，至少符合一條才寫入）

1. 會改變未來行為？（例：偏好 Foundry 而非 Hardhat）
2. 有後果的承諾？（例：deadline 是週五）
3. 附帶理由的決策？（例：選 X 因為 Y）
4. 會再次用到的穩定事實？（例：Alice 是專案負責人）
5. 使用者明確說「記住這個」？

### 相關指令

- `/diary` -- 手動產生 session 日記
- `/reflect` -- 分析累積日記，辨識行為模式，升級到 identity.md

---

## Hooks 系統

位於 `~/.claude/hooks/hooks.json`，自動觸發的 shell 腳本。

### PreToolUse（工具執行前）

| Hook | 功能 |
|------|------|
| Dev server blocker | `npm run dev` 等必須走 tmux，否則攔截 |
| tmux reminder | 長時間指令（install/test/build）提醒用 tmux |
| git push review | push 前暫停，確認要不要繼續 |
| .md file blocker | 攔截不必要的 .md 建立（排除 Obsidian vault 和記憶系統路徑） |
| strategic compact | 編輯/寫入時建議手動 compaction |

### PostToolUse（工具執行後）

| Hook | 功能 |
|------|------|
| PR logger | `gh pr create` 後記錄 PR URL |
| Prettier | JS/TS 檔案編輯後自動格式化 |
| TypeScript check | .ts/.tsx 編輯後跑 tsc 型別檢查 |
| console.log warning | 編輯的檔案有 console.log 就警告 |

### 生命週期 Hooks

| Hook | 功能 |
|------|------|
| SessionStart | 輸出三層記憶狀態，提醒載入 |
| PreCompact | 發 CRITICAL 信號，記錄 compaction 事件 |
| Stop | 同步記憶到 Obsidian、最終 console.log 審計 |

---

## Rules 系統

位於 `~/.claude/rules/`，共 11 個規則檔，每次 session 自動載入。

| 規則檔 | 控制範圍 |
|--------|----------|
| `memory-system.md` | 三層記憶架構協議 |
| `language.md` | 繁體中文回覆 |
| `output-style.md` | 禁 emoji、禁 Co-authored-by |
| `coding-style.md` | immutability 優先、小檔案、錯誤處理 |
| `git-workflow.md` | hackathon 推 main、conventional commits |
| `agents.md` | agent 使用時機、平行執行 |
| `hooks.md` | hooks 系統說明 |
| `patterns.md` | API response / custom hooks / repository pattern 範本 |
| `performance.md` | model 選擇策略（Haiku/Sonnet/Opus）、context 管理 |
| `security.md` | OWASP 檢查清單、secret 管理 |
| `testing.md` | TDD 流程、80% 覆蓋率（hackathon 例外） |

---

## Skills（自訂技能）

位於 `~/.claude/skills/`，用 `/skill-name` 呼叫。

### 核心工作流

| 指令 | 功能 |
|------|------|
| `/plan` | 需求分析 + 實施計畫，等確認後才動手 |
| `/brainstorming` | 創意發想，探索需求和設計 |
| `/tdd` | 測試驅動開發流程 |
| `/code-review` | 程式碼品質審查 |
| `/build-fix` | 修 build 和型別錯誤 |
| `/quality-gates` | 四級品質閘門（PASS/CONCERNS/REWORK/FAIL） |
| `/systematic-debugging` | 系統化除錯流程 |
| `/verification-before-completion` | 完成前強制驗證 |

### Hackathon 專用

| 指令 | 功能 |
|------|------|
| `/hackathon-init` | 初始化專案：技術選型、鷹架、環境 |
| `/hackathon-workflow` | 階段管理和時間分配 |
| `/hackathon-review` | 模擬評審打分 |
| `/hackathon-demo` | Demo 腳本、備案、部署確認 |
| `/hackathon-submit` | 提交準備：程式碼清理、文件、雙語材料 |

### 區塊鏈 / 安全

| 指令 | 功能 |
|------|------|
| `/blockchain-notes <chain>` | 多鏈 Obsidian 筆記擴展 |
| `/security-review` | 安全審查清單 |
| `/owasp-security` | OWASP Top 10:2025 + ASVS 5.0 |
| `secure-contracts-tob/*` | Trail of Bits 多鏈漏洞掃描（Solana/Cosmos/Cairo/TON/Substrate/Algorand） |
| `auditmos-defi/*` | DeFi 審計（借貸/預言機/重入/滑點/數學精度/狀態驗證） |

### 生產力

| 指令 | 功能 |
|------|------|
| `/humanizer-zh` | 去除中文文本的 AI 寫作痕跡 |
| `/diary` | 產生 session 日記 |
| `/reflect` | 月度記憶反思和模式辨識 |
| `/pitch-deck` | 生成 PPT pitch deck |
| `/office-docx` `/office-pdf` `/office-pptx` | Office 文件處理 |
| `/youtube-clipper` | YouTube 影片剪輯 + 雙語字幕 |
| `/readme-generator` | 自動生成 README |
| `/handoff` | 工作交接文件 |

### UI / 前端

| 指令 | 功能 |
|------|------|
| `/ui-ux-pro-max` | UI/UX 設計（67 styles, 96 palettes, 57 font pairings） |
| `/frontend-design` | 前端介面設計，避免 AI 美學 |

### 開發工具

| 指令 | 功能 |
|------|------|
| `/dispatching-parallel-agents` | 平行 agent 調度 |
| `/subagent-driven-development` | 子 agent 驅動開發 |
| `/using-git-worktrees` | Git worktree 隔離開發 |
| `/devops-cicd` | CI/CD pipeline 設計和除錯 |
| `/devops-monitoring` | 監控和可觀測性策略 |
| `/concise-output` | 強制精簡輸出 |

---

## MCP Servers

| Server | 功能 | 用途 |
|--------|------|------|
| github | GitHub API | PR、issue、repo 操作 |
| filesystem | 檔案系統存取 | 直接讀寫 Obsidian vault |
| sequential-thinking | 結構化思考 | 複雜推理分步驟 |
| Notion | Notion API | 頁面搜尋/建立/更新 |

---

## Obsidian 整合

| 項目 | 設定 |
|------|------|
| Vault 路徑 | `/mnt/c/Users/tl/Desktop/ObsidianVault` |
| GitHub repo | `ARZER-TW/obsidian-vault`（public） |
| 自動同步 | Obsidian Git 插件，每 5 分鐘 commit + push |
| 記憶同步目標 | `Claude Notes/Memory/` |
| 同步內容 | identity.md -> Memory Mirror.md, working.md -> Working Context.md, diary -> Sessions/ |

### Vault 結構

```
ObsidianVault/
├── Inbox/
├── Daily Notes/
├── Projects/
├── Templates/
├── Claude Notes/
│   └── Memory/         # Claude 記憶自動同步到這裡
│       ├── Memory Mirror.md
│       ├── Working Context.md
│       └── Sessions/
└── Ethereum/            # 55 篇密碼學深度筆記
    ├── Ethereum MOC.md
    ├── 交易流程/
    ├── 密碼學基礎/
    ├── Ethereum 資料結構/
    ├── 帳戶與交易/
    ├── 區塊與共識/
    └── 進階主題/
```

---

## Agent 系統

內建特化 agent，hackathon 模式下多數 optional。

| Agent | 用途 | 何時觸發 |
|-------|------|----------|
| build-error-resolver | 修 build 錯誤 | build 壞了（永遠啟用） |
| code-reviewer | 程式碼審查 | 寫完程式碼後 |
| planner | 複雜功能規劃 | 大型 feature |
| architect | 系統架構設計 | 架構決策 |
| tdd-guide | TDD 引導 | 新功能/bug fix |
| security-reviewer | 安全審查 | 處理敏感程式碼 |
| e2e-runner | Playwright E2E 測試 | 關鍵使用者流程 |
| refactor-cleaner | 清理死碼 | 程式碼維護 |
| doc-updater | 文件更新 | 更新 docs |

---

## Hackathon 模式例外

啟用 hackathon context 時自動放寬的規則：

- 跳過 TDD、覆蓋率要求
- 跳過完整安全審計（只查 hardcoded secrets 和 XSS）
- 直接推 main，不走 PR
- 允許檔案到 1000 行
- 簡化錯誤處理為 try/catch
- Agent 都是 optional（除了 build-error-resolver）

---

## 檔案位置速查

| 類別 | 路徑 |
|------|------|
| Rules | `~/.claude/rules/` |
| Skills | `~/.claude/skills/` |
| Memory | `~/.claude/memory/` |
| Hooks config | `~/.claude/hooks/hooks.json` |
| Hook scripts | `~/.claude/hooks/memory-persistence/` |
| Settings | `~/.claude/settings.json` |
| 舊 MEMORY.md | `~/.claude/projects/-home-james/memory/MEMORY.md`（已重導向） |
| Obsidian vault | `/mnt/c/Users/tl/Desktop/ObsidianVault` |
