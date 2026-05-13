# Monica NPI Project Chart

Monica NPI 專案排程看板（單頁版），用甘特圖方式顯示各專案里程碑，並透過 Google Apps Script 連動雲端資料。

## 專案特色

- 單一 `index.html` 即可運作，無需打包流程。
- 以日期軸呈現專案階段（Development / Pre-golden / Golden FPY rack）。
- 支援 **Series**（試算表欄位 `category`）、**Site**、`PIC` 篩選，快速聚焦專案。
- 支援編輯模式（密碼驗證）新增、更新、刪除專案與 phase。
- 透過 Google Apps Script API 讀寫資料，頁面可直接同步雲端。

## 技術與相依

- React 18（CDN UMD）
- ReactDOM 18（CDN UMD）
- Babel Standalone（瀏覽器端 JSX 轉譯）
- Tailwind CSS（CDN）

> 本專案屬於前端靜態頁面，無 `package.json`、無 Node.js 安裝需求。

## 快速開始

### 1) 直接開啟

用瀏覽器開啟 `index.html` 即可使用。

### 2) 建議方式（避免 CORS / 快取問題）

用本機靜態伺服器啟動，例如：

```bash
python -m http.server 5500
```

然後開啟：`http://localhost:5500`

## 使用流程

1. 進入頁面後會自動向雲端拉取資料（須已設定 GAS URL；若首次使用未儲存網址，會先跳出設定並請完成 **GAS URL**）。
2. 點右上角 **Settings** 預設會開啟 **Display** 分頁；可切換 **Series**、**Phase**、**Site**、**Admin**、**GAS URL**。主畫面右上角 **分享連結** 可一鍵複製含 `?gas=` 的網址（與 Settings → GAS URL 內功能相同）。
3. 使用上方篩選列切換 **SERIES / SITE / PIC**。
4. 開啟 **Settings → Admin**，輸入密碼後按 **Enable edit mode** 進入編輯模式。
5. 點 `New Project` 可新增專案與多個 phase。
6. 編輯完成後按 `Sync Update`，資料會送回雲端並重新同步。

## 資料欄位格式

每筆 phase 會以物件形式儲存，主要欄位如下：

- `project`: 專案名稱
- `category`: 分類／Series（UI 顯示為 Series；值可自訂，例如產品代號）
- `pic`: 負責人（可用 `/`、`,`、`，`、`&` 分隔多人）
- `site`: 廠區（例如 `MX`、`LZ`）
- `task`: 工作描述
- `type`: 階段類型（即 Phase 的 key；內建有 `DevelopmentTNRS` / `DevelopmentPI` / `PreGolden` / `GoldenFPY` / `VB`，亦可在 Settings 自訂新增）
- `start`: 開始日期（`YYYY-MM-DD`）
- `end`: 結束日期（`YYYY-MM-DD`）

## 重要設定

- **GAS Web App 網址**：存於 `localStorage` 鍵 `monica-npi-gas-webapp-url`。若尚未儲存過網址（且未使用網址列 `?gas=` 參數），首次進入頁面會**自動開啟 Settings 並停留在 GAS URL 分頁**，須貼上 `/exec` 網址並按「儲存並連線」，或按「還原內建預設網址」使用程式內建的 `DEFAULT_GAS_URL`。在 **Settings → GAS URL** 可一鍵複製「含 `?gas=` 的分享連結」，方便把同一支 Web App 交給他人開啟（對方瀏覽器會自動帶入該網址）。
- **編輯模式密碼**：**已改為雲端共用**，存於 GAS `DocumentProperties.editPassword`，所有使用同一支 Web App 的人共用一份；**不再存於 localStorage**。
  - **預設無密碼**：新部署的 GAS 上沒有設定 `editPassword` 屬性 → 任何人點 **Settings → Admin → Enable edit mode** 即可直接解鎖（不需輸入）。
  - 設定 / 變更：**Settings → Admin** 在「設定 / 變更雲端密碼」輸入新值並按「儲存到雲端」（前端 POST `{ action: "setEditPassword", password }`）。
  - 清除：清空輸入框後按「儲存到雲端」即可移除密碼，回到預設「人人可解鎖」狀態。
  - **本機備份遷移**：若本機 `localStorage` 內有舊版 `editPassword`，首次連到新版 GAS 時會嘗試自動上傳；上傳成功（下次重整見雲端有密碼）後本機備份才會清除。
  - **GAS 端需**：`doGet(?meta=1)` 回傳 `editPassword`、`doPost(e)` 支援 `setEditPassword` 分支；範例見 `.git/App.gs.js`。修改後請**重新部署 Web App**。
  - **安全提醒**：此密碼以明文存於 DocumentProperties，僅是前端「軟性閘門」，**不是真正的存取控制**（任何人拿到 GAS URL 都能 GET 拿到資料與密碼）。需要真正權限請靠 Web App 部署設定（限定特定 Google 帳號）。
