# Monica NPI Project Chart

Monica NPI 專案排程看板（單頁版），用甘特圖方式顯示各專案里程碑，並透過 Google Apps Script 連動雲端資料。

- **單一 `index.html`** 即可運作，無打包流程。
- 以日期軸呈現專案階段（Development / Pre-golden / Golden FPY / V/B…，可自訂）。
- **Series / Site / PIC** 三層篩選，快速聚焦。
- 編輯模式可在主畫面 Header 一鍵切換（可選密碼保護）。
- 透過 Google Apps Script 讀寫 Google 試算表，多人共用同一支 Web App 即可共享資料。

---

## TL;DR – 從零到能用，總共要做的事

1. 建一張**全新空白**的 Google 試算表。
2. 在試算表 `擴充功能 → Apps Script` 貼上下方的 GAS 程式碼。
3. 部署為「網頁應用程式」並複製 `/exec` 結尾的 URL。
4. 打開 `index.html`，把 URL 貼進跳出來的 Settings → GAS URL。
5. 點 Header 的 `Enable edit mode` → 開 Settings 在 `Series` / `Phase` 各加一筆 → 關閉設定 → `New Project` → 填資料 → Save，完成。

> 完全不需要事先在試算表建欄位或 header；第一次儲存時 GAS 會自動建好 8 個欄位列。
>
> Series 與 Phase 預設**空清單**，要先在 Settings 內自己定義一份才能建立 Project；Site 則內建一組常用預設（`MX / LZ / MY / CZ / TN`），不滿意可在 Settings → Site 增刪。

---

## 全新使用者完整上手指南

下面以「我從來沒用過這個工具，也沒設定過任何 GAS」的角度，一步一步走完。

### Step 1 – 準備一張全新空白的 Google 試算表

