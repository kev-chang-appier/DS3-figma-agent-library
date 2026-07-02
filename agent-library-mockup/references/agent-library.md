# Agent Library — 知識統整

> **目的**：把 Figma 上設計完整的 AI Agent 對話元件，整理成一份 AI 可學習的規格，讓 AI 學會所有 property/value，日後能依需求直接組合出畫面。
> **Figma fileKey**：`xvxUGkh3KDidpIdwHwIKg4`（page「Agent」`0:1`，庫名 `Agent | Library`）
> 狀態：✅ 已確認 ｜ 🔍 已爬待補 ｜ ⬜ 未爬

---

## 一、心智模型（最重要）

整套東西 = **1 個核心容器 `Chatroom`** + **塞進它 slot 的對話內容 `Response/Turn`**，再由**屬性開關**與**外層容器寬度**衍生出兩種版面與 history 呈現。

```
版面（外層容器決定寬度）
└── Chatroom（核心容器，含 header / Chat thread slot / footer-輸入區）
        └── Chat thread (slot)
              └── Response/Turn ×N （一輪 = User message + AI message）
                    ├── Response/User message
                    └── Response/AI message
                          └── Response content (slot)：文字 / 互動卡片(提案) / 圖表 widget / 表格

Chat history（獨立元件）：依版面以「第二層 sidebar」或「整面板」呈現
```

**兩種版面不是靠獨立 layout 變體切換，而是用屬性組合：**

| | Full-page（滿版） | Side-panel（側欄） |
|---|---|---|
| 外層容器寬 | 1456（內容置中 960） | 500（靠左停靠） |
| Chatroom `Header` | **off**（隱藏） | **on**（顯示 avatar+標題+icon） |
| Chat input field `Type` | **Full page** | **Side panel** |
| Response/Turn `Size` | **Full chat** | **Chat panel** |
| user bubble max-w | 600 | 376 |
| 外層附加 | 只有頂 bar + icon rail | 多 breadcrumb + Cancel 列 |
| history 呈現 | 第二層 sidebar（300px），可收合 | 取代整個面板（500px），可返回 |

---

## 二、元件屬性參考

### 1. Chatroom（核心容器）✅ 主檔 `4:3279`
> ⚠️ MCP code-gen 只吐最上層 `Header`/`Hint`，其餘為**暴露的巢狀子元件屬性**，以 Figma 屬性面板為準。

| 群組 | Property | 值 | 預設 | 用途 |
|---|---|---|---|---|
| 頂層 | `Header` | bool | on | 標題列顯示與否，依需求 |
| 頂層 | `Hint` | bool | off | 顯示 agent 運作資訊（Multi-task，含 spinner +「Processing…」）|
| Chatroom header | `Minimize button` | bool | on | 右上 icon 鈕（more / 編輯）|
| Chat thread | `Type` | `Short` \| `Long` | Short | **Short**＝初始單則→對齊上；**Long**＝多則長對話→對齊下 |
| Chat thread | `Chat thread` | slot | Modified | 放對話內容（Response/Turn）|
| Chat input field | `Type` | `Full page` \| `Side panel` | Side panel | 依版面 |
| Chat input field | `States` | `Enabled` \|（其他操作狀態，enum 待補）| Enabled | 操作狀態 |
| Chat input field | `Alert` | bool | off | 輸入狀態有問題時顯示 |
| Chat input field | `Reference` | bool | off | 插入 reference / quote 時 |
| Chat input field | `Target` | bool | off | 參照介面中其他元素作為輸入 |
| Chat input field | `Uploaded files` | bool | off | 附件 attachment |

- footer 固定有：輸入框、toolbar（`+` add / `@` mention / 書本 reference／中間「Auto plan ▾」／send 鈕）、Disclaimer「AI may provide inaccurate info…」。
- Chat thread 內容最大寬 **960**。

### 2. Response/Turn（一輪對話）✅ 主檔 `6:3710`（component_set）
| Property | 值 | 說明 |
|---|---|---|
| `Size` | `Chat panel` \| `Full chat` | 兩種 view 選對，尺寸不同；連動傳給內部 User/AI message |

- Chat panel：bubble max-w 376、容器 ~472｜Full chat：bubble max-w 600、容器 ~960
- 結構 = 1 則 User message（上）+ 1 則 AI message（下）。

### 3. Response/AI message（AI 回應）✅ 主檔 `6:4587`
| 群組 | Property | 值 | 說明 |
|---|---|---|---|
| 頂層 | `Size` | `Chat panel` \| `Full chat` | 依版面 |
| 頂層 | `Type` | `Default` \| `Retry` | Retry＝無回應、需使用者重試 |
| 頂層 | `Response content` | slot | 回覆主文（可換內容，不限純文字）|
| 頂層 | `Show Quick response` | bool | 顯示快速選項；**只掛最後一則** |
| Progress status | `Steps` | `Done …`（通常選 done；進行中另有狀態，enum 待補）| 思考步驟 / ToolCallGroup |
| Reactions | `Credit` | bool | 此回應是否消耗 credit（divider + 點數）|