- **「今日」紅線**：一律依 **台北時區（Asia/Taipei）當日**，無需手動設定日期。
- **可視日期範圍**：在 **Settings → Display** 設定「今日之前／之後」各顯示幾天（存於 `localStorage`）；內建預設為前 7 天、後 90 天（見 `DEFAULT_VISIBLE_DAYS_BEFORE` / `DEFAULT_VISIBLE_DAYS_AFTER`）。
- **甘特圖標題**：標題就是 **Google 試算表的檔案名稱**（單一來源，不存本機）。
  - 進站讀取：透過 GAS `?meta=1` 取得 `ss.getName()`，所有開啟同一支 GAS 的人都會看到一致的標題。
  - 在 **Settings → Display** 修改並按「儲存標題」：前端 POST `{ action: "setTitle", title }`，GAS 端呼叫 `ss.rename(...)` 重新命名整本試算表；其他使用者重新整理後即會同步。
  - GAS 端需在 `doGet(e)` 加 `?meta=1` 分支、在 `doPost(e)` 加 `setTitle` 分支；範例見 `.git/App.gs.js`。修改後請**重新部署 Web App**；第一次重新命名可能需在 Apps Script 編輯器手動執行一次以完成 Drive 授權。
  - 取不到試算表名稱時（例如 GAS 尚未部署支援 `?meta=1` 的版本）使用後備預設 `DEFAULT_CHART_TITLE`。
- **Series / Phase / Site 候選清單與顏色**：**已改為雲端共用**，存於 GAS 端 `PropertiesService.getDocumentProperties()`，鍵名 `labels`；**不再存於 localStorage**。所有開啟同一支 GAS 的使用者重新整理後都會看到一致的清單與顏色。
  - **Series**：**Settings → Series** 新增 / 刪除，並可逐項點色塊調整 Series 標籤顯示顏色（會即時反映在主畫面的專案列）。內建預設見 `INITIAL_SERIES_LIST` 與 `INITIAL_SERIES_COLORS`，新增 Series 時會依 `SERIES_COLOR_PALETTE` 自動分配顏色。
  - **Phase**：**Settings → Phase** 新增 / 刪除 / 重新命名，並可逐項調整顯示顏色（Gantt 色塊與 Legend 的顏色）。內建預設見 `INITIAL_PHASE_LIST`。Phase 的內部 `key`（試算表 `type` 欄位儲存的值）會依名稱自動產生，避免與既有資料衝突。
  - **Site**：**Settings → Site** 新增 / 刪除，內建預設見 `INITIAL_SITE_LIST`。
  - 進站讀取：透過 GAS `?meta=1` 同時取得 `labels` 物件（`{ seriesList, siteList, seriesColors, phaseList }`）。任何在 UI 上的新增/刪除/改顏色，前端會立即 POST `{ action: "setLabels", labels }`，GAS 端寫入 `PropertiesService` 並廣播給其他使用者（對方重新整理可見）。
  - **GAS 端需在 `doGet(e)` 擴充 `?meta=1` 回傳 `labels`、在 `doPost(e)` 加 `setLabels` 分支**；完整範例見 `.git/App.gs.js`。修改後請**重新部署 Web App**（並可能需在 Apps Script 編輯器手動執行一次完成授權）。
  - **本機備份遷移**：若在啟用雲端化之前曾於本機累積過 Series/Phase/Site 設定，首次連到新版 GAS 時會嘗試自動把本機設定**一次性**上傳到雲端；上傳成功後（下次重整見雲端有資料）本機備份才會被清除。
  - 篩選選項會與試算表資料中已出現的 `category` / `site` 合併；**PIC** 候選仍由 `DEFAULT_PICS` 與資料中的 `pic` 合併。
  - **Resource Check** 仰賴 `DevelopmentTNRS` 與 `DevelopmentPI` 兩個 key；若於 Settings 刪除這兩個 Phase，超載偵測將不會觸發（其他 Phase 不影響）。
  - 取不到 `labels`（例如 GAS 尚未部署支援 `?meta=1` 新版本）→ 顯示「正在從 Google 試算表載入…」並暫時使用 `INITIAL_*` 內建預設，**不會破壞畫面**。

## 安全與維護建議

- 目前密碼與 API URL 是前端明碼，建議改為後端驗證或至少改為環境化配置。
- 若需多人共同維護，建議加上版本控管流程（分支、PR、審查）。
- 若資料結構擴充，請同步更新雲端 Script 端欄位處理邏輯。