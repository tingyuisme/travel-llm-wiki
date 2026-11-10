# AGENTS.md — 個人旅遊知識庫 (Travel LLM Wiki Schema)

本專案採用 Andrey Karpathy 提出的 **LLM Wiki** 架構，旨在建立一個**「可長期累積、跨趟旅程複用、持續演進」**的個人旅遊知識庫，並以 **Obsidian** 作為視覺化瀏覽與筆記介面。

身為維護此知識庫的 LLM Agent，你必須嚴格遵守本文件的架構、資料格式與標準作業流程（SOP）。**使用者負責提供素材、篩選興趣與決定方向；你負責所有的知識編譯、交叉連結（Cross-referencing）、矛盾偵測與帳務維護（Bookkeeping）。**

---

## 1. 核心架構與目錄規範 (Architecture)

知識庫分為三個核心層級與一個共用資源夾：

```text
LLM Wiki Notes/
├── AGENTS.md                # 【Schema 層】知識庫憲法、規範與工作流程（本文件）
├── assets/                  # 【共用資源】Obsidian 預設圖片與附件存放區（raw 與 wiki 皆可引用）
├── raw/                     # 【原始資料層】唯讀 (Immutable)！LLM 僅讀取，絕不修改或刪除既有原始檔
│   ├── articles/            # 網頁文章、部落格遊記（如 Obsidian Web Clipper 擷取檔）
│   ├── social/              # 社群貼文與零散筆記（IG、Threads、小紅書、PTT/Dcard 貼文或截圖）
│   ├── videos/              # YouTube 旅遊影片逐字稿、字幕或時間軸筆記
│   └── bookings/            # 機票、住宿、交通票券、預約憑證與訂單資訊
└── wiki/                    # 【知識庫層】由 LLM 全權維護、持續編譯與更新的 Markdown 網路
    ├── index.md             # 知識庫總目錄（Content-oriented catalog，每次 Ingest/變更必更新）
    ├── log.md               # 時間軸操作日誌（Chronological append-only log）
    ├── destinations/        # 國家 / 廣域地區 / 縣市 / 街區總覽（例如：鳥取縣.md、松江市.md）
    ├── places/              # 具體實體頁面（景點、餐廳、咖啡廳、購物點、溫泉飯店）
    ├── guides/              # 主題攻略、交通票券規則、季節穿搭、退稅/入境須知（例如：山陰冬季交通攻略.md）
    ├── trips/               # 單趟旅程專案與行程表（例如：2027-02-山陰鳥取島根8天7夜.md）
    ├── slides/              # 行程簡報與投影片（Marp Markdown 簡報與瀏覽器互動式 HTML 簡報）
    └── sources/             # 原始資料的結構化摘要頁（一對一記錄每份 raw 檔案提取出的所有資訊）
```

### 鐵則 (Golden Rules)
1. **原始資料神聖不可侵犯**：`raw/` 內的檔案是唯一事實來源（Source of Truth）。除非使用者直接將文字/圖片貼在對話中請你代存進 `raw/`，否則絕不修改 `raw/` 既有內容。
2. **知識與行程分離**：
   - `destinations/`、`places/`、`guides/` 是**長期可複用的知識資產**。
   - `trips/` 是**具時效性與特定需求的單次行程專案**，透過 `[[WikiLinks]]` 引用上述知識頁面。
3. **不重複造輪子，持續累積（Compound Knowledge）**：新增資料時，先檢查 `wiki/index.md` 是否已有對應頁面。若有，必須**整併更新（Update & Synthesize）**既有頁面，而非建立重複檔案。
4. **主動標記矛盾與時效風險**：旅遊資訊極易隨時間與季節變動（如：票價調漲、營業時間更改、冬季公休、預約制度變更）。當新舊資料衝突時，必須在頁面中明確標註雙方說法、來源日期與待確認事項（例如：`> [!WARNING] 時效/資訊衝突提醒`）。

---

## 2. 進行中的重點旅程 (Active Trip Context)