**圖層**：Response head（avatar +「Show thinking ▾」）→ Response content (slot) → Quick response（Action 按鈕，受 `Show Quick response` 控）→ Reactions（reference/讚/倒讚/link/branch + Credit）。

### 4. Response/User message（使用者訊息）🔍 主檔 `6:4397`
- 右對齊藍色半透明氣泡。slot：`mediaGroup2`、`image row`。
- code-gen 露出的 prop（待用面板確認完整）：`media`(bool)、`uploadedImgs`(bool)、`mediaGroup2`/`imageRow`(slot)、`size`、`type=user`。
- mediaGroup2 可放：Reference chip、Target nodes chip（trigger 等）。

### 5. Chat history（歷史清單）🔍 主檔：Full page `119:16575`／Side panel `6:3200`
| 容器 Property | 值 | 說明 |
|---|---|---|
| `type` | `Full page`（＋Side panel，待補）| 版面 |
| `scroll` | bool | 是否顯示捲軸 |

每列 **Chat session** props：`type`(`Group title`/`item`)、`state`(`Enabled`/`Selected`/`na`)、`icon`(bool)、`status`(bool, spinner)、`action`(bool, more 鈕)、`text`。
- Full page（300px）：標題「Appier AI Agent」+ 收合鈕、「+ New chat」、`{chatName}` 清單、底部「⚙ Agent settings」。
- Side panel（500px）：標題「Chats」+ 返回 `‹`、「+ New chat」、清單（**無** Agent settings）。

---

## 三、組合規則與互動邏輯（組畫面必守）

1. **版面切換**＝改 3 個開關：Chatroom `Header` + Chat input field `Type` + Response/Turn `Size`（外加外層容器寬度）。三者要一致（全 full-page 或全 side-panel）。
2. **對話對齊**：只有一則用 Chat thread `Type=Short`（對齊上）；多則用 `Long`（對齊下）。
3. **Quick response 規則**：只掛「目前最後一則 AI message」。使用者點某 Action＝把該文字當輸入送出 → 生成新的 Response/Turn → 前一則的 Quick response 隱藏。
4. **AI 回應結構（anatomy）**，建議分層：
   1. Steps（Show thinking）
   2. **Session 1 — Executive Summary / Key Findings**：直接回答的敘述洞察 / 行動摘要
   3. **Session 2 — Evidence / Data Visualization**：圖表卡片（widget）+ 結構化表格
   4. **Guiding question + Quick Reply**：引導提問（「want me to compare more?」「save the report?」…）走 Quick response 機制
   5. Quick actions / Credit
5. **輸入區附加狀態**：Reference / Target / Uploaded files / Alert 四個 boolean 各自對應 footer 內預設隱藏的 slot。

---

## 四、設計 token（片段）
- surface：primary `white`、secondary `#f4f6f9`、blue `#296aff`
- border/general `#d9dae1`；content：high `#000a3a`、med `rgba(0,10,58,.8)`、lowminus `rgba(0,10,58,.45)`
- user bubble 底色 compbg/bluelite `rgba(41,106,255,.12)`；selectedBlue `rgba(41,106,255,.07)`
- 字體 Inter：body1 14/20、body2 12/16、headline5 14/20 SemiBold
- radius：r4 / r8 / r12 / r20 / full(999)

---

## 五、起始錨點與識別（mockup clone / 重建來源）

> ⚠️ 定位優先序：**名稱 > component key > 現值 id**（id 會移動）。
> ⚠️ **`clone()` 只能同檔運作**；跨檔請用 component key `importComponentByKeyAsync` 重建。

### ⭐ Path C 主路 — 版型模板「元件」keys（import → detach → retext）
源檔 fileKey `VTxr1LgruTEL86yinALPw0`（「Agent | Library」）內，4 個版型已做成 component set
**`[test]Template_agent_pages`**（key `acd4b964a532223cc666a1fe0d2572cfd90f4757`，variant 屬性「Property 1」）。
每個 variant 是**含 chrome 的完整頁面**（1512×915）。跨檔用法：`importComponentByKeyAsync(key).createInstance()` → `detachInstance()` →（detach 後巢狀 instance 全保留）→ 改字。