1. 進入 [Google Drive](https://drive.google.com/)。
2. 左上角 `+ 新增` → `Google 試算表 → 空白試算表`。
3. 給它一個名字（例如 `Monica NPI 排程`）—— 這個名字之後**就是甘特圖的標題**，可以隨時改。
4. **保持完全空白**，不要手動建任何 header 或資料列。

> ⚠️ 重要：每次甘特圖儲存資料時，GAS 會先 `sheet.clear()` 整張清空再重寫。所以若把 GAS 綁到一個「裡面有其他資料的試算表」，那些資料會被清掉。**請務必用一張新空白試算表。**

### Step 2 – 安裝 Apps Script

1. 在剛建的試算表中：選單 `擴充功能 → Apps Script`。
2. 新分頁會打開 Apps Script 編輯器，預設有一個 `Code.gs` 檔，內含 `function myFunction() {}` 範例。
3. **把預設內容整個刪除**，貼上下方「[Apps Script 程式碼](#apps-script-程式碼)」整段。
4. 按 `Ctrl+S`（或左上角磁碟圖示）儲存。
5. 左上角專案標題目前是「未命名專案」，可改成例如 `Monica NPI API`。

### Step 3 – 部署為 Web App 並取得 URL

1. Apps Script 編輯器右上角 → `部署 → 新增部署作業`。
2. 左側齒輪 → 選 `網頁應用程式`。
3. 填寫：
   - **說明**：任意（例如 `v1`）。
   - **執行身分**：`我（你的 Google 帳號）`。
   - **誰可以存取**：選 `所有人`。
4. 按 `部署`。
5. **首次部署會跳出授權流程**：
   - `授權存取權` → 選擇你的 Google 帳號。
   - 看到「Google 尚未驗證這個應用程式」警告 → 點左下角 `進階` → `前往「(你的專案名稱)」（不安全）`。
   - 在後續畫面按 `允許`，把試算表讀寫權限授予給這支腳本。
6. 部署成功後會跳出對話框，複製其中的「**網頁應用程式網址**」（結尾為 `/exec`）。

> 之後若修改 `App.gs.js`，記得回 `部署 → 管理部署作業 → 編輯（鉛筆）→ 新版本 → 部署`，**否則前端拿到的還是舊版**。

### Step 4 – 開啟甘特圖頁面並貼上 URL

開啟後：

1. 因為還沒有 GAS URL，頁面會**自動跳出 Settings 並停在 `GAS URL` 分頁**。
2. 把 Step 3 複製的 `/exec` 網址貼進輸入框。
3. 可先按 `Test connection` 確認 OK，再按 `儲存並連線`。

> ⚠️ **重要：儲存後請立刻把目前網址加到書籤**
>
> 儲存後瀏覽器網址列會自動加上 `?gas=…`，這就是<strong>下次開啟此甘特圖的入口</strong>。
> **沒有 `?gas=` 參數的網址會被視為初次使用者**，下次又得重新貼一次 URL。
>
> 要分享給同事 → 用 Header 上的「分享連結」按鈕直接複製含 `?gas=` 的網址。
4. 關閉 Settings。畫面開始向你的試算表抓資料（一開始當然是空的）。

### Step 5 – 啟用編輯模式

預設**沒有密碼**，所以可以直接操作：

1. Header 右上角應該會看到綠色 `Enable edit mode` 按鈕 → 點下去（如果之前在 Admin 設過密碼會跳密碼框）。
2. 此時 Header 也會出現 `Settings`、`Finish editing` 等按鈕都解鎖。

### Step 6 – 先到 Settings 建立你自己的 Series 與 Phase

新使用者進來時，**Series（產品系列）與 Phase（階段）兩個清單預設都是空的**。`+ New Project` 按鈕在這兩個任一為空時都會 disabled，要先補齊：

1. Header 點 `Settings`。
2. 切到 `Series` 分頁：
   - 在「新增 Series」輸入框打你的系列名稱（例如 `ProductA`、`Gen11`、`2026 Project`…），按 `新增`。
   - 至少加一筆。可隨時調色塊、改名、刪除。
3. 切到 `Phase` 分頁：
   - 同樣在「新增 Phase」輸入框打階段名稱（例如 `Design`、`Pre-golden`、`MP`…），可選顏色，按 `新增`。
   - 至少加一筆。系統會自動把你輸入的名稱轉成一個內部 `key`（不會跟既有資料衝突），實際試算表存的是這個 key。
4. （可選）切到 `Site` 分頁微調預設的 `MX / LZ / MY / CZ / TN`。
5. 關閉 Settings。

> Series 與 Phase **儲存在雲端**（GAS DocumentProperties），所以同一個 GAS Web App 下的所有使用者重新整理後都會看到一致的清單。

### Step 7 – 新增第一個 Project

1. 篩選列下方此時 `+ New Project` 按鈕變綠 → 點下去。
2. 在 Modal 填入：
   - `Project Name`：專案名稱。
   - `Series`：從你剛剛在 Settings 建立的下拉選單挑一筆。
   - `PIC`：負責人（可用 `/`、`,`、`，`、`&` 分隔多人）。
   - `Site`：廠區。
   - 至少一段 Phase：上面是 Task 描述、下面挑 Phase Type（顏色 chip）、再選 `Start` / `End` 日期；可按 `+ Add Phase` 多加。
3. 按 `Save`。

這個動作會發 POST 到你的 GAS Web App，GAS 收到後會：

- 先 `sheet.clear()` 把試算表整張清空。
- 自動寫入 header 列：`project | category | task | type | start | end | pic | site`。
- 再把這筆 project 的所有 phases 寫成資料列。

打開試算表你會看到 header + 第一筆資料。**這就是「不需要預先建 schema」的意思——第一次 Save 自動建好。**

### Step 8 – （可選）為這份甘特圖加密碼

如果你不想讓所有看到 URL 的人都能編輯：

1. Header 點 `Settings`。
2. 切到 `Admin` 分頁。
3. 「設定 / 變更雲端密碼」輸入你要的密碼 → `儲存到雲端`。
4. 之後其他人開啟頁面：點 `Settings` 或 `Enable edit mode` 都會跳密碼框。
5. 想取消密碼 → 同樣位置把輸入框**清空 → 儲存到雲端**，回到「人人可解鎖」狀態。

> 密碼是「軟性閘門」：只擋前端互動，**不是真權限**。拿到 GAS URL 的人仍可用 curl 直接 GET 到資料。要真正的權限請靠 Step 3 的 `誰可以存取` 設為 `任何 Google 帳號` 或更嚴。

### Step 9 – （可選）分享給其他人

兩種方式：

- **A. 同事自己設定**：把 `/exec` 網址給對方，請對方在自己瀏覽器的 Settings → GAS URL 貼入。
- **B. 一鍵分享連結**：Header 上的 `分享連結` 按鈕（也可在 Settings → GAS URL 找到）會複製當前頁面網址並附 `?gas=...` 參數。對方打開該連結，瀏覽器會自動帶入你的 GAS Web App，**不需要手動設定**。

> ⚠️ 分享連結若指向 `localhost`，只在你的電腦有效。要給外部使用者，請把 `index.html` 放到任何可公開存取的網址（GitHub Pages、Netlify、內部 web 伺服器…）後再產分享連結。

---

## Apps Script 程式碼

把以下整段貼到 Step 2 的 Apps Script 編輯器（取代預設的 `myFunction`）。本檔在 repo 內也存有一份 `App.gs.js` 供參考。

```javascript
/**
 * 處理 GET 請求：
 *   - 預設：從表格讀取專案資料（JSON 陣列）
 *   - ?meta=1：回傳試算表 metadata（甘特圖標題 + Series / Phase / Site / Resource Rules / Holidays 設定 + 編輯密碼）
 */
function doGet(e) {
  if (e && e.parameter && e.parameter.meta === '1') {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getActiveSheet();
    const props = PropertiesService.getDocumentProperties();

    let labels = null;
    try {
      const raw = props.getProperty('labels');
      if (raw) labels = JSON.parse(raw);
    } catch (err) {
      labels = null;
    }

    const editPasswordRaw = props.getProperty('editPassword');
    const editPassword = editPasswordRaw == null ? '' : String(editPasswordRaw);

    const payload = {
      title: ss.getName(),
      sheetName: sheet.getName(),
      labels: labels,
      editPassword: editPassword,
      updatedAt: new Date().toISOString()
    };
    return ContentService.createTextOutput(JSON.stringify(payload))
      .setMimeType(ContentService.MimeType.JSON);
  }

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = sheet.getDataRange().getValues();

  if (data.length <= 1) {
    return ContentService.createTextOutput(JSON.stringify([]))
      .setMimeType(ContentService.MimeType.JSON);
  }

  const headers = data.shift();
  const json = data.map(row => {
    const obj = {};
    headers.forEach((h, i) => {
      if (row[i] instanceof Date) {
        const d = row[i];
        const year = d.getFullYear();
        const month = String(d.getMonth() + 1).padStart(2, '0');
        const day = String(d.getDate()).padStart(2, '0');
        obj[h] = `${year}-${month}-${day}`;
      } else {
        obj[h] = row[i];
      }
    });
    return obj;
  });

  return ContentService.createTextOutput(JSON.stringify(json))
    .setMimeType(ContentService.MimeType.JSON);
}

/**
 * 處理 POST 請求：
 *   - { action: "setTitle", title }：重新命名整本試算表
 *   - { action: "setLabels", labels }：儲存 Series / Phase / Site / Resource Rules / Holidays 設定（DocumentProperties）
 *   - { action: "setEditPassword", password }：儲存編輯密碼（""/缺省 = 無密碼）
 *   - 陣列 [...]：高效能整塊寫入專案資料
 */
function doPost(e) {
  try {
    const body = JSON.parse(e.postData.contents);

    if (body && !Array.isArray(body) && body.action === 'setTitle') {
      const newTitle = String(body.title || '').replace(/[\r\n]+/g, ' ').trim();
      if (!newTitle) return ContentService.createTextOutput("Error: empty title");
      if (newTitle.length > 120) return ContentService.createTextOutput("Error: title too long (>120)");
      SpreadsheetApp.getActiveSpreadsheet().rename(newTitle);
      return ContentService.createTextOutput("Success: renamed to " + newTitle);
    }

    if (body && !Array.isArray(body) && body.action === 'setLabels') {
      const labels = body.labels;
      if (!labels || typeof labels !== 'object' || Array.isArray(labels)) {
        return ContentService.createTextOutput("Error: invalid labels payload");
      }
      const sanitized = sanitizeLabels_(labels);
      const json = JSON.stringify(sanitized);
      if (json.length > 9000) {
        return ContentService.createTextOutput("Error: labels payload too large");
      }
      PropertiesService.getDocumentProperties().setProperty('labels', json);
      return ContentService.createTextOutput("Success: labels saved");
    }

    if (body && !Array.isArray(body) && body.action === 'setEditPassword') {
      const props = PropertiesService.getDocumentProperties();
      const raw = body.password == null ? '' : String(body.password);
      const newPassword = raw.replace(/[\r\n]+/g, '').trim();
      if (newPassword.length > 200) {
        return ContentService.createTextOutput("Error: password too long (>200)");
      }
      if (newPassword === '') {
        props.deleteProperty('editPassword');
        return ContentService.createTextOutput("Success: edit password cleared");
      }
      props.setProperty('editPassword', newPassword);
      return ContentService.createTextOutput("Success: edit password saved");
    }

    const tasks = Array.isArray(body) ? body : [];
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    sheet.clear();
    if (tasks.length === 0) return ContentService.createTextOutput("Empty");

    const headers = ["project", "category", "task", "type", "start", "end", "pic", "site"];
    const values = [headers];
    tasks.forEach(t => {
      values.push([
        String(t.project || ""),
        String(t.category || ""),
        String(t.task || ""),
        String(t.type || ""),
        String(t.start || ""),
        String(t.end || ""),
        String(t.pic || ""),
        String(t.site || "")
      ]);
    });

    sheet.getRange(1, 1, values.length, headers.length).setValues(values);
    return ContentService.createTextOutput("Success");
  } catch (err) {
    return ContentService.createTextOutput("Error: " + err.toString());
  }
}

function sanitizeLabels_(input) {
  const out = {};

  if (Array.isArray(input.seriesList)) {
    const seen = {};
    out.seriesList = [];
    input.seriesList.forEach(function (v) {
      const s = String(v == null ? '' : v).replace(/[\r\n]+/g, ' ').trim();
      if (!s || s.length > 120 || seen[s]) return;
      seen[s] = true;
      out.seriesList.push(s);
    });
  }

  if (Array.isArray(input.siteList)) {
    const seen = {};
    out.siteList = [];
    input.siteList.forEach(function (v) {
      const s = String(v == null ? '' : v).replace(/[\r\n]+/g, ' ').trim();
      if (!s || s.length > 120 || seen[s]) return;
      seen[s] = true;
      out.siteList.push(s);
    });
  }

  if (input.seriesColors && typeof input.seriesColors === 'object' && !Array.isArray(input.seriesColors)) {
    out.seriesColors = {};
    Object.keys(input.seriesColors).forEach(function (name) {
      const k = String(name || '').trim();
      const c = String(input.seriesColors[name] || '').trim();
      if (!k || k.length > 120) return;
      if (!/^#[0-9a-fA-F]{3}$|^#[0-9a-fA-F]{6}$/.test(c)) return;
      out.seriesColors[k] = c.toLowerCase();
    });
  }

  if (Array.isArray(input.phaseList)) {
    const seen = {};
    out.phaseList = [];
    input.phaseList.forEach(function (p) {
      if (!p || typeof p !== 'object') return;
      const key = String(p.key || '').trim();
      const label = String(p.label || '').replace(/[\r\n]+/g, ' ').trim();
      const color = String(p.color || '').trim();
      if (!key || key.length > 60 || seen[key]) return;
      if (!label || label.length > 120) return;
      if (!/^#[0-9a-fA-F]{3}$|^#[0-9a-fA-F]{6}$/.test(color)) return;
      seen[key] = true;
      out.phaseList.push({ key: key, label: label, color: color.toLowerCase() });
    });
  }

  // resourceRules: [{ id, name, enabled, seriesScope, phaseKeys, maxConcurrent }]
  if (Array.isArray(input.resourceRules)) {
    const seenIds = {};
    out.resourceRules = [];
    input.resourceRules.forEach(function (r) {
      if (!r || typeof r !== 'object') return;
      const id = String(r.id || '').trim();
      if (!id || id.length > 60 || seenIds[id]) return;
      const name = String(r.name || '').replace(/[\r\n]+/g, ' ').trim();
      if (!name || name.length > 120) return;
      const enabled = r.enabled !== false;
      const dedupList = function (arr, maxLen) {
        const out = [];
        const seen = {};
        (Array.isArray(arr) ? arr : []).forEach(function (v) {
          const s = String(v == null ? '' : v).replace(/[\r\n]+/g, ' ').trim();
          if (!s || s.length > maxLen || seen[s]) return;
          seen[s] = true;
          out.push(s);
        });
        return out;
      };
      const seriesScope = dedupList(r.seriesScope, 120);
      const phaseKeys = dedupList(r.phaseKeys, 60);
      if (phaseKeys.length === 0) return;
      const maxNum = Number(r.maxConcurrent);
      if (!isFinite(maxNum) || maxNum < 0 || maxNum > 999) return;
      seenIds[id] = true;
      out.resourceRules.push({
        id: id,
        name: name,
        enabled: enabled,
        seriesScope: seriesScope,
        phaseKeys: phaseKeys,
        maxConcurrent: Math.floor(maxNum)
      });
    });
  }

  // holidays: [{ id, name, kind:'off'|'work', mode:'single'|'range'|'yearly', start?, end?, month?, day? }]
  if (Array.isArray(input.holidays)) {
    const seenHol = {};
    const isDateStr = function (s) {
      if (!/^\d{4}-\d{2}-\d{2}$/.test(s)) return false;
      const d = new Date(s + 'T12:00:00');
      return !isNaN(d.getTime());
    };
    out.holidays = [];
    input.holidays.forEach(function (h) {
      if (!h || typeof h !== 'object' || out.holidays.length >= 200) return;
      const id = String(h.id || '').trim();
      if (!id || id.length > 60 || seenHol[id]) return;
      let name = String(h.name || '').replace(/[\r\n]+/g, ' ').trim();
      if (!name) name = '未命名';
      if (name.length > 60) name = name.slice(0, 60);
      const kind = h.kind === 'work' ? 'work' : 'off';
      const mode = (h.mode === 'range' || h.mode === 'yearly') ? h.mode : 'single';

      let entry = null;
      if (mode === 'yearly') {
        const month = Number(h.month), day = Number(h.day);
        if (!(month >= 1 && month <= 12 && day >= 1 && day <= 31)) return;
        const probe = new Date(2024, month - 1, day, 12, 0, 0);
        if (probe.getMonth() !== month - 1 || probe.getDate() !== day) return;
        entry = { id: id, name: name, kind: kind, mode: mode, month: month, day: day };
      } else {
        const start = String(h.start || '').split('T')[0];
        if (!isDateStr(start)) return;
        if (mode === 'range') {
          const end = String(h.end || '').split('T')[0];
          if (!isDateStr(end) || end < start) return;
          const span = Math.round((new Date(end + 'T12:00:00') - new Date(start + 'T12:00:00')) / 86400000) + 1;
          if (span > 366) return;
          entry = { id: id, name: name, kind: kind, mode: mode, start: start, end: end };
        } else {
          entry = { id: id, name: name, kind: kind, mode: 'single', start: start };
        }
      }
      seenHol[id] = true;
      out.holidays.push(entry);
    });
  }

  return out;
}
```

---

## 資料欄位格式

每筆 phase 會以物件形式儲存到 Google Sheet 的一個列：

| 欄位 | 說明 |
|---|---|
| `project` | 專案名稱 |
| `category` | 分類 / Series（UI 顯示為 Series，值可自訂） |
| `pic` | 負責人；可用 `/`、`,`、`，`、`&` 分隔多人 |
| `site` | 廠區（例如 `MX`、`LZ`） |
| `task` | 工作描述 |
| `type` | 階段類型（Phase 的 key）。**新使用者預設無內建 Phase**，需在 Settings → Phase 自己建立。建立 Phase 時系統會依名稱自動生成 key（最多 32 字元，移除非英數字元），實際試算表 `type` 欄位儲存的就是這個 key |
| `start` | 開始日期 `YYYY-MM-DD` |
| `end` | 結束日期 `YYYY-MM-DD` |

---

## UI 速查

頁首採兩列結構，加上下方篩選列總共三排（手機 / 視窗較窄時會自動 wrap）：

- **第一列（標題列，深底）**：
  - 置中：同步狀態指示燈（綠 = 同步完成、橙色閃爍 = 同步中）+ 試算表標題 + `Gantt System Template · Designed by Dixon Chu`。標題本身已水平置中、左右各預留等寬空白讓視覺平衡。
  - 最右：`分享連結`（紫）/ `Settings`（天藍）。
- **第二列（篩選 + 動作列，稍淺底）**：
  - 左到右：`SERIES` / `SITE` / `PIC` chip 篩選；超寬時自動換行。
  - 最右：`Archived hidden / shown`（灰，僅在有專案被自動封存時出現）/ `Enable edit mode`（綠）或 `Finish editing`（橘）/ `Resource Check` 開關（OFF = 青；ON 有啟用規則 = 紅；ON 但無規則 = 琥珀，會提醒「請到 Settings → Resource 新增規則」）。
- **第三列（PHASE 圖例）**：每個 Phase 的顏色色塊 + label，方便對照 Gantt cell。
- **Gantt cell tooltip**：滑鼠移到任一 phase 色塊上會浮出詳細資訊 — Phase 名稱、起訖日、**工作天數**（綠色標籤，該 phase 區間扣掉假日後的天數）與「共 N 天，假日 M 天」、task 內容、所屬專案；若該格觸發 Resource Check 超載，還會列出命中的規則。工作天的假日定義與時間軸上的紅色日期一致（週末 + Settings → Holiday 自訂的假日，扣掉補班日）。
- **時間軸日期顏色**：假日顯示紅色（灰底），自訂假日另加紅色點狀底線；補班日顯示藍色加藍色點狀底線並計入工作天。滑鼠移到日期上會顯示該日的假日名稱。
- **Projects portfolio 表頭**：左側是 `Projects` 標題 + 模糊搜尋框（依專案名稱關鍵字過濾，例如輸入 `Gen10` 會找到所有名稱含 Gen10 的專案）；搜尋有值時所有 Series 會自動展開。
- **鎖頭提示**：雲端有設密碼且尚未解鎖時，`Settings` 與 `Enable edit mode` 會帶鎖頭 icon，點下去要先輸入密碼；雲端未設密碼則完全跳過密碼步驟，按一下直接生效。
- **Settings 分頁**：
  - `Display`：可視日期範圍（前/後幾天）、甘特圖標題（= 試算表檔名）、自動封存已結束的專案。
  - `Series`：新增 / 刪除 Series、調色。
  - `Phase`：新增 / 刪除 / 重新命名 Phase、調色。
  - `Site`：新增 / 刪除 Site。
  - `Resource`：規則式 Resource Check 管理 — 新增 / 編輯 / 刪除 / 啟用切換規則；每條規則可指定 Series 範圍（空 = 全部）、Phase keys（至少一個）、最大同時數。
  - `Holiday`：自訂假日行事曆 — 在內建的週末之外補上國定假日與補班日；支援單日 / 區間 / 每年重複。
  - `Admin`：設定 / 變更 / 清除雲端密碼。
  - `Import / Export`：把目前的標題、Series / Phase / Site / 顏色、Resource Rules、所有 task 一鍵匯出成 JSON 檔；亦可上傳 JSON 檔回來**覆寫**或**合併**。詳見下方「Import / Export」段落。
  - `GAS URL`：切換 Web App URL，複製分享連結；首次使用時會在這裡看到完整的「新使用者 7 步驟上手指南」。

---

## 重要設定與雲端共用機制

> 這節是「會用」之後的詳細參考，新手照「全新使用者完整上手指南」走完就能跳過。

- **GAS Web App 網址**：**唯一權威來源是網址列的 `?gas=` 參數**。
  - 開啟頁面時：有 `?gas=...` → 直接連線並把該 URL 寫入 `localStorage`（鍵 `monica-npi-gas-webapp-url`）作為本機備份；沒有 `?gas=` → **一律視為初次使用者**，自動跳出 Settings → `GAS URL` 並展開新使用者指引（不會去讀 `localStorage` 內的舊資料）。
  - 透過 Settings → GAS URL 按「儲存並連線」時，前端會用 `history.replaceState` 把 `?gas=` 寫進當前網址列，這樣 refresh / 加書籤都能保留你的 Web App，**不會每次都被當成初次使用者**。
  - 每位使用者必須跟著指引建立並貼上**自己的** Apps Script `/exec` 網址（程式內**不再內建任何共用預設 URL**）。
  - 在 Settings → GAS URL 可隨時複製「含 `?gas=` 的分享連結」轉給同事。
  - 設計目的：避免多人共用同一台電腦時，前一個人的 GAS URL 從 `localStorage` 漏給下一個開啟頁面的人；也方便你想「重新體驗 onboarding」時直接清空網址列 `?gas=` 參數即可。
- **編輯模式密碼**：雲端共用，存於 GAS `DocumentProperties.editPassword`。預設無密碼。
  - 設定 / 變更：Settings → Admin 輸入新值並按「儲存到雲端」（前端 POST `{ action: "setEditPassword", password }`）。
  - 清除：輸入框留空再「儲存到雲端」。
  - 已舊版本本機 `localStorage` 殘留會在首次連到新版 GAS 時自動上傳並清掉。
- **甘特圖標題**：標題 = Google 試算表的檔名（單一來源）。Settings → Display 改標題時，前端 POST `{ action: "setTitle", title }`，GAS 呼叫 `ss.rename()` 重新命名整本試算表。
- **Series / Phase / Site / 顏色**：雲端共用，存於 GAS `DocumentProperties.labels`（JSON 字串）。在 UI 任何修改都會即時 POST `{ action: "setLabels", labels }`，其他人重新整理即可同步看到。
- **可視日期範圍**：個人偏好，存於 `localStorage`，每人不同。預設前 7 天、後 90 天。
- **「今日」紅線**：永遠依台北時區（Asia/Taipei）當日，不需手動設。
- **自動封存已結束的專案**：個人偏好，存於 `localStorage`，每人不同。**預設啟用、門檻 30 天。**
  - **判定方式**：把一個 project 底下所有 phase 的 `end` 取最大值（`end` 空白時退而取 `start`），若早於「台北時區今日 − 門檻天數」，該 project 就從清單隱藏。
  - **純顯示層**：只是不畫出來，**不會刪除或修改試算表任何一列**。關掉設定或按一下 Header 的 `Archived` 按鈕就全部回來。
  - **門檻 0 天** = 一結束就隱藏；門檻上限 3650 天。日期全部解析不出來的 project **永遠不會被封存**，避免資料還沒填完就消失。
  - **搜尋時自動停用**：只要 Projects 搜尋框有輸入內容就不套用封存，確保舊專案一定搜得到。
  - **Header 按鈕**：僅在「當下真的有專案被封存」時才出現，顯示被隱藏的總數。點一下切換顯示。此開關**不會被記住**，重新整理後回到隱藏狀態。
  - **呈現方式**：封存的專案仍歸屬於原本的 Series，不會被抽到另一個獨立區塊。展開時每個 Series 群組內會依序是「現行專案 → `已封存 N` 小節標頭 → 封存專案」，封存列淡化並標上 `ARCHIVED`。這樣即使某個現行專案的起始日比封存專案更早（例如跨半年的長專案），兩者也不會交錯。
  - **Series 標頭計數**：`N Items` **只計算現行專案**，封存數另外以灰色 `N archived` 標示，所以切換顯示時前面那個數字不會跳動。若某個 Series 的專案全被封存，收合狀態下整個群組（含標頭）都不顯示；展開後會以 `0 Items · N archived` 出現。
  - ⚠️ 若把「可視日期範圍 → 往前」設得比封存門檻大（例如往前 365 天、門檻 30 天），時間軸上會出現一段「本來該有色塊、卻因為專案被封存而空白」的區間，Resource Check 也不會把這些專案計入。需要完整回顧歷史時，請按 `Archived` 按鈕展開，或直接關閉自動封存。
- **Resource Check**：採**規則式 (rule-based) 設計**，每條規則同步存於 GAS `DocumentProperties.labels.resourceRules`，所有使用同一支 Web App 的人共用一份。
  - **資料結構**：`{ id, name, enabled, seriesScope: string[], phaseKeys: string[], maxConcurrent: number }`。`seriesScope` 為空 = 套用到所有 Series；否則只比對命名清單。`phaseKeys` 至少要有一個，否則整條規則會被忽略不檢查。
  - **觸發邏輯**：對每個 Series、每天（排除假日）數出「有任何 active phase 屬於 `rule.phaseKeys`」的 project 數，**> `rule.maxConcurrent`** 就視為超載；多條規則並行套用，命中任一條即標紅。Cell tooltip 會列出命中的規則名稱、count vs max。
  - **建立 / 編輯 / 刪除**：Settings → Resource。每條規則內可用 chip 多選 Series 與 Phase，數字輸入框設門檻。所有變更立即同步雲端。
  - **Header 按鈕**：總開關。ON 且至少一條規則啟用 → 紅色 + `N rules` 計數；ON 但無規則 → 琥珀色 + `no rules` 提示；OFF → 灰色。Tooltip 會說明當下狀態。
  - **舊版相容**：舊版內建「Dev TNRS + Dev PI、所有 Series、同時段 > 2」是寫死的邏輯；新版預設無規則，需明確新增。Settings → Resource 在規則為空時會顯示「套用預設」按鈕，一鍵建立與舊版等同的規則。
  - ⚠️ **升級時要記得重貼 Apps Script**：規則靠 `sanitizeLabels_` 處理 `resourceRules` 欄位。若 GAS code 是 2026 之前的舊版，後端會默默丟掉 `resourceRules` 而**不報錯**，症狀就是「在 UI 編完規則、重整頁面後規則消失」。請依 [Apps Script 程式碼](#apps-script-程式碼) 重貼最新版並 `部署 → 管理部署作業 → 編輯 → 新版本 → 部署`，URL 不變。
- **假日行事曆（Settings → Holiday）**：自訂國定假日與補班日，存於 GAS `DocumentProperties.labels.holidays`，**全團隊共用一份**，確保每個人算出來的工作天一致。
  - **週末是內建的**，不需要也不應該在這裡逐週新增；這份清單只用來記「週末以外的例外」。
  - **資料結構**：`{ id, name, kind, mode, start?, end?, month?, day? }`。
    - `kind`：`off` = 放假、`work` = **補班日**（把原本是週末的那天拉回工作日，例如台灣調整放假的補上班）。
    - `mode`：`single` 單日 / `range` 起訖區間（連假一筆搞定，上限 366 天）/ `yearly` 每年重複的固定月日（例如每年 1/1、10/10，不必逐年新增）。
  - **優先序**：指定到日的設定（`single` / `range`）會蓋過 `yearly`，所以某一年的節日若調整放假，只要為那一年單獨補一筆即可，不用動每年重複的規則。而任何一筆設定都蓋過「週末」預設。
  - **影響範圍**：時間軸上的日期顏色、phase tooltip 的工作天數、Resource Check 的每日統計（假日不計）三處共用同一份判定，不會出現對不起來的情況。
  - **主 UI 呈現**：假日 = 紅色日期（自訂的另加紅色點狀底線，hover 顯示名稱）；補班日 = 藍色日期加藍色點狀底線，並且**計入工作天**。
  - **勞動節**：舊版把「每年 5/1」寫死在假日判定裡，現在改成預設就存在清單中的一筆 `yearly` 規則，行為不變但可以自行修改或刪除。
  - **容量**：labels 整包（Series / Phase / Site / 顏色 / Resource Rules / Holidays）在 GAS 端有 9000 字元上限，超過會被拒收。前端已改為**超過就不送出**（不再靜默失敗），且使用量達 85% 時 Holiday 分頁會顯示警告。連假請用 `range` 而不是逐日新增，可大幅節省空間。
  - ⚠️ **同樣需要重貼 Apps Script**：`holidays` 也是靠 `sanitizeLabels_` 處理，舊版 GAS 會默默丟掉這個欄位，症狀是「假日設定重整後消失」。處理方式同上。

---

## Import / Export

`Settings → Import / Export` 分頁讓你把整份甘特圖打包搬移，常見使用情境：

- **備份**：操作大幅異動前先匯出一份，出事可一鍵還原。
- **快速建置範本**：在一支 GAS 上配好理想的 Series / Phase / Site / 顏色甚至範例 tasks，匯出後給新同事，新同事在自己的 GAS 上一鍵套用，省去從零打字的時間。
- **遷移**：要把這份甘特圖換到另一張試算表 / 另一支 Apps Script 時，匯出再匯入即可。

### 匯出

按「下載 JSON」會產生 `<chart 標題>-YYYYMMDD-HHmm.json`，內容如下：

```json
{
  "schemaVersion": 1,
  "exportedAt": "2026-05-13T10:00:00.000Z",
  "title": "Monica NPI 排程",
  "labels": {
    "seriesList":   ["ProductA", "Gen11"],
    "siteList":     ["MX", "LZ", "MY", "CZ", "TN"],
    "seriesColors": { "ProductA": "#bfdbfe", "Gen11": "#fecaca" },
    "phaseList":    [ { "key": "Design", "label": "Design", "color": "#facc15" }, ... ],
    "resourceRules": [
      { "id": "rule-xxxx", "name": "Dev resource limit", "enabled": true,
        "seriesScope": [], "phaseKeys": ["DevelopmentTNRS", "DevelopmentPI"], "maxConcurrent": 2 }
    ],
    "holidays": [
      { "id": "hol-labour-day", "name": "勞動節", "kind": "off", "mode": "yearly", "month": 5, "day": 1 },
      { "id": "hol-xxxx", "name": "春節", "kind": "off", "mode": "range", "start": "2027-02-14", "end": "2027-02-22" },
      { "id": "hol-yyyy", "name": "補班", "kind": "work", "mode": "single", "start": "2027-02-27" }
    ]
  },
  "tasks": [
    { "project": "...", "category": "...", "task": "...", "type": "...", "start": "YYYY-MM-DD", "end": "YYYY-MM-DD", "pic": "...", "site": "..." },
    ...
  ]
}
```

> 為了安全，**編輯密碼不會被匯出**；個人偏好（可視天數、Resource Check 開關狀態、自動封存設定）也不在內，因為它們只屬於目前這顆瀏覽器。Resource Check 的**規則本身**屬於雲端共用設定，會被一起匯出 / 匯入。

### 匯入

1. 在分頁內選擇 `.json` 檔，前端會自動解析並顯示「檔案內容摘要」。
2. **挑匯入模式**（兩個分頁按鈕，預設為「覆寫」）：

   | 模式 | Title | Labels | Tasks | 適用情境 |
   |------|-------|--------|-------|---------|
   | 🔴 **覆寫** | 取代試算表名稱 | 整包覆寫雲端 `labels` | `sheet.clear()` 後寫入匯入內容 | 還原備份、整批替換 |
   | 🟢 **合併** | 仍會取代標題（若勾選） | 取**聯集**：既有 series / site / phase 保留，新項目補上；既有顏色與 phase 設定優先 | **追加**到現有 tasks 後面（不去重） | 把更多案子加入既有計畫 |

3. 三個獨立勾選框（依模式行為不同）：
   - **套用試算表標題**（呼叫 GAS `setTitle` 重新命名整本試算表；不論哪種模式都是取代）
   - **套用 Series / Phase / Site / 顏色**（覆寫模式 = 取代；合併模式 = 聯集）
   - **套用 tasks 資料**（覆寫模式 = `sheet.clear()` 後寫入；合併模式 = `[...現有, ...匯入]` 直接追加）
4. 按「套用覆寫」/「套用合併」執行。完成後會再 fetch 一次雲端確保畫面同步。

> ⚠️ **合併模式不做去重**：若匯入檔內有跟現有資料完全相同的列，會直接重複出現。建議匯入前先看一下檔案內的 project 名是否與現有重疊；若會撞名，建議先改名再匯入。

匯入會經過與一般 UI 相同的 sanitize：不合法的色碼、重複的 key、超長的字串都會被丟棄。可以放心餵任意 JSON 檔，不會把雲端搞壞。

---

## 常見問題

**Q1. 我修改了 `App.gs.js`，前端卻看不到效果？**
A. Apps Script 改完要「重新部署」：右上角 `部署 → 管理部署作業 → 編輯（鉛筆）→ 新版本 → 部署`。**不能只按儲存。**

**Q2. 我貼了 URL 但連不上？**
A. 確認三件事：
1. URL 結尾必須是 `/exec`，不是 `/dev`（`/dev` 只給編輯者本人測試）。
2. Step 3 的 `誰可以存取` 是否設成「任何人」或「任何 Google 帳號」。如果是「只有我」，其他帳號就會 403。
3. 是否完成了首次授權流程（允許讀寫試算表）。

**Q3. 試算表突然被清空 / 多筆資料消失？**
A. 每次儲存會 `sheet.clear()` 整張清空後重寫，所以**不能與其他資料共用同一個 sheet 分頁**。另外只認 `getActiveSheet()`，多分頁時要確認操作的是「打開試算表預設顯示的那個分頁」。

**Q4. 我忘記密碼怎麼辦？**
A. 打開 GAS 編輯器 → 左側 `專案設定` 或執行下面這個一次性函式：
```javascript
function _clearPasswordEmergency() {
  PropertiesService.getDocumentProperties().deleteProperty('editPassword');
}
```
按執行（會跳一次授權），跑完後重新整理甘特圖頁面，密碼就被清掉了，回到無密碼狀態。

**Q5. 我能把 Series / Phase / Site 重置回內建預設嗎？**
A. 在 GAS 編輯器執行：
```javascript
function _clearLabelsEmergency() {
  PropertiesService.getDocumentProperties().deleteProperty('labels');
}
```
重新整理前端後會回到 `INITIAL_SERIES_LIST` / `INITIAL_PHASE_LIST` / `INITIAL_SITE_LIST` 內建預設。

---

## 技術相依

- React 18（CDN UMD）
- ReactDOM 18（CDN UMD）
- Babel Standalone（瀏覽器端 JSX 轉譯）
- Tailwind CSS（CDN）

> 本專案屬於前端靜態頁面，無 `package.json`，也無 Node.js 安裝需求。

---

## 安全與維護建議

- 此頁的密碼僅為前端「軟性閘門」；真正的權限請靠 Step 3 的 Web App `誰可以存取` 設為 `任何 Google 帳號` 或 `只有我`。
- 多人共同維護時建議走標準 Git 流程（分支、PR、審查）。
- 若擴充資料欄位，請同步更新 `App.gs.js` 內 `doPost` 的 `headers` 陣列與前端寫入順序。
