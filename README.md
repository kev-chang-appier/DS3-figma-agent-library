# DS3 Figma Agent Library

把 Figma 上設計完整的 **Appier AI Agent 對話元件**（`Agent | Library`）整理成一份「AI 可學習的規格」，讓 AI 學會所有元件的 property / value 與組合規則，日後只要給一個主題，就能用**真正的 library 元件實例**在 Figma 組出一串人機對話 mockup —— 而不是重畫一堆矩形。

> **這是什麼**：一個 **純文件 / skill** 的 repo，本身沒有程式碼。它是給 AI agent（Claude Code + Figma MCP）讀的知識庫與工作流程。

---

## 這個 repo 在做什麼？

Appier 的設計團隊在 Figma 裡做好了一整套 AI Agent 聊天介面元件（Chatroom、對話輪、AI/使用者訊息、聊天歷史、輸入框…）。問題是：**AI 不會自動知道這些元件有哪些屬性、怎麼搭配才正確。**

這個 repo 解決的就是這件事：

1. **把元件規格寫成文件** —— 每個元件的屬性、可選值、預設值、設計 token，以及最穩定的 Figma **component key**（跨檔重建用）。
2. **把「怎麼組」寫成 skill** —— 一套 step-by-step 工作流程：先確認目的地 → 寫對話腳本並給使用者確認 → 用真元件實例在 Figma 組出來 → 截圖驗證。

最終目標：使用者只要說「用 agent library 幫我組一個 618 檔期行銷對話的 mockup」，AI 就能照著這份規格與流程，在指定的 Figma 檔案裡產出乾淨、可編輯的 mockup。

---

## 檔案結構

```
.
├── README.md                            ← 你正在看的這份
├── AGENT-LIBRARY-NOTES.md               ← 元件知識統整（原始筆記，中文）
└── agent-library-mockup/
    ├── SKILL.md                         ← Claude skill：組 mockup 的工作流程
    └── references/
        └── agent-library.md             ← 元件完整規格（skill 的 source of truth）
```

| 檔案 | 角色 | 內容重點 |
|---|---|---|
| **`agent-library-mockup/SKILL.md`** | **工作流程** | 何時觸發、Step 0–4 的建置步驟、Figma 操作的關鍵地雷 |
| **`agent-library-mockup/references/agent-library.md`** | **規格書（真理來源）** | 元件屬性表、組合規則、設計 token、component keys、版型模板 |
| **`AGENT-LIBRARY-NOTES.md`** | **原始筆記** | 與 references 幾乎同源的知識統整，含 TODO / 待補清單 |

---

## 心智模型（最重要的一件事）

整套東西 = **1 個核心容器 `Chatroom`** + **塞進它 slot 的對話輪 `Response/Turn`**：

```
版面（外層容器決定寬度）
└── Chatroom（header / Chat thread slot / footer 輸入區）
      └── Chat thread (slot)
            └── Response/Turn ×N（一輪＝User message + AI message）
                  ├── Response/User message
                  └── Response/AI message
                        └── Response content (slot)：文字 / 卡片 / 圖表 widget / 表格

Chat history（獨立元件）：依版面以「第二層 sidebar」或「整面板」呈現
```

兩種版面**不是**靠獨立 layout 變體，而是**同時翻三個開關**（三者必須一致）：

| | Full-page（滿版） | Side-panel（側欄） |
|---|---|---|
| Chatroom `Header` | **off** | **on** |
| Chat input field `Type` | **Full page** | **Side panel** |
| Response/Turn `Size` | **Full chat** | **Chat panel** |
| 外層容器寬 | 1456（內容置中 ~960） | 500（靠左停靠） |

---

## 核心元件一覽

| 元件 | 說明 | Component key |
|---|---|---|
| **Chatroom** | 核心容器（header / thread slot / 輸入區） | `ca1f8c756f87b84bbab33d3a9c6bd0d7907fd07f` |
| **Response/Turn** | 一輪對話（User + AI message） | `556814d926be1e2bae0e89fe393f1cfc2bd4d89b` |
| **Response/AI message** | AI 回應（含 thinking、快速回覆、reactions） | `392869ed49c3ffe5ac55cc8e9415002e04969044` |
| **Response/User message** | 使用者訊息（右對齊藍色氣泡） | `9bca3b071b5217dbc43ef95a01e4ac955acc047a` |
| **Chat history** | 聊天歷史清單（sidebar / 整面板） | `d5f4085e0852db96fd6c6553bd9be9e6bb94e786` |
| **Chat input field** | 底部輸入框 + toolbar | `984b14e273ca1ae7d76111d953bc94d2bd150e98` |

> 完整屬性、預設值、版型模板（`[test]Template_agent_pages`）與設計 token 見
> [`agent-library-mockup/references/agent-library.md`](agent-library-mockup/references/agent-library.md)。

---

## 怎麼使用（給 AI agent 的流程）

`SKILL.md` 定義的工作流程摘要：

1. **Step 0 — 確認目的地（硬性關卡）**：在建立任何節點前，先確認並複述目標 Figma 檔案 + frame / 位置。未確認絕不動手。
2. **Step 1 — 界定情境**：主題、版面（full-page / side-panel）、對話輪數、要不要 history、語言、是否含圖表 widget。
3. **Step 2 — 寫對話腳本並取得確認**：依「AI 回應 anatomy」（Thinking → Key Findings → Evidence/圖表 → 引導問題 + Quick Reply → Credit）產生純文字腳本，先給使用者過目。
4. **Step 3 — 用真元件在 Figma 組**：主路 Path C（import 版型模板 → `detachInstance()` → 改字 / 補齊輪數）；備援 Path B（用個別 component key 逐一組）。
5. **Step 4 — 截圖驗證並修正**：確認三個版面開關一致、快速回覆只掛最後一輪、對齊正確。

**關鍵規則**：
- 快速回覆（Quick response）**只掛在最新一輪 AI message**。
- 多輪對話 → Chat thread `Type=Long`（靠下對齊）；單輪 → `Short`（靠上）。
- 圖表 / 表格 widget **尚未納入 library**，需要時放「明確標示的虛線 placeholder 卡」，不要假造真圖表。

### 前置條件
- 目標 Figma 檔案需啟用 `Agent | Library`（否則 `importComponentByKeyAsync` 會失敗）。
- 呼叫 `use_figma` 前**必須**先載入 `figma-use` skill。

---

## 目前狀態與待補（TODO）

- [ ] `Response/User message` 面板級完整屬性（media / reference / target / uploaded 語意）
- [ ] Chat input field `States` 完整 enum
- [ ] AI message `Steps`（Progress status）完整狀態 enum
- [ ] Chat history `type` 是否含 Side panel 值、兩版是否同元件
- [ ] **Widget / 圖表卡片** 建檔納入 library
- [ ] 把筆記正式轉為 skill（觸發描述、組合範例）

---

## 參考

- **Figma fileKey**：`xvxUGkh3KDidpIdwHwIKg4`（測試檔）／`VTxr1LgruTEL86yinALPw0`（模板源檔，庫名 `Agent | Library`）
- 定位優先序：**名稱 > component key > 現值 node id**（id 會隨編輯移動，勿硬記）。
