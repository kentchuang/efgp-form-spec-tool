# 📋 EFGP 電子表單規格確認與功能盤點互動工具
### EFGP Form Specification & Design Interactive Tool

> 專為鼎新 **EFGP (EasyFlow GP)** 電子流程表單開發量身打造的**單一檔案純前端視覺化設計、需求盤點與規格確認工具**。  
> 消除業務需求端 (User)、系統分析師 (SA) 與開發工程師 (PG) 之間的溝通鴻溝，讓表單規劃像積木拼裝一樣直覺流暢！

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Zero-Dependency](https://img.shields.io/badge/Dependencies-Zero-green.svg)](#-快速開始-quick-start)
[![Pure Frontend](https://img.shields.io/badge/Platform-Pure%20HTML5%20%2F%20JS-orange.svg)](#-技術架構-architecture)

---

## 💡 為什麼需要這個工具？ (Why This Tool?)

在企業導入或客製鼎新 EFGP 電子表單時，最常遭遇以下痛點：
1. **溝通成本極高**：業務需求單位 (User) 用 Word/Excel 畫出無格線邏輯的草稿，工程師難以精準還原為 EFGP 的 Bootstrap 格線排版。
2. **規格遺漏與反覆修改**：欄位是否必填、開窗關聯代碼、資料來源（ERP / SAP RFC / 系統變數）以及簽核關卡權限經常在開發後期才發現定義不明。
3. **歷史表單逆向困難**：接手維護舊表單時，面對龐雜的 `.form` XML 檔案，難以快速掌握所有欄位清單與排版架構。

**本工具將「訪談確認 ➔ 視覺化排版 ➔ 防呆規格定義 ➔ 關卡權限矩陣 ➔ 匯出簽核」整合為一體化流程！**

---

## 🌟 核心功能特色 (Key Features)

### 1. 🚀 零環境相依・雙擊即用 (Zero Dependency)
- **單一 HTML 檔案**：不需安裝 Node.js、Python 或架設任何後端資料庫。
- 只要有現代瀏覽器（Chrome, Edge, Firefox, Safari），雙擊即可立刻開始設計。

### 2. 🎨 仿 EFGP 設計師的視覺化排版 (Visual Designer)
- **區塊與多列樣板 (Rows)**：支援單欄 (100%)、等寬雙欄 (50:50)、比例雙欄 (40:60 / 60:40 / 30:70 / 70:30)、三欄與四欄等彈性排版。
- **26+ 種完整表單元件庫**：文字框 (TextBox)、多行文字 (TextArea)、下拉選單 (Dropdown)、單選鈕/核取方塊 (Radio/Checkbox)、開窗查詢複合元件 (Dialog/Label)、單身動態表格 (Grid)、日期選擇 (Date)、附件 (Attachment)、SubTab 頁籤容器等。
- **常設固定區塊**：預載標準「申請人基本資料」8 大常設欄位（工號、姓名、部門、分機、申請日期等）。

### 3. 🔄 逆向工程：支援直接匯入 `.form` 原始檔
- **一鍵逆向解析**：支援匯入 EFGP 匯出的 `.form` (XML) 原始設計檔，自動逆向剖析元件代號、中文名稱、排版位置與單身 Grid 定義，快速建立表單盤點清單。

### 4. 🛡️ 關卡權限矩陣 (setFieldControl)
- **精準權限定義**：直接對應 EFGP JS 的 `setFieldControl()` 規範，視覺化配置填單人、直屬主管、權責單位、會辦主管與核准關卡之**編輯 (Editable) / 唯讀 (Read-only) / 隱藏 (Hidden)** 狀態。

### 5. 💾 自動儲存與草稿保護機制 (Auto-Save & Recovery)
- **瀏覽器 LocalStorage 即時存檔**：編輯過程即時背景自動備份。
- **智慧還原提示**：意外關閉瀏覽器或斷電時，再次開啟會主動提示還原未匯出的最新草稿，防止心血遺失。

### 6. 📤 跨格式成果匯出與列印 (Export & Print)
- **匯出 JSON**：保存完整規格結構，方便跨團隊傳遞、版本控制與二度載入。
- **匯出 Markdown 規格書**：一鍵產生結構化需求規格文件，可直接貼入 HackMD、GitLab Wiki 或 Notion。
- **專業列印樣式 (Print Sheet)**：內建專屬列印 CSS，隱藏所有編輯按鈕並自動計算分頁與框線，一鍵轉為 PDF 或紙本供主管簽署確認。

---

## 👥 適用對象 (Target Audience)

| 角色 | 使用場景與效益 |
| :--- | :--- |
| **業務需求單位 (User)** | 透過白話元件字典與積木式排版，無需懂程式碼也能清楚表達表單期望。 |
| **系統分析師 (SA) / 顧問** | 現場訪談時直接開啟工具投影排版，同步確認欄位防呆、資料來源與簽核權限。 |
| **專案經理 (PM)** | 匯出專業 Markdown 或列印紙本簽核單，作為驗收與範疇基準 (Scope Baseline)。 |
| **前端/流程工程師 (PG)** | 依照規格書明確的元件 ID、觸發事件（`formCreate`, `checkPointOnClose` 等）直接實作，大幅減少通靈與重工。 |

---

## 🚦 4 步快速上手 (Quick Start Workflow)

```mermaid
flowchart LR
    A["1. 填寫基本資料\n(表單名稱/代號/目標)"] --> B["2. 視覺化排版\n(挑選樣板/置放元件)"]
    B --> C["3. 防呆與資料來源\n(必填/來源/正規表達式)"]
    C --> D["4. 關卡權限與匯出\n(矩陣設定/匯出JSON/列印)"]
```

1. **填寫基本資料**：設定表單中文名稱、Form ID（如 `MaGpTrans`）與需求背景摘要。
2. **視覺化排版**：新增業務區塊，挑選列樣板（單欄/雙欄/比例欄），點選空格放入元件。
3. **設定防呆與規則**：點擊畫布上的元件，設定標籤名稱、必填狀態、下拉選單選項或 ERP 開窗查詢來源。
4. **權限確認與匯出**：至「關卡權限矩陣」配置各簽核關卡欄位權限，點選頂部按鈕匯出 **JSON**、**Markdown** 或**列印簽核**。

---

## 🛠️ 技術架構 (Architecture)

- **核心技術**：純 HTML5 + Vanilla JavaScript (ES6+)
- **樣式框架**：[Tailwind CSS (CDN)](https://tailwindcss.com/)
- **圖示庫**：[FontAwesome 6 (CDN)](https://fontawesome.com/)
- **資料儲存**：瀏覽器客戶端記憶體 + `window.localStorage`
- **相容性**：支援現代主流瀏覽器（Google Chrome、Microsoft Edge、Mozilla Firefox、Apple Safari）。

---

## 📂 檔案使用說明

1. 將本專案 Clone 或直接下載單一檔案：
   ```bash
   git clone https://github.com/kentchuang/efgp-form-spec-tool.git
   ```
2. 在檔案總管中直接雙擊開啟 `Form_Specification_Interactive_Tool.html`（或 `index.html`）。
3. 開始規劃您的電子表單規格！

---

## 📄 授權條款 (License)

本專案採用 [MIT 授權條款](LICENSE) - 歡迎自由取用、修改、分發與整合至企業內部流程中。