目前正在籌備的核心旅程（在 Ingest、Query 與 Lint 時應特別留意相關季節與區域特性）：
- **旅程頁面**：`[[2027-02-山陰鳥取島根8天7夜]]` (`wiki/trips/2027-02-山陰鳥取島根8天7夜.md`)
- **旅遊日期**：2027/02/12（五）～ 2027/02/19（五），共 8 天 7 夜
- **目標區域**：日本山陰地區（`[[鳥取縣]]`、`[[島根縣]]`）
- **季節與特別注意事項（2月冬季）**：
  - **氣候與雪況**：日本海側冬季多雪、風大，需特別留意**JR/巴士停駛或誤點風險**、冬季自駕雪胎限制、戶外景點（如鳥取砂丘、大山、浦富海岸遊覽船）的冬季營業狀態。
  - **冬季限定體驗與美食**：松葉蟹季（11月～3月）、冬季溫泉（皆生溫泉、三朝溫泉、玉造溫泉、溫泉津溫泉等）、雪景庭園（足立美術館、由志園）。
  - **日照與營業時間**：2月日落較早（約 17:30-17:50），許多景點與鄉間餐廳冬季提早打烊或冬季休業（12月～2月），摘錄資訊時務必核對**冬季營業時間**。
  - **春節/連假期間**：2027/02/12～02/19 涵蓋華人農曆新年與日本冬季旅遊旺季，熱門溫泉旅館、螃蟹料理與特急列車需特別注意預約開放時間。

---

## 3. 標準作業流程 (Core Workflows)

### 操作一：Ingest（互動式消化新資料 — 兩階段確認制）
當使用者將新檔案放入 `raw/`，或在對話中提供文章、貼文、影片逐字稿、訂單時，**嚴禁未經討論就直接大量建立 `places/` 頁面**。必須嚴格執行以下兩階段流程：

#### Phase 1：解析、摘要與清單勾選（與使用者討論）
1. **歸檔原始資料**：若使用者直接貼上文字或圖片，先詢問或協助存成 `raw/` 下的對應檔案（命名格式：`YYYY-MM-DD_來源簡稱_主題.md`）。若文章含圖片，確保圖片位於 `assets/` 並逐一檢視圖片內容以擷取隱藏資訊（如菜單價格、時刻表、地圖截圖）。
2. **回報重點摘要與待選清單**：在對話中向使用者報告：
   - **核心重點與亮點**：這份資料最有價值的情報、作者評價或踩雷提醒（特別標出與「冬季/2月」或「交通/預約」有關的細節）。
   - **資訊衝突/更新偵測**：是否與目前 Wiki 既有頁面產生矛盾或補充？
   - **實體勾選清單（Entity Selection List）**：列出資料中出現的所有景點、餐廳、飯店、交通攻略，並標註簡短介紹與關聯狀態（`[已存在於 Wiki，建議更新]` 或 `[新發現]`），**請使用者挑選/勾選哪些有興趣建立獨立的 `wiki/places/` 或 `wiki/guides/` 頁面**。

#### Phase 2：編譯寫入與交叉連結（使用者確認後執行）
收到使用者的挑選結果後，一次完成以下所有檔案維護：
1. **建立來源摘要頁（`wiki/sources/`）**：
   - 為該份原始資料建立對應摘要頁，完整記錄該來源提到的**所有**資訊（包含使用者未勾選建立獨立頁面的順帶店家，留在 Source 頁備查即可，不弄髒 `places/`）。
2. **建立/更新勾選的實體頁與攻略頁（`wiki/places/`、`wiki/guides/`）**：
   - 使用者勾選有興趣的項目，建立獨立頁面或更新既有頁面（累加 `source_count`、更新 `last_updated`、整合多方評價）。
3. **更新區域總覽與行程頁（`wiki/destinations/`、`wiki/trips/`）**：
   - 將新地點的 `[[WikiLink]]` 加入所屬縣市/地區頁面（如 `[[鳥取縣]]`、`[[島根縣]]`）。若使用者指定排入某天行程，同步更新對應的 `wiki/trips/`。
4. **更新總目錄（`wiki/index.md`）與日誌（`wiki/log.md`）**：
   - 在 `index.md` 新增或更新條目摘要。
   - 在 `log.md` 尾端追加一筆標準格式的 `ingest` 紀錄。

---

### 操作二：Query & Plan（查詢、比較與行程編排）
當使用者針對知識庫提問（例如：「幫我比較玉造溫泉與皆生溫泉」、「排出松江市一日遊順路動線」）：
1. **先讀目錄再鑽研**：先讀取 `wiki/index.md` 鎖定相關頁面，再讀取對應的 `places/`、`destinations/`、`guides/`。
2. **附上明確引用**：回答時使用 `[[頁面名稱]]` 標註知識來源。
3. **知識回存（File Back to Wiki）**：
   - 凡是產出有價值的**行程草案、比較表、交通轉乘方案、預算表**，在回答後主動詢問（或依指示）將結果寫回 `wiki/trips/` 或 `wiki/guides/`，並記錄於 `index.md` 與 `log.md`，讓探討成果留在知識庫中持續複利累積。

---

### 操作三：Lint（知識庫健康檢查與行前雷達）
當使用者要求執行 `Lint` 或健康檢查時，全面檢視 `wiki/` 並產出檢查報告：
1. **矛盾與過期資訊（Contradictions & Staleness）**：不同來源對營業時間、公休日、票價、預約方式的說法是否衝突？是否缺少 2 月冬季營業確認？
2. **動線與交通可行性（Routing & Transit Gaps）**：行程中的景點銜接是否合理？山陰地區 JR/巴士班次較少，轉乘時間是否經過驗證？
3. **預約與待辦缺口（Reservation Radar）**：標記為 `reservation_required: true` 或 `unknown` 的餐廳/體驗/車票，是否已查明開放預約日與預約管道？
4. **結構完整性（Orphans & Missing Pages）**：
   - 是否有沒有被任何 `destinations/` 或 `index.md` 連結到的孤兒頁面？
   - 是否有被多次 `[[提及]]` 卻尚未建立頁面的重要樞紐概念或地區？
5. 將 Lint 結果追加記錄至 `wiki/log.md`。

---

### 操作四：Slide Deck Graph（三階段投影片編譯工作流）
當使用者要求將 `wiki/trips/` 的行程製作成簡報或投影片時，**必須先載入專屬技能檔 `.agents/skills/slide-deck-graph/SKILL.md`**，並嚴格依照以下三階段狀態機（File-Backed State Graph）推進，**禁止跳過大綱與逐頁內容確認就直接生成投影片**：

1. **Stage 1：大綱規劃與版型配置（`wiki/slides/<旅程>_01_大綱.md`）**
   - **`Node 1 (Outline_Generator)`**：規劃總頁數、敘事動線、每頁主題，並為每一頁綁定標準版型代號（`L1_Cover`～`L7_TableBudget`）與引用的 `[[Wiki頁面]]`，寫入 `01_大綱.md`。
   - **`Gate 1A (大綱自動檢查)`**：檢查景點/餐廳/住宿覆蓋率是否 100%、每頁區塊數是否未超過該版型上限、公休日與季節順序是否合理。
   - **`Gate 1B (使用者確認 HITL 1)`**：請使用者確認大綱（可於對話提出修改，或直接於 Obsidian 編輯 `01_大綱.md`）。
2. **Stage 2：逐頁內容稿與字數預算編譯（`wiki/slides/<旅程>_02_逐頁內容稿.md`）**
   - **`Node 2 (Content_Drafter)`**：大綱確認後，依各版型的**「欄位字數預算表」**一次產出全簡報所有頁面的完整逐頁內容稿，寫入 `02_逐頁內容稿.md`（作為唯一文案事實來源 Single Source of Copy）。
   - **`Gate 2A (逐頁內容自動檢查)`**：檢查單行標題（`H1`、`H3`、`H4`、時間軸粗體、短列點）是否超標、執行多行段落**「行尾餘數預檢」**（消除最後一行僅剩 1～2 個孤字的情況）、核對 Wiki 事實與繁體字形。
   - **`Gate 2B (使用者確認 HITL 2)`**：請使用者一次審閱全頁文字稿（可對話修改或直接於 Obsidian 修改 `02_逐頁內容稿.md`）。
3. **Stage 3：投影片渲染與全頁視覺 QA（`wiki/slides/<旅程>_互動簡報.html` ＆ `_Marp簡報.md`）**
   - **`Node 3 (Slide_Builder)`**：讀取核准的 `02_逐頁內容稿.md` 編譯為 HTML 互動簡報與 Marp 簡報。
   - **`Gate 3A (全頁視覺排版自動檢查)`**：逐頁（P1～Pn）掃描確認：① **文字與郵戳標籤零重疊**（採 Flex 並排）、② **零單字/雙字斷行孤字**（含 `data-lock-style` 鎖定與視覺行偵測合併）、③ **雙層頁首平衡**（右上膠囊不留白、下排 `H1` 全寬單行不斷行）、④ **16:9 舞台無溢出**。
   - **`Gate 3B (使用者最終驗收 HITL 3)`**：交付預覽並依回饋微調，定案後同步更新 `wiki/index.md` 與 `wiki/log.md`。

---

## 4. 頁面格式與 YAML Frontmatter 規範

所有 `wiki/` 下的內容頁面（`index.md` 與 `log.md` 除外）都必須包含 **YAML Frontmatter**，以支援 Obsidian Properties 與 Dataview 查詢，並統一使用繁體中文與 `[[雙向連結]]`。

### 4.1 地點頁 (`wiki/places/<名稱>.md`)
```yaml
---
type: place
category: sight | food | cafe | hotel | onsen | shopping
country: 日本
prefecture: "[[鳥取縣]]" # 或 "[[島根縣]]"
city: 鳥取市 # 例如：米子市、倉吉市、松江市、出雲市
area: 鳥取砂丘周邊
status: wishlist | planned | booked | visited | skipped
winter_open: confirmed | caution | unknown # 2月冬季營業狀態確認
reservation_required: true | false | unknown
price_range: "¥1,500-¥3,000"
hours: "09:00-17:00 (冬季可能縮短)"
closed_days: "週二"
google_maps: ""
official_url: ""
source_count: 1
last_updated: YYYY-MM-DD
tags:
  - 山陰
  - 鳥取
---
```
**內文必備段落**：
- `## 一句話亮點`：為什麼想去這裡？核心特色是什麼？
- `## 綜合情報與評價`：彙整各個來源的推薦必看/必點、停留時間建議、優缺點與防雷提醒。
- `## 營業、預約與冬季注意事項`：公休日、預約方式、2月雪季/冬季特別限制（若有衝突資訊，以 `> [!WARNING]` 標示）。
- `## 交通方式與周邊順遊`：如何抵達（搭什麼車、哪一站下車）、鄰近可串聯的 `[[地點]]`。
- `## 資訊來源`：列出引用的 `[[sources/...]]` 連結與重點摘錄。

### 4.2 區域總覽頁 (`wiki/destinations/<區域>.md`)
```yaml
---
type: destination
level: region | prefecture | city | area
country: 日本
parent: "[[山陰地區]]"
last_updated: YYYY-MM-DD
tags:
  - 目的地
---
```
**內文必備段落**：區域地理與特色簡介、主要分區與交通樞紐、已收錄的景點/美食/住宿清單（依子區域或類別分類，使用 `[[WikiLinks]]`）、區域內交通攻略連結。

### 4.3 攻略與主題頁 (`wiki/guides/<主題>.md`)
```yaml
---
type: guide
category: transport | pass | weather | food_guide | shopping_tax | packing
target_region: "[[山陰地區]]"
source_count: 1
last_updated: YYYY-MM-DD
tags:
  - 攻略
---
```

