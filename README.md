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
2. 點右上角 **Settings** 預設會開啟 **Display** 分頁；可切換 **Series & Site**、**Admin**、**GAS URL**。主畫面右上角 **分享連結** 可一鍵複製含 `?gas=` 的網址（與 Settings → GAS URL 內功能相同）。
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
- `type`: 階段類型（`Development` / `PreGolden` / `GoldenFPY`）
- `start`: 開始日期（`YYYY-MM-DD`）
- `end`: 結束日期（`YYYY-MM-DD`）

## 重要設定

- **GAS Web App 網址**：存於 `localStorage` 鍵 `monica-npi-gas-webapp-url`。若尚未儲存過網址（且未使用網址列 `?gas=` 參數），首次進入頁面會**自動開啟 Settings 並停留在 GAS URL 分頁**，須貼上 `/exec` 網址並按「儲存並連線」，或按「還原內建預設網址」使用程式內建的 `DEFAULT_GAS_URL`。在 **Settings → GAS URL** 可一鍵複製「含 `?gas=` 的分享連結」，方便把同一支 Web App 交給他人開啟（對方瀏覽器會自動帶入該網址）。
- **編輯模式密碼**：在 **Settings → Admin** 設定（存於 `localStorage`）；程式內建預設為 `DEFAULT_EDIT_PASSWORD`。
- **「今日」紅線**：一律依 **台北時區（Asia/Taipei）當日**，無需手動設定日期。
- **可視日期範圍**：在 **Settings → Display** 設定「今日之前／之後」各顯示幾天（存於 `localStorage`）；內建預設為前 7 天、後 90 天（見 `DEFAULT_VISIBLE_DAYS_BEFORE` / `DEFAULT_VISIBLE_DAYS_AFTER`）。
- **Series / Site 候選清單**：在 **Settings → Series & Site** 管理（存於 `localStorage` 的 `monica-npi-admin-prefs`）；內建預設見 `INITIAL_SERIES_LIST` / `INITIAL_SITE_LIST`。篩選選項會與試算表資料中已出現的 `category` / `site` 合併。**PIC** 候選仍由 `DEFAULT_PICS` 與資料中的 `pic` 合併。

## 安全與維護建議

- 目前密碼與 API URL 是前端明碼，建議改為後端驗證或至少改為環境化配置。
- 若需多人共同維護，建議加上版本控管流程（分支、PR、審查）。
- 若資料結構擴充，請同步更新雲端 Script 端欄位處理邏輯。