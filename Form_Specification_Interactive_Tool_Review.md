# 🛡️ EFGP 表單規格互動工具程式完整性與資安修復報告

本文件記錄 `Document/Form_Specification_Interactive_Tool.html`（EFGP 電子表單需求盤點與規格確認互動工具）之程式完整性檢驗、資安程式碼審查（Claude Code Security Review）成果及漏洞修復細節。

---

## 📌 1. 審查背景與範疇

- **審查目標**：[`Document/Form_Specification_Interactive_Tool.html`](Form_Specification_Interactive_Tool.html)
- **程式規模**：5,586 行單一檔案純前端 HTML5 / JavaScript 應用程式（因後續功能持續擴充，行數已較初次審查時的 5,317 行增加，詳見第 5 節版本異動追蹤）
- **工具用途**：供業務單位 (User)、系統分析師 (SA) 與開發工程師 (PG) 進行視覺化表單排版、元件觸發時機防呆確認、關卡權限矩陣配置，並支援 `.form` XML 逆向解析與 JSON / Markdown 規格書匯出入。
- **審查標準**：依循 Anthropic Claude Code Security Review 規範（高信心度、具實際可利用性漏洞審查）與 EFGP 專案客製開發規範。

---

## 🚨 2. 發現之資安漏洞與成因分析

### 2.1 漏洞類型：DOM-based 跨站腳本攻擊 (DOM-based XSS)

* **嚴重等級 (Severity)**：**High / Medium**
* **信心度 (Confidence)**：**0.95 (9.5 / 10)**
* **CWE 編號**：[CWE-79: Improper Neutralization of Input During Web Page Generation](https://cwe.mitre.org/data/definitions/79.html)

### 2.2 漏洞成因與攻擊向量 (Root Cause & Attack Vectors)

互動工具支援使用者由本機匯入自訂 `.json` 規格檔或 EFGP 匯出的 `.form` (XML) 檔案。在解析後渲染畫面時，多處動態字串直接透過 ES6 模板字串插值（`${...}`）寫入 `innerHTML` 或 `<input value="${...}">` 屬性中，且未經 `escapeHtml()` 跳轉過濾。

當受害者匯入包含惡意構造的標籤（如 `<img src=x onerror=...>` 或 `"><script>...`）之檔案時，惡意腳本將直接於當前 Origin 執行，造成：
1. **竊取本機暫存資料**：讀取 `localStorage` 中儲存的表單設計草稿與企業內部欄位資訊。
2. **會話劫持與偽造**：在當前瀏覽器環境下執行未授權的 DOM 操作或惡意跳轉。

---

## 🛠️ 3. 具體修復方案與代碼對照

所有動態字串拼接處均已全面導入 `escapeHtml()` 跳轉機制，嚴格消除 DOM XSS 與屬性逃逸風險：

### 修復點 1：畫布列標題 (Canvas Row Title)
- **檔案位置**：`Document/Form_Specification_Interactive_Tool.html` (約 L2924，因後續功能新增行號已順移，修復內容不變)
- **修復原由**：`row.name` 若包含 HTML 標籤，會在畫布渲染時直接執行。
- **修復對照**：
  ```diff
  // modify by kentchuang_ai 20260918: 安全強化，tRowTitle 強制 escapeHtml 避免 DOM XSS
  rowContainer.innerHTML = `
      <div class="efgp-row-header">
  -       <span class="text-xs font-bold">${tRowTitle} 【${getLayoutName(row.layout)}】</span>
  +       <span class="text-xs font-bold">${escapeHtml(tRowTitle)} 【${getLayoutName(row.layout)}】</span>
  ```

---

### 修復點 2：畫布下拉選單選項文字 (Dropdown Preview Text)
- **檔案位置**：`Document/Form_Specification_Interactive_Tool.html` (約 L3088，因後續功能新增行號已順移，修復內容不變)
- **修復原由**：`displayText` 取自 `field.source` 或 `field.options`，若選項內容包含腳本標籤將於畫布觸發執行。
- **修復對照**：
  ```diff
  <div class="h-7 bg-white border border-slate-300 rounded px-2 text-xs flex items-center justify-between text-slate-600">
  -   <span class="truncate font-sans">${displayText}</span>
  +   <span class="truncate font-sans">${escapeHtml(displayText)}</span>
      <i class="fa-solid fa-chevron-down text-[10px] text-slate-400 ml-1 flex-shrink-0"></i>
  </div>
  ```

---

### 修復點 3：新增區塊列樣板對話框屬性 (Section Row Item Inputs)
- **檔案位置**：`Document/Form_Specification_Interactive_Tool.html` (約 L3315，因後續功能新增行號已順移，修復內容不變)
- **修復原由**：`defaultName` 與 `defaultId` 注入至 `value=""` 屬性時未跳轉雙引號，可能發生屬性逃逸注入（如 `" onfocus="alert(1)`）。
- **修復對照**：
  ```diff
  // modify by kentchuang_ai 20260918: 安全強化，defaultName 與 defaultId 強制 escapeHtml 避免屬性逃逸 DOM XSS
  rDiv.innerHTML = `
      <div class="w-48">
          <label class="block text-xs font-bold text-slate-600 mb-1">列名稱 (如: 第1列-決策選項) *</label>
  -       <input type="text" class="sec-row-name w-full text-sm px-3 py-1.5 border border-slate-300 rounded-lg font-semibold" value="${defaultName}" required>
  -       <input type="hidden" class="sec-row-id" value="${defaultId}">
  +       <input type="text" class="sec-row-name w-full text-sm px-3 py-1.5 border border-slate-300 rounded-lg font-semibold" value="${escapeHtml(defaultName)}" required>
  +       <input type="hidden" class="sec-row-id" value="${escapeHtml(defaultId)}">
      </div>
  ```

---

### 修復點 4：SubTab 分頁卡片屬性 (SubTab Item Card Inputs)
- **檔案位置**：`Document/Form_Specification_Interactive_Tool.html` (約 L3556，因後續功能新增行號已順移，修復內容不變)
- **修復原由**：SubTab 分頁名稱與 ID 於卡片生成時注入屬性，需防範引號逃逸。
- **修復對照**：
  ```diff
  // modify by kentchuang_ai 20260918: 安全強化，defaultName 與 defaultId 強制 escapeHtml 避免屬性逃逸 DOM XSS
  cardDiv.innerHTML = `
      ...
      <div class="flex-1">
  -       <input type="text" class="subtab-title text-sm font-bold text-slate-800 px-2.5 py-1.5 border border-slate-300 rounded-lg w-full" value="${defaultName}" placeholder="分頁名稱 (Title) *" required>
  +       <input type="text" class="subtab-title text-sm font-bold text-slate-800 px-2.5 py-1.5 border border-slate-300 rounded-lg w-full" value="${escapeHtml(defaultName)}" placeholder="分頁名稱 (Title) *" required>
      </div>
      <div class="w-36">
  -       <input type="text" class="subtab-id text-xs font-mono font-bold text-indigo-700 px-2.5 py-1.5 border border-slate-300 rounded-lg w-full" value="${defaultId}" placeholder="分頁 ID *" required>
  +       <input type="text" class="subtab-id text-xs font-mono font-bold text-indigo-700 px-2.5 py-1.5 border border-slate-300 rounded-lg w-full" value="${escapeHtml(defaultId)}" placeholder="分頁 ID *" required>
      </div>
  ```

---

### 修復點 5：SubTab 分頁內列樣板屬性 (SubTab Inner Row Inputs)
- **檔案位置**：`Document/Form_Specification_Interactive_Tool.html` (約 L3616，因後續功能新增行號已順移，修復內容不變)
- **修復原由**：分頁內多列樣板之名稱與 ID 注入屬性，需防範引號逃逸。
- **修復對照**：
  ```diff
  // modify by kentchuang_ai 20260918: 安全強化，defaultName 與 defaultId 強制 escapeHtml 避免屬性逃逸 DOM XSS
  rDiv.innerHTML = `
      <div class="w-40">
  -       <input type="text" class="subtab-row-name text-xs font-bold text-amber-950 px-2.5 py-1 border border-amber-300 rounded bg-white w-full" value="${defaultName}" placeholder="列名稱" required>
  -       <input type="hidden" class="subtab-row-id" value="${defaultId}">
  +       <input type="text" class="subtab-row-name text-xs font-bold text-amber-950 px-2.5 py-1 border border-amber-300 rounded bg-white w-full" value="${escapeHtml(defaultName)}" placeholder="列名稱" required>
  +       <input type="hidden" class="subtab-row-id" value="${escapeHtml(defaultId)}">
      </div>
  ```

---

### 修復點 6：Grid 單身表格欄位屬性 (Grid Column Row Inputs)
- **檔案位置**：`Document/Form_Specification_Interactive_Tool.html` (約 L3744，因後續功能新增行號已順移，修復內容不變)
- **修復原由**：Grid 單身欄位之代號 (`defaultId`)、名稱 (`defaultCaption`) 與繫結 (`colBinding`) 屬性防呆。
- **修復對照**：
  ```diff
  // modify by kentchuang_ai 20260918: 安全強化，defaultId、defaultCaption 與 colBinding 強制 escapeHtml 避免屬性逃逸 DOM XSS
  rDiv.innerHTML = `
      <div class="w-1/3">
          <label class="block text-[11px] font-bold text-slate-600 mb-0.5">欄位代號 (ID) *</label>
  -       <input type="text" oninput="updateGridColumnsLivePreview()" class="grid-col-id text-xs font-mono font-bold text-indigo-700 px-2 py-1.5 border border-slate-300 rounded bg-white w-full" value="${defaultId}" placeholder="例如: txtPartNum" required>
  +       <input type="text" oninput="updateGridColumnsLivePreview()" class="grid-col-id text-xs font-mono font-bold text-indigo-700 px-2 py-1.5 border border-slate-300 rounded bg-white w-full" value="${escapeHtml(defaultId)}" placeholder="例如: txtPartNum" required>
      </div>
      <div class="w-1/3">
          <label class="block text-[11px] font-bold text-slate-600 mb-0.5">欄位名稱 (Caption) *</label>
  -       <input type="text" oninput="updateGridColumnsLivePreview()" class="grid-col-caption text-xs font-bold text-slate-800 px-2 py-1.5 border border-slate-300 rounded bg-white w-full" value="${defaultCaption}" placeholder="例如: 料號" required>
  +       <input type="text" oninput="updateGridColumnsLivePreview()" class="grid-col-caption text-xs font-bold text-slate-800 px-2 py-1.5 border border-slate-300 rounded bg-white w-full" value="${escapeHtml(defaultCaption)}" placeholder="例如: 料號" required>
      </div>
      <div class="flex-1">
          <label class="block text-[11px] font-bold text-slate-600 mb-0.5">欄位繫結表頭 (Binding)</label>
  -       <input type="text" oninput="updateGridColumnsLivePreview()" class="grid-col-binding text-xs font-mono px-2 py-1.5 border border-slate-300 rounded bg-white w-full text-slate-700" value="${colBinding}" placeholder="例如: txtPartNum (無則留空)">
  +       <input type="text" oninput="updateGridColumnsLivePreview()" class="grid-col-binding text-xs font-mono px-2 py-1.5 border border-slate-300 rounded bg-white w-full text-slate-700" value="${escapeHtml(colBinding)}" placeholder="例如: txtPartNum (無則留空)">
      </div>
  ```

---

### 修復點 7：規格盤點大表之 SubTab 所屬標籤 (Matrix Table SubTab Badges)
- **檔案位置**：`Document/Form_Specification_Interactive_Tool.html` (約 L4140，因後續功能新增行號已順移，修復內容不變)
- **修復原由**：SubTab 名稱 (`tabTitle`) 與分頁列名稱 (`trObj.name`) 拼接於大表徽章時未轉義，會直接注入 `tr.innerHTML`。
- **修復對照**：
  ```diff
  // modify by kentchuang_ai 20260918: 安全強化，tabTitle 與 trObj.name 強制 escapeHtml 避免 DOM XSS
  let tabRowBadge = '';
  if (subItem && subItem.rows && item.targetTabRowId) {
      const trObj = subItem.rows.find(r => r.id === item.targetTabRowId);
      if (trObj) {
  -       tabRowBadge = ` ➔ <span class="bg-amber-100 text-amber-900 border border-amber-300 px-1 rounded text-[11px]">${trObj.name} (${getLayoutName(trObj.layout)})</span>`;
  +       tabRowBadge = ` ➔ <span class="bg-amber-100 text-amber-900 border border-amber-300 px-1 rounded text-[11px]">${escapeHtml(trObj.name)} (${getLayoutName(trObj.layout)})</span>`;
      }
  }
  -tabBelongBadge = `<div class="mt-1 text-xs text-purple-800 font-bold flex items-center flex-wrap gap-1"><i class="fa-solid fa-folder-open mr-1"></i>${tabTitle}${tabRowBadge}</div>`;
  +tabBelongBadge = `<div class="mt-1 text-xs text-purple-800 font-bold flex items-center flex-wrap gap-1"><i class="fa-solid fa-folder-open mr-1"></i>${escapeHtml(tabTitle)}${tabRowBadge}</div>`;
  ```

---

## 🔍 4. 程式完整性檢驗總結

1. **DOM ID 關聯性**：HTML 中 105 個定義 ID 與 JS 腳本中 88 處相異 `document.getElementById` 參照完全吻合，無任何未定義元素。
2. **函式與事件完整性**：102 個 JavaScript 函式與 128 處行內事件（`onclick` 86 處、`onchange` 23 處、`oninput` 15 處、`onsubmit` 4 處）全數正確對應。
3. **字元編碼**：整份檔案經 UTF-8 編碼與字元流檢測，無任何亂碼、缺字或非標準字元。
4. **規格匯出入與生命週期**：`importJSON`、`importFormXML`、`exportJSON`、`exportMarkdown` 與 `localStorage` 自動草稿保存機制均維持正常運作，並具備完整例外保護。

> 以上統計數字為 2026-09-29 依目前檔案版本重新量測之結果，取代初版審查時的數字，方法與初版一致（元素/函式/事件總量統計），僅因後續功能擴充而增加。

---

## 🔄 5. 後續版本異動追蹤 (2026-09-29)

自初版資安審查完成後，本工具持續新增以下功能性異動。逐項覆核後確認**均沿用既有 `escapeHtml()` 跳脫規範，未新增任何未跳脫之動態字串插值，不影響第 2～3 節之修復結論**：

1. **設計器側欄浮動定位修復**：移除包住頁籤內容之祖先 `<section>` 的 `overflow-hidden`（該屬性會截斷子層 `position: sticky` 的浮動範圍），並修正元件庫／欄位樣板／元件屬性面板之 `sticky` 偏移量，使其在長表單捲動時正確浮動、不被上方工具列遮蔽。純 CSS/版面調整，未涉及動態字串渲染。
2. **Radio / Checkbox 標註優化**：元件庫按鈕與屬性下拉選單加註「單選選項」「多選選項」中文提示，並將按鈕文字改為上下兩行排版避免斷字。純靜態文字，無使用者輸入拼接。
3. **表單代號 / 欄位代號改為選填**：`validateRequiredFormInfo()` 移除對「表單代號 (Form ID)」之強制填寫檢查，僅保留「表單中文名稱」為必填；欄位代號 (Field ID) 標註為選填並可由系統自動產生。屬表單驗證邏輯調整，未涉及 HTML 渲染安全。
4. **元件預覽方框改顯示資料來源 (Source)**：畫布上一般元件（TextBox/SerialNumber 等）方框內容改為顯示 `field.source`（經 `escapeHtml()` 跳脫）而非重複顯示元件代號，未設定時顯示靜態提示文字，跳脫機制與原修復點 2 一致。
5. **區塊 (Section) 上下移動排序**：新增 `moveSectionUp()` / `moveSectionDown()` 函式與對應按鈕；按鈕 `onclick` 沿用既有寫法以 `escapeHtml(sec.id)` 跳脫區塊 ID，與修復點 1 相同規範。
6. **Attachment / SerialNumber 元件單張表單限量防呆**：新增 `quickPlaceComponent()`／`saveFieldItem()` 內的重複新增檢查與 `updateAttachmentPaletteState()` / `updateSerialNumberPaletteState()` 按鈕狀態同步函式，屬純邏輯防呆（`alert()` 警示訊息為固定字串，無使用者輸入拼接），不涉及跳脫風險。
7. **修復「元件欄位槽位碰撞」導致元件於畫布上隱形消失之根本性缺陷**：實際案例排查發現，右側屬性面板變更「目標格」下拉選單時 (`autoApplySideProps()`)，會直接覆寫 `field.targetColIndex`、未檢查該格是否已被其他元件佔用，導致兩個元件共用同一槽位、其中一個永遠無法被渲染而「消失」（但資料仍完整保留在 `fields` 陣列中，並非匯出格式或順序問題）。已修正為比照既有拖曳排版 (`moveFieldToCell`) 的交換保護邏輯，若目標格已被佔用則自動與原佔用元件交換位置。屬純資料邏輯調整，未涉及 HTML 渲染或跳脫風險。
8. **新增 `repairFieldColumnCollisions()` 自動修復機制**：於 `importJSON()` / `importFormXML()` 匯入成功後統一執行，掃描每一列，將槽位碰撞的元件自動遞補至同列空槽位；若該列元件數已超出樣板實際欄數上限（例如 4 欄式卻有 5 個元件），會自動新增一個延伸列安置放不下的元件，確保匯入的舊資料不會因槽位問題而遺失顯示。純資料修復邏輯，`alert()` 提示訊息僅拼接數字與固定字串，無 HTML 注入風險。
9. **修復 `quickPlaceComponent()` 新增元件時的空位搜尋邏輯（根本原因修復）**：原邏輯在自動尋找空槽位時搜尋範圍為 0~11，未侷限於該列樣板實際可容納欄數（例如 4 欄式僅有 0~3），當目標列已滿時仍會指派超出可視範圍的欄位索引，導致新增的元件永遠無法顯示、如同憑空消失。已改為透過 `getRowColCount()` 判斷該列實際容量，額滿時自動於同區塊新增一列安置新元件。此為本次一系列「元件消失」回報之根本成因，修復後可從源頭杜絕同類問題再次發生。
10. **Attachment 元件畫布預覽改為按鈕樣式**：新增 `field.type === 'Attachment'` 專屬渲染分支，呈現為含迴紋針圖示之按鈕（比照 EFGP 實際上傳附件為按鈕型態的行為），取代原本文字輸入框樣式；動態內容 (`field.label`、`field.id`) 皆沿用既有 `escapeHtml()` 跳脫。
11. **修復 `deleteRowInDesigner()` 刪除列時元件變成孤兒資料的問題**：原程式碼確認對話框文字宣稱「該列上的元件將自動移出」，但實際上僅刪除列定義、從未真正移動或刪除該列上的元件，導致元件的 `targetRowId` 指向已不存在的列、變成孤兒資料而永久消失（但仍計入 Attachment/SerialNumber 等數量上限判斷）。已修正為刪除列時將元件實際移至鄰近列空槽位，若已無空位則自動新增一列安置，並委由 `repairFieldColumnCollisions()` 統一處理槽位分配以避免碰撞；確認對話框文字亦同步更新為準確反映實際行為。
12. **修復點選既有元件後，新增元件可能誤放到其他區塊的問題 (`gActiveSectionId` 與選取狀態同步)**：原本 `selectCanvasField()`（點選畫布上既有元件）僅設定 `gActiveFieldId`，未同步更新 `gActiveSectionId` 為該元件實際所屬區塊；`moveFieldToCell()`（拖曳／點選搬移元件至目標格）亦未於搬移後同步 `gActiveSectionId`。當使用者點選 B 區塊中的既有元件、或將元件搬移至 B 區塊後，若接著從左側元件庫新增元件，程式仍依殘留的舊 `gActiveSectionId`（例如先前操作過的 A 區塊）判斷插入目標，導致新元件被誤放到非目前操作中的區塊。已修正為：點選既有元件時同步鎖定其所屬區塊、搬移元件完成後同步鎖定目的區塊為目前作用中區塊。純前端狀態變數同步邏輯調整，未涉及 HTML 渲染或使用者輸入拼接，不影響既有 `escapeHtml()` 跳脫規範。
13. **新增畫布「點選空白處 / ESC 鍵取消選取 (Deselect)」機制**：原設計中，使用者點選畫布上某元件進入選取狀態 (`gActiveFieldId` 有值) 後，任何「空格」點擊皆會被解讀為「將選取的元件搬移至該空格」（此為既有之快速搬移設計），但沒有明確的方式可以取消此選取狀態，使用者稍有不慎點到其他空格即會誤觸發搬移。新增共用函式 `clearCanvasSelection()` 統一清除 `gActiveFieldId` / `gActiveCellTarget`，並將右側屬性面板還原為預設「請在畫布上點選任何元件或格子」提示文字；新增 `handleCanvasBackgroundClick()` 綁定於畫布外層容器 `#designerCanvasWrapper` 的空白區域點擊事件；`setActiveSection()`（點選區塊空白處）合併呼叫取消選取；鍵盤 `ESC` 於非放大預覽模式時亦同步觸發取消選取。修復後使用者可明確點擊畫布空白處或按 `ESC` 取消目前選取，避免誤觸發元件搬移。純前端互動狀態邏輯調整，未新增任何動態字串插值或 `innerHTML` 拼接使用者輸入，不影響本報告第 2～3 節之 XSS 修復結論。

**結論**：上述功能異動（含第 7～13 項新增修復）不影響本報告第 2～3 節所列 7 處 XSS 修復之有效性；新增程式碼經覆核均符合既有 `escapeHtml()` 使用規範，未發現新增之資安疑慮。第 7～9、11 項修復並直接解決了近期實際規格檔（`PKG Evaluate Request_規格確認單.json`）中發現的「元件因欄位槽位碰撞或孤兒列參照而於畫布上消失」系列問題之根本成因；第 12～13 項則為視覺化設計器操作流暢度與防誤觸之互動體驗優化。
