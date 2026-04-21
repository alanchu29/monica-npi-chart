# Monica NPI Project Chart

Monica NPI 專案排程看板（單頁版），用甘特圖方式顯示各專案里程碑，並透過 Google Apps Script 連動雲端資料。

## 專案特色

- 單一 `index.html` 即可運作，無需打包流程。
- 以日期軸呈現專案階段（Development / Pre-golden / Golden FPY rack）。
- 支援 `Gen`、`Site`、`PIC` 篩選，快速聚焦專案。
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

1. 進入頁面後會自動向雲端拉取資料。
2. 使用上方篩選列切換 `GEN / SITE / PIC`。
3. 點 `OPEN EDIT MODE`，輸入密碼進入編輯模式。
4. 點 `New Project` 可新增專案與多個 phase。
5. 編輯完成後按 `Sync Update`，資料會送回雲端並重新同步。

## 資料欄位格式

每筆 phase 會以物件形式儲存，主要欄位如下：

- `project`: 專案名稱
- `category`: 分類（例如 `Gen10`、`Gen11`）
- `pic`: 負責人（可用 `/`、`,`、`，`、`&` 分隔多人）
- `site`: 廠區（例如 `MX`、`LZ`）
- `task`: 工作描述
- `type`: 階段類型（`Development` / `PreGolden` / `GoldenFPY`）
- `start`: 開始日期（`YYYY-MM-DD`）
- `end`: 結束日期（`YYYY-MM-DD`）

## 重要設定（目前寫在 `index.html`）

以下設定在程式中為常數，若要調整請直接修改 `index.html`：

- `LOCKED_GAS_URL`：Google Apps Script API 端點
- `EDIT_PASSWORD`：編輯模式密碼
- `TODAY_INDICATOR`：時間軸中的「今日」標記日期
- `DEFAULT_CATEGORIES` / `DEFAULT_SITES` / `DEFAULT_PICS`：預設篩選項目

## 安全與維護建議

- 目前密碼與 API URL 是前端明碼，建議改為後端驗證或至少改為環境化配置。
- 若需多人共同維護，建議加上版本控管流程（分支、PR、審查）。
- 若資料結構擴充，請同步更新雲端 Script 端欄位處理邏輯。