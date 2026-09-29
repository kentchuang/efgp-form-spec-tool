# 📋 EFGP 電子表單規格確認與功能盤點互動工具
### EFGP Form Specification & Design Interactive Tool

> 專為鼎新 **EFGP (EasyFlow GP)** 電子流程表單開發量身打造的**單一檔案純前端視覺化設計、需求盤點與規格確認工具**。  
> 消除業務需求端 (User)、系統分析師 (SA) 與開發工程師 (PG) 之間的溝通鴻溝，讓表單規劃像積木拼裝一樣直覺流暢！

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Zero-Dependency](https://img.shields.io/badge/Dependencies-Zero-green.svg)](#-技術架構與資安規範-architecture--security)
[![Pure Frontend](https://img.shields.io/badge/Platform-Pure%20HTML5%20%2F%20JS-orange.svg)](#-技術架構與資安規範-architecture--security)
[![Security Review](https://img.shields.io/badge/Security-Passed%20(Claude%20Code%20Review)-brightgreen.svg)](Form_Specification_Interactive_Tool_Review.md)
[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-success.svg)](https://kentchuang.github.io/efgp-form-spec-tool/)

👉 **[點此立刻線上體驗 (Live Demo)](https://kentchuang.github.io/efgp-form-spec-tool/)**

---

## 💡 為什麼需要這個工具？ (Why This Tool?)

在企業導入或客製鼎新 EFGP 電子表單時，最常遭遇以下痛點：
1. **溝通成本極高**：業務需求單位 (User) 用 Word/Excel 畫出無格線邏輯的草稿，工程師難以精準還原為 EFGP 的 Bootstrap 格線排版。
2. **規格遺漏與反覆修改**：欄位是否必填、開窗關聯代碼、資料來源（ERP / SAP RFC / 系統變數）以及簽核關卡權限經常在開發後期才發現定義不明。
3. **流程與表單權限混淆**：傳統在設計師設定欄位權限極度繁瑣，業務單位難以理解流程各關卡與表單欄位填寫權責的對應。
4. **歷史表單逆向困難**：接手維護舊表單時，面對龐雜的 `.form` XML 檔案，難以快速掌握所有欄位清單與排版架構。
5. **排版衝突與元件遺失**：複雜表單編輯時，常因欄位槽位碰撞、樣板容量超出或刪除列操作造成元件隱形消失或變成孤兒資料。

**本工具將「訪談確認 ➔ 視覺化排版 ➔ 防呆規格定義 ➔ 智慧槽位防護 ➔ 區塊權限矩陣 ➔ 匯出簽核」整合為一體化流程！**

---

## 🌟 核心功能特色 (Key Features)

### 1. 🚀 零環境相依・企業級資安與完整性 (Zero Dependency & Enterprise Security)
- **單一 HTML 檔案**：純前端單檔架構（5,600+ 行純 HTML5/Vanilla JS），不需安裝 Node.js、Python 或架設任何後端資料庫。
- **雙擊即可開始**：只要有現代瀏覽器（Chrome, Edge, Firefox, Safari），下載後直接開啟即可設計。
- **100% 離線純前端運作**：所有規格資料僅儲存於瀏覽器本機 `localStorage`，無任何後端伺服器連線或資料外洩風險。
- **通過 Claude Code Security Review 資安審查**：
  - 全面防堵 DOM-based 跨站腳本攻擊 ([CWE-79](https://cwe.mitre.org/data/definitions/79.html))，所有畫布列標題、下拉選單、SubTab、Grid 欄位與屬性面板均導入嚴格 `escapeHtml()` 跳脫機制。
  - **程式完整性保證**：HTML 105 個 DOM ID 與 JS 腳本 88 處元素參照 100% 吻合，102 個核心函式與 128 處行內事件精確綁定，無任何懸空指標或未定義元素。
  - 詳細審查報告請參閱 [Form_Specification_Interactive_Tool_Review.md](Form_Specification_Interactive_Tool_Review.md)。

### 2. 🎨 專注現代 RWD 響應式排版與流暢互動 (Visual RWD Designer & Fluid UX)
- **Bootstrap 12 格線體系**：支援單欄 (100%)、等寬雙欄 (50:50)、比例雙欄 (40:60 / 60:40 / 30:70 / 70:30)、三欄與四欄等多元響應式佈局。
- **26+ 種完整表單元件庫**：文字框 (TextBox)、多行文字 (TextArea)、下拉選單 (Dropdown)、單選選項 (Radio)、多選選項 (Checkbox)、開窗查詢複合元件 (Dialog/Label)、單身動態表格 (Grid)、日期選擇 (Date)、附件按鈕 (Attachment)、SubTab 頁籤容器等。
- **常設固定區塊**：預載標準「申請人基本資料」8 大常設欄位（工號、姓名、部門、分機、申請日期等）。
- **側欄浮動定位優化**：元件庫、欄位樣板與元件屬性面板採用強化 `sticky` 浮動佈局，長表單捲動時始終維持可視，不被頂端工具列遮蔽。
- **流暢且防誤觸之畫布互動**：
  - **點選空白處 / ESC 鍵取消選取 (Deselect)**：選取元件後若不想移動，點擊畫布外層空白區域或按下 `ESC` 鍵即可立即取消選取，避免誤點空格造成意外搬移。
  - **操作區塊與選取狀態精準同步**：點選畫布既有元件或拖曳搬移後，系統自動將作用中區塊 (`gActiveSectionId`) 鎖定為該元件所屬區塊，杜絕新元件誤置入其他區塊。
  - **區塊自由上下排序**：支援各業務區塊之一鍵「上移 / 下移」排序功能。
  - **直覺顯示資料來源 (Source)**：畫布元件方框直接顯示資料來源標籤（ERP 開窗、SQL 查詢、SAP RFC 或系統變數），方便業務即時核對。
  - **Attachment / SerialNumber 單表限量防呆**：附件與流水號元件具備單張表單限量新增檢查與按鈕狀態自動同步。
  - **代號彈性選填**：表單代號與欄位代號支援選填並自動產生，降低訪談初期的填寫負擔。

### 3. 🛡️ 獨家「智慧槽位防碰撞與自動修復引擎」(Collision Protection & Auto-Healing)
針對大型複雜表單最易發生的「元件神秘消失」痛點，內建全方位防碰撞與資料修復機制：
- **屬性變更目標格自動交換**：在右側屬性面板調整元件目標格時，若目標格已有其他元件，系統自動採取交換位置保護，杜絕覆寫導致元件隱形。
- **匯入自動修復 (`repairFieldColumnCollisions`)**：匯入舊版 JSON 或 `.form` XML 時，自動掃描每一列；若發現槽位碰撞或元件數量超出樣板欄數上限（例如 4 欄式樣板容納 5 個元件），自動遞補至同列空槽或動態新增延伸列安置，確保匯入時資料零遺失。
- **精準空位搜尋 (`getRowColCount`)**：新增元件時依據樣板實際可容納容量搜尋空位，額滿時自動於同區塊開闢新列安置，從根本杜絕指派超出可視範圍之無效槽位。
- **列刪除元件保護 (`deleteRowInDesigner`)**：刪除某一列時，自動將該列元件安全轉移至鄰近列空槽或新列，杜絕元件成為無效列參照之「孤兒資料」。

### 4. ⚖️ 創新的「區塊權限矩陣」(Block-Based Permission Matrix)
- **區塊自動連動關卡**：預設為「申請填單」關卡；畫布中每新增一個業務區塊（如：審查意見、主管覆核），系統自動動態增列對應關卡。
- **直覺權責劃分**：該區塊所屬欄位於對應關卡預設為「編輯」，其餘區塊欄位自動為「唯讀」，精準契合企業實際作業邏輯。
- **極速批次操作**：支援單元格點擊快速輪巡（編輯/唯讀/隱藏）、單列橫向全關卡批次套用、單欄縱向全欄位批次套用，以及一鍵重置為區塊權責預設值。
- **內建系統防呆**：嚴格遵循 EFGP 規範，表單流水號 (`SerialNumber`) 禁設停用 (disable)。

### 5. 🔄 逆向工程：支援直接匯入 EFGP `.form` 原始檔
- **一鍵逆向剖析**：支援匯入 EFGP 匯出的 `.form` (XML) 原始設計檔，自動逆向剖析元件代號、中文名稱、排版位置與單身 Grid 定義，快速建立表單盤點清單。
- **舊檔自動容錯與修復**：匯入時自動調用修復引擎，處理舊系統可能存在的座標重疊與欄位異常。

### 6. 💾 即時自動儲存與草稿保護機制 (Auto-Save & Crash Recovery)
- **瀏覽器 LocalStorage 即時存檔**：編輯過程即時於背景自動保存最新草稿，並於導覽列即時更新存檔狀態徽章。
- **智慧還原提示**：意外關閉瀏覽器、重新整理或斷電時，再次開啟會主動提示還原未匯出的最新草稿，防止心血遺失。

### 7. 📤 跨格式成果匯出與專業列印 (Export & Print)
- **匯出 JSON**：保存完整規格結構，方便跨團隊傳遞、版本控制與二度載入。
- **匯出 Markdown 規格書**：一鍵產生標準需求規格文件，內含基本資訊、多列樣板清單、SubTab 結構、Grid 單身清單、`setFieldControl()` 關卡權限矩陣與三方規格簽認表，可直接貼入 HackMD、GitLab Wiki 或 Notion。
- **專業列印樣式 (Print Sheet)**：內建專屬列印 CSS，隱藏所有編輯按鈕並自動計算分頁與框線，一鍵轉為 PDF 或紙本供主管簽署確認。

---

## 👥 適用對象 (Target Audience)

| 角色 | 使用場景與效益 |
| :--- | :--- |
| **業務需求單位 (User)** | 透過白話元件字典與積木式排版，無需懂程式碼也能清楚表達表單期望。 |
| **系統分析師 (SA) / 顧問** | 現場訪談時直接開啟工具投影排版，同步確認欄位防呆、資料來源、槽位配置與簽核權限。 |
| **專案經理 (PM)** | 匯出專業 Markdown 或列印紙本簽核單，作為專案驗收與範疇基準 (Scope Baseline)。 |
| **前端/流程工程師 (PG)** | 依照規格書明確的元件 ID、觸發事件（`formCreate`, `checkPointOnClose` 等）與 `setFieldControl` 矩陣直接實作，大幅減少通靈與重工。 |

---

## 🚦 4 步快速上手 (Quick Start Workflow)

```mermaid
flowchart LR
    A["1. 填寫基本資料\n(表單名稱/目標背景)"] --> B["2. 視覺化排版\n(區塊/樣板/放置元件)"]
    B --> C["3. 防呆與資料來源\n(來源/必填/正則檢核)"]
    C --> D["4. 權限矩陣與匯出\n(關卡矩陣/JSON/MD/列印)"]
```

1. **填寫基本資料**：設定表單中文名稱、Form ID（如 `MaGpTrans`，選填）與需求背景摘要。
2. **視覺化排版**：新增業務區塊，挑選列樣板（單欄/雙欄/比例欄/多欄），點選空格或使用元件庫放置元件，支援隨時拖曳、點選空格搬移或按 `ESC` 取消選取。
3. **設定防呆與規則**：點擊畫布上的元件，於右側面板設定標籤名稱、資料來源 (ERP/SAP/SQL)、必填狀態、下拉選單選項或驗證正則。
4. **權限確認與匯出**：至「區塊權限矩陣」確認各關卡權限連動，點選頂部按鈕匯出 **JSON**、**Markdown 規格書** 或**列印簽認單**。

---

## 🛠️ 技術架構與資安規範 (Architecture & Security)

- **核心技術**：純 HTML5 + Vanilla JavaScript (ES6+)，單一獨立 HTML 檔案（5,600+ 行代碼）。
- **樣式與圖示**：[Tailwind CSS (CDN)](https://tailwindcss.com/) + [FontAwesome 6 (CDN)](https://fontawesome.com/)。
- **資料儲存**：瀏覽器客戶端記憶體 + `window.localStorage`（100% 離線純前端，零後端相依）。
- **資安防護**：通過 [Claude Code Security Review](Form_Specification_Interactive_Tool_Review.md) 資安審核，全輸出 `escapeHtml()` 處理，徹底杜絕 DOM-based XSS (CWE-79)。
- **瀏覽器相容性**：支援現代主流瀏覽器（Google Chrome、Microsoft Edge、Mozilla Firefox、Apple Safari）。

---

## 📂 檔案使用說明

### 方式 A：線上免安裝使用
直接造訪 👉 **[GitHub Pages 線上展示](https://kentchuang.github.io/efgp-form-spec-tool/)**

### 方式 B：本機離線使用
1. 將本專案 Clone 或直接下載單一檔案：
   ```bash
   git clone https://github.com/kentchuang/efgp-form-spec-tool.git
   ```
2. 在檔案總管中直接雙擊開啟 `Form_Specification_Interactive_Tool.html`（或 `index.html`）。
3. 開始規劃您的電子表單規格！

---

## 🔗 相關文件 (Related Documentation)

- [🛡️ 工具程式完整性與資安修復報告 (Form_Specification_Interactive_Tool_Review.md)](Form_Specification_Interactive_Tool_Review.md)
- [💻 EFGP 專案客製開發規範與原則 (.agents/AGENTS.md)](../.agents/AGENTS.md)
- [📐 EFGP RWD 表單 JS 設計規範指南 (.agents/skills/efgp_rwd_form_js/SKILL.md)](../.agents/skills/efgp_rwd_form_js/SKILL.md)

---

## 📄 授權條款 (License)

本專案採用 [MIT 授權條款](LICENSE) - 歡迎自由取用、修改、分發與整合至企業內部流程中。