| 版型（variant） | component key |
|---|---|
| full-page chat ( with AI agent) | `f860d871f076be9e0bd4bbc4df2d7816020f19c5` |
| side -panel chat ( with ai agent) | `630133a62a582f61c170ec64d3d0110d2ff74072` |
| full-page chat ( history) | `147c4a1a6dfde1eaab26e51de1d1d2db5c09fdc2` |
| side -panel chat ( history) | `6272eb91b96f4185339c70b37fb15d719bbcac12` |

> ⚠️ 需**已發佈**到 team library 才能跨檔 import。模板 turn 數固定（1）→ detach 後同檔 clone 補齊。
> 名稱有 `[test]` 前綴 = 仍在測試階段，正式化後 key 可能換，用前先驗證。

### （參考）原始版型 frame（未元件化前的 node-id，會移動）
Side-panel `1078:8781`｜Full-page `1078:8801`｜Side-panel+history `1078:8793`｜Full-page+history `1078:8816`

### Component keys（最穩定，跨檔/改名/移動不變；6 個已驗證可 `importComponentByKeyAsync`）
| 元件 | component key |
|---|---|
| Chatroom | `ca1f8c756f87b84bbab33d3a9c6bd0d7907fd07f` |
| Response/Turn | `556814d926be1e2bae0e89fe393f1cfc2bd4d89b` |
| Chat history | `d5f4085e0852db96fd6c6553bd9be9e6bb94e786` |
| Response/AI message | `392869ed49c3ffe5ac55cc8e9415002e04969044` |
| Response/User message | `9bca3b071b5217dbc43ef95a01e4ac955acc047a` |
| Chat input field | `984b14e273ca1ae7d76111d953bc94d2bd150e98` |

### App chrome keys（版型範本的外框；Path B 要一起匯入才像範本）
| 元件 | key | 尺寸 / 擺位 |
|---|---|---|
| `*Navi-top bar` | `6f44c526500b24c543869d7fcb0f5cb7633c3f15` | 1512×56，置頂 x0 y0 |
| `*sidebar 3/ primary sidebar` | `a73ccb76059e009a709585fa901c352fc911e3c6` | 56 寬，x0 y56，高度拉到滿（~859） |
| `Viewer editing`（**側欄版才有**：breadcrumb + Cancel 列） | `086f53ae232e6030fe89ce67bc06b882fa595388` | 1456×56 |

**Full-page 外框排版**：frame 1512×915 → Navi-top bar(1512×56, y0) + sidebar(56寬, x0 y56) + Chatroom(1456×859, x56 y56)。
**Side-panel 外框排版**：Navi-top bar + sidebar + Viewer editing 列，Chatroom 窄面板(500)靠左停靠。

### 測試檔內的範例（次要，僅當就在該檔工作時可同檔 clone）
fileKey `xvxUGkh3KDidpIdwHwIKg4`：Full-page `119:13019`｜Side-panel `119:11980`｜Full-page+history `119:15795`｜Side-panel+history `119:15787`（現值 id 會變，先按名稱找）

### 元件 spec 區（純元件、無 app 外框；只要對話本體時用，在測試檔 `xvxUGkh3KDidpIdwHwIKg4`）
| 元件 | 現值 id（會變） | 標籤 |
|---|---|---|
| Chatroom | `4:3279` | ① |
| Chat history (Side panel) | `6:3200` | ② |
| Response/Turn (Chat panel) | `6:3710` | ③ component_set |
| Response/User message | `6:4397` | ④ |
| Response/AI message | `6:4587` | ⑤ |
| Chat input field | `6:4856` | ⑥ |
| Chat history (Full page) 主檔 `119:16575` / 實例 `119:17097` | — | — |

> 註：使用者最初提供的前兩個版型 URL 標籤對調，已以 Figma 實際內容為準。

---

## 六、方法論（寫進 skill 時要強調）
- **Figma 屬性面板是 ground truth**；MCP `get_design_context` 產生的程式碼會「漏報」暴露的巢狀屬性（如 Chatroom 只露 Header/Hint）。抓 property 時優先參考面板截圖，或交叉比對。
- 讀主元件 prop 的正確做法：對**畫布上的獨立 spec 實例**（上表 node）跑 `get_design_context`，nested 函式的 prop type 才會浮現；對 slot 內被攤平的實例則讀不到。

---

## 七、待補（TODO）
- [ ] Response/User message 面板級完整屬性（media/reference/target/uploaded 開關語意）
- [ ] Chat input field `States` 完整 enum
- [ ] AI message `Steps`（Progress status）完整狀態 enum
- [ ] Chat history `type` 是否含 Side panel 值、兩版是否同元件
- [ ] **Widget / 圖表卡片**（AOV 折線圖卡：標題 / Create report / Date table）— 尚未納入 library，需建檔
- [ ] 把本文件轉成正式 skill（含觸發描述、組合範例）