### 4.4 旅程專案頁 (`wiki/trips/<旅程名稱>.md`)
```yaml
---
type: trip
trip_name: "2027冬・山陰鳥取島根8天7夜"
start_date: 2027-02-12
end_date: 2027-02-19
days: 8
nights: 7
destinations:
  - "[[鳥取縣]]"
  - "[[島根縣]]"
status: planning | booked | completed
last_updated: YYYY-MM-DD
---
```

### 4.5 原始資料摘要頁 (`wiki/sources/<YYYY-MM-DD_來源名稱>.md`)
```yaml
---
type: source
source_type: article | social | video | booking
raw_file: "[[raw/articles/...]]"
original_url: ""
author: ""
published_date: YYYY-MM-DD
ingested_date: YYYY-MM-DD
regions:
  - "[[鳥取縣]]"
---
```
**內文必備段落**：
- `## 來源核心摘要`：作者的行程路線、季節背景、核心觀點。
- `## 提及地點與情報總表`：完整列出此來源提到的**所有**地點（包含已建獨立頁面的 `[[地點]]`，以及未建獨立頁面的順帶提及地點與簡評，確保資訊不遺失）。

---

## 5. `index.md` 與 `log.md` 維護格式

### `wiki/index.md` 格式
- 分類清晰的總目錄：分為 **進行中旅程 (Active Trips)**、**目的地總覽 (Destinations)**、**景點與體驗 (Sights & Onsen)**、**美食與咖啡 (Food & Cafe)**、**住宿 (Hotels)**、**實用攻略 (Guides)**、**已消化來源 (Sources)**。
- 每個條目格式：`- [[頁面名稱]] — 一句話重點摘要（區域 | 狀態 | 更新日期）`

### `wiki/log.md` 格式
- 採 **Append-only（只追加不覆寫）**，最新紀錄加在檔案最下方（或統一固定順序，本庫採**由舊到新追加於檔案底部**，方便 `tail` 檢視）。
- 每筆紀錄標題必須嚴格遵循可解析格式：
  `## [YYYY-MM-DD] <動作: ingest | query | plan | slide | lint | init> | <簡短標題>`
- 標題下方以條列記錄：新增了哪些頁面、更新了哪些頁面、主要變更摘要。

---

## 6. Git 版本控制與 GitHub 同步規範 (Version Control & Backup)

本知識庫採用 **Git + GitHub 遠端私有儲存庫（Private Repository）** 進行版本管理與異地備份。

1. **自動化 Commit 觸發時機（Auto-Commit on Milestone）**：
   - 每當完成一個完整的知識庫里程碑（包含：**`Ingest` Phase 2 編譯完成**、**`Query & Plan` 行程回存完成**、**`Slide Deck Graph` Stage 1 / Stage 2 / Stage 3 定案**、**`Lint` 健康檢查完成**），Agent 應自動執行 `git add -A` 與 `git commit`。
2. **Commit Message 命名規範（對齊 `wiki/log.md`）**：
   - Commit 標題統一採用與 `wiki/log.md` 一致的結構：
     `<type>: <簡短標題>`（例如：`ingest: 消化樂吃購日本鳥取冬季行程與建立8處地點頁`、`slide: 完成2027山陰8天7夜三階段簡報編譯與視覺QA`、`plan: 更新Day3溫泉候選與Day6日御碕行程`）。
3. **遠端同步（GitHub Private Repo Push）**：
   - 若已綁定遠端儲存庫（`origin`），於每次 Milestone Commit 後自動執行 `git push` 同步備份至 GitHub。
4. **忽略清單（`.gitignore`）**：
   - 排除 `.obsidian/workspace.json`、`.obsidian/workspace-mobile.json`、`.obsidian/cache`、`.trash/` 等頻繁變動的視窗狀態檔，確保 Commit 歷史乾淨聚焦於知識與設定本身。
