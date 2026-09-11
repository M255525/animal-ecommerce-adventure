# CLAUDE.md — animal-ecommerce-adventure（動物朋友的跨境電商大冒險）

單檔前端「故事化學習地圖」：把工作區內 7 個既有、各自獨立的跨境電商實作工具串成一段動物朋友闖關故事，依序（流量排名 → 跨境物流 →〈中途休息島：關鍵字文字雲，不計分〉→ 商品標題 → 五點式產品說明 → 成本利潤 → 整合工作流）帶學員走過完整的跨境電商知識脈絡。**本專案本身不含任何跨境電商計算/生成邏輯**，純粹是敘事＋進度追蹤＋導覽入口；每個關卡的「🚪 前往關卡練習」按鈕在新分頁開啟對應工具的真實網址，任務在那邊完成。無建置步驟、無框架、無 package.json，直接開啟 `index.html`（`file://`）或以靜態伺服器託管即可。此資料夾本身是獨立 git 儲存庫，本地身分 `Mark Tsai <tsaimark@gmail.com>`。

2026-09-11 依使用者提供的故事腳本建置，關卡順序為使用者指定的「1→5」固定順序（流量排名比較工具 → 跨境物流大挑戰 → 商品標題產生器 → 5點式產品說明產生器 → Amazon成本分析計算機），非本專案自行決定；同日稍後使用者再要求加入第 6 關「內容工作流工作室」（`content-workflow-studio-cf`，放在最後一關）並調整故事，同時補上使用警語與創作者資訊；再稍後又要求把「跨境電商關鍵字文字雲產生器」（`product-keyword-cloud`）插進第 2、3 關中間，定位為「休息島嶼，只有練習不計分」。

## 六個必要關卡＋一座不計分休息島對應表

| # | 主題 | 對應工具 | 網址 | 該工具 localStorage 序號 key | 與本頁同源？ | 計入進度？ |
|---|------|---------|------|------------------------------|--------------|-----------|
| 1 | 流量排行榜森林 | `行銷內容工具/traffic-rank-estimator` | <https://m255525.github.io/traffic-rank-estimator/> | `trafficRankSerial` | 是 | 是 |
| 2 | 跨境物流大河 | `互動遊戲/amazon-logistics-game` | <https://m255525.github.io/amazon-logistics-game/> | `logisticsGameSerial` | 是 | 是 |
| 🏝️ | 中途休息島 | `行銷內容工具/product-keyword-cloud` | <https://m255525.github.io/product-keyword-cloud/> | `pkcLicenseSerial` | 是 | **否（`optional:true`）** |
| 3 | 商品標題工坊 | `行銷內容工具/product-title-generator` | <https://m255525.github.io/product-title-generator/> | `ptgSerial` | 是 | 是 |
| 4 | 商品魔法屋 | `行銷內容工具/amazon-listing-generator` | <https://m255525.github.io/amazon-listing-generator/> | `alGenSerial` | 是 | 是 |
| 5 | 老貓店長的帳本 | `資料儀表板/amazon-cost-calculator` | <https://m255525.github.io/amazon-cost-calculator/> | `amazonCostCalcSerial` | 是 | 是 |
| 6 | 跨境電商指揮塔 | `行銷內容工具/content-workflow-studio-cf` | <https://content-workflow-studio-cf.content-workflow.workers.dev/> | `workflowStudioEntrySerial` | **否**（部署在 Cloudflare Workers 網域，非 `m255525.github.io`） | 是 |

這些網址與 localStorage key 名稱是**寫死在 `index.html` 的 `STAGES`／`DOWNSTREAM_KEYS` 常數裡**，若任一來源工具改名、換網址、或改了自己的 `STORAGE_KEY` 變數名稱，本頁對應項目要跟著手動更新（沒有共用設定檔或自動同步機制）。**只有第 6 關跟本頁不同源**——`content-workflow-studio-cf` 部署在 Cloudflare Workers（`*.content-workflow.workers.dev`），因此第 6 關的 `workflowStudioEntrySerial` 刻意**不放進** `DOWNSTREAM_KEYS`（寫了也不會被那個網域讀到），`STAGES` 陣列裡該筆多了 `crossOrigin:true` 與 `note` 欄位；休息島的 `product-keyword-cloud` 雖然也部署在 `m255525.github.io`（同源），但因為它是額外插進去的練習站，仍需比照其他同源關卡列進 `DOWNSTREAM_KEYS`（已加入）。

### 「休息島」（`optional:true`）不計分的實作細節

`STAGES` 陣列裡的必要關卡都多一個 `number`（1-6，寫死對應 `kicker` 文字裡的「第N關」）欄位；休息島這筆改用 `optional:true`，沒有 `number`。`render()` 依此分流：

- `var requiredStages = STAGES.filter(function(s){return !s.optional;});`——所有計分/計數/解鎖邏輯只看這個子集合，不是原始 `STAGES` 陣列的相鄰索引。**這是關鍵**：休息島插在陣列中間（logistics 和 title 之間），如果沿用「跟陣列前一筆比對」的舊寫法，商品標題工坊（title）的解鎖判斷會誤看休息島的完成狀態（而休息島永遠不會被標記完成，會讓 title 永久卡住無法解鎖）。改法是每個必要關卡改用 `requiredStages.indexOf(stage)` 找出自己在「必要關卡子序列」裡的位置，跟該子序列的前一筆比較。
- 休息島 `isLocked` 恆為 `false`（不受任何關卡影響，也不影響任何關卡）、不渲染「我已完成」checkbox（`isOptional` 為真時 `stage-actions` 只有 CTA 按鈕）、卡片上改渲染 `.stage-optional-badge`（「🏝️ 不計分．自由練習」）取代打勾方塊、`.stage-node` 顯示 `stage.emoji`（🏝️）而非數字、卡片改虛線邊框＋米色底（`.stage.optional .stage-card`）視覺上跟六個正式關卡區隔。
- `doneCount`／`requiredTotal`／進度條／進度文字／全破關判斷全部改用 `requiredStages`，維持「X / 6」不受休息島影響。
- **日後如果要再插入更多「不計分」的支線關卡**，比照這個模式：`STAGES` 裡加 `optional:true`（不給 `number`），不要手動調整其他必要關卡的陣列順序判斷邏輯——`requiredStages` 這層抽象已經處理好穿插的情境。

## 核心機制：一次登入序號、自動代入大部分關卡（使用者本次明確要求；第 6 關為例外，見上表）

首次開啟會出現「森林動物探索社」全螢幕報到閘門（`#licenseGate`），要求輸入**探索隊序號**（可選填姓名／組別／學號／系所，用於證書、截圖與進度存檔個人化）。這是**本專案自己新建的獨立序號授權後端**（見下），不是沿用任一既有工具的序號池。

**同來源 localStorage 共享是這個機制成立的關鍵，但只適用於前 5 關**：前 5 個關卡工具與本頁面全部部署在同一個 GitHub Pages 使用者網域 `m255525.github.io` 底下（只是 path 不同：`/animal-ecommerce-adventure/`、`/traffic-rank-estimator/`……），瀏覽器的 `localStorage` 是以 origin（scheme+host+port）為界線，**與 path 無關**——因此本頁閘門驗證通過後，`saveSerial()` 會同時把同一組序號寫進 `DOWNSTREAM_KEYS` 陣列列出的 5 個 key，使用者點開任一關卡工具時，該工具自己的序號閘門邏輯（`input.value = loadSerial(); if (input.value) runCheck(input.value, {silent:true})`，5 個工具皆用此寫法）會讀到這組預先寫入的值並自動靜默重驗——**如果**主辦單位也把同一組序號值加進該工具自己綁定的 Google Sheet 分頁，就會直接驗證通過、不需使用者再輸入一次；如果沒有，該工具仍會顯示「查無此序號」，但欄位已預先帶入這組序號，使用者通常只需要修正而非整個重打。**第 6 關「內容工作流工作室」部署在 Cloudflare Workers 的不同網域，這套機制對它完全不生效**，序號永遠需要另外手動輸入（見上表與 `note` 欄位）。

**這個機制不能反向運作**：學員若直接先打開某個關卡工具、在那邊輸入序號，不會回頭同步到本頁面或其他關卡（只有本頁的閘門邏輯有寫入 `DOWNSTREAM_KEYS` 的程式碼，其他 5 個工具的閘門邏輯完全沒有修改，也不應該修改——它們是各自獨立、各自維運的專案，不因為本專案而增加耦合）。

**給主辦單位的操作提醒**（也寫在 `manual.html`）：如果要讓「一組序號、大部分關卡免重複輸入」真的生效，必須把同一組序號值**分別新增到 7 個地方**——本專案自己的「AnimalAdventure序號」分頁，加上 6 個同源關卡工具各自的授權分頁（TrafficRank序號／AmazonLogistics序號／關鍵字文字雲專屬分頁／product-title-generator 專屬分頁／AmazonListing序號／AmazonCost序號）。只加在本專案自己的分頁，只會讓「森林動物探索社報到」這一關通過，不會讓後面關卡自動解鎖。第 6 關「內容工作流工作室」的序號則要另外加進它自己獨立的授權清單（`content-workflow-studio-cf` 自己的 `Code.gs`／Google Sheet，本專案不知情也管不到）。

## 序號授權後端（本專案專屬，12 個月，比照工作區既有骨架）

- **綁定的 Google Sheet**：沿用 `product-title-generator`／`amazon-listing-generator`／`traffic-rank-estimator`／`amazon-cost-calculator`／`amazon-logistics-game` 共用的既有表 <https://docs.google.com/spreadsheets/d/1pqGlCvUstowBzZh7J4xEa0jy3KoK4UeHUiyMTzcSGo4/edit>，`Code.gs` 固定操作獨立分頁「AnimalAdventure序號」（`SHEET_NAME` 常數），與其餘 5 個分頁互不干擾。分頁不存在時 `getLicenseSheet_()` 會自動 `insertSheet()` 並寫入表頭（序號／開始日期／結束日期）。
- **部署方式**：`clasp create --parentId <SheetID>`（不加 `--type`，在本專案 `.gas-deploy/` 內操作，該資料夾已加入 `.gitignore` 不進版控）→ 寫入客製化 `Code.gs`（`SHEET_NAME="AnimalAdventure序號"`，其餘邏輯逐字沿用 `traffic-rank-estimator` 已驗證過的骨架）→ `appsscript.json` 加 `webapp:{executeAs:"USER_DEPLOYING",access:"ANYONE_ANONYMOUS"}` → `clasp push --force` → `clasp deploy`。
- 已部署：`LICENSE_CHECK_URL = https://script.google.com/macros/s/AKfycbxM07ltAJairQi5oRlbbRC_cn4KNx-FIdXY-OZnE3bRqDRlQtRF3mqjUE1fd0ILbaRs6A/exec`，Apps Script 編輯器：<https://script.google.com/d/1Lut1ZcRXwv7agZMKpzk412OJjptSnoJUGjmQFJpQQ1h_JwEj0dFKsNw5/edit>。
- **✅ 2026-09-11 已確認 OAuth 授權完成**：曾在瀏覽器 `fetch()` 對 `LICENSE_CHECK_URL` 送出測試序號，回傳正常的 `{"valid":false,"reason":"serial_not_found"}` JSON（而非早期的「需要存取權」錯誤頁），確認一次性 OAuth 同意流程已由使用者完成，`licenseGate` 可以正常運作。
- 目前「AnimalAdventure序號」分頁應該仍只有表頭、無任何序號列（除非使用者已自行新增）——需請使用者（或用 `SN-maker`）新增至少一筆序號才能真正開放使用，比照 `license-gate-rollout-amazon-tools-traffic-rank` 記載的同類收尾步驟。

## 身分欄位（姓名／組別／學號／系所，2026-09-11 應使用者要求新增）

報到閘門的姓名輸入框旁新增 `.gate-id-grid`（3 欄，420px 以下斷點收成 2 欄）：組別／學號／系所，皆選填。四欄合併存成單一 JSON 物件 `localStorage['animalAdventureIdentity']`（`{name, group, studentId, dept}`），**由閘門 IIFE 與主程式 IIFE 共用同一個 key**（兩個 IIFE 各自定義了同名的 `IDENTITY_KEY`／`loadIdentity()`，未共用變數但寫讀同一把 localStorage key，屬於刻意的鬆耦合設計，跟其餘「獨立運作」的 IIFE 慣例一致）。填寫過一次下次開啟會自動帶回四個輸入框（比照 `amazon-logistics-game` 的 `loadIdentityDraft()` 慣例）。四欄只用於畫面顯示（證書／截圖／進度存檔），**完全不會送進任何序號驗證請求**。

## 進度追蹤、證書、截圖分享與存檔（純前端、無後端）

- 5 關的「我已完成這一關的任務」勾選狀態存在 `localStorage['animalAdventureProgress']`（`{stageId: boolean}`），純自我紀錄，**不會回傳給任何伺服器**，換裝置或清瀏覽器資料會重置（可用下方存檔／讀檔功能因應）。
- 關卡採**軟性循序解鎖**：`render()` 依 `STAGES` 陣列順序判斷前一關是否已勾選完成，未完成則疊加 `.stage-lock-overlay` 視覺鎖定（CSS 遮罩+🔒文字），但**這只是本頁畫面上的提示，不會真的擋住玩家直接開網址使用任一關卡工具**——5 個工具本身是完全獨立、可直接存取的網站。
- 5 關全部勾選完成後，`treasureSection` 淡入、顯示 `#treasureIdentity`（`identityLine()` 組合姓名／組別／學號／系所，全空時顯示「（未填寫……）」，比照 `traffic-rank-estimator` 的 `renderResultIdentity()` 慣例）、觸發 `confetti()`（逐字沿用 `amazon-logistics-game` 已驗證過的 CSS 彩帶效果），並提供三種輸出：
  - **🎓 下載我的探險家證書**：`downloadCertificate(identity)` 用 Canvas 2D 手繪一張 1000×700 PNG（含姓名、組別/學號/系所——用 `ctx.measureText()` 量寬過長才自動換行，手法比照 `amazon-logistics-game` 的 `fitIdentityLines()`、五關摘要、結語金句、日期），`canvas.toBlob()+<a download>` 觸發下載，零外部依賴。
  - **📷 下載截圖**（2026-09-11 新增）：`html2canvas`（CDN `cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1`）直接對 `#treasureSection` 整個節點截圖成 PNG，內容與畫面所見一致（含識別資訊列），手法比照 `traffic-rank-estimator` 的 `downloadScreenshot()`。
  - **📤 分享**（2026-09-11 新增）：只有 `navigator.share`＋`navigator.canShare` 都存在時才顯示按鈕（主要是手機瀏覽器）；截圖後包成 `File` 呼叫 `navigator.share({files:[...]})` 跳系統分享選單，`canShare` 檢查未過或分享失敗則退回等同「下載截圖」的行為。
- 「🔄 重新挑戰一次」只清空 `animalAdventureProgress`，不影響已驗證的序號（`animalAdventureSerial`）與身分欄位。

## 進度存檔／讀檔（`.json` 檔案，2026-09-11 應使用者要求新增：「沒完成也能存檔，下次接著玩」）

進度條下方 `.progress-actions` 提供：

- **💾 下載進度存檔**：把 `{type, version, exportedAt, identity, progress}` 包成 JSON，`Blob`+`<a download>` 存成 `跨境電商冒險進度_<姓名>_<日期>.json`。
- **📂 匯入進度存檔**：隱藏的 `<input type="file" accept="application/json">` 由按鈕觸發 `.click()`，`FileReader.readAsText()` 讀檔→`JSON.parse`→驗證 `data.progress` 是物件才採用（格式不符會 `alert` 提示、不覆蓋現有進度）→分別寫回 `animalAdventureProgress`／`animalAdventureIdentity`→同步回填報到閘門四個輸入框→呼叫 `render()` 重繪→`alert('進度已匯入！')`。
- 這組存檔/讀檔**只搬動 `progress` 與 `identity`，刻意不包含序號**（`animalAdventureSerial`）——換裝置/瀏覽器時序號驗證仍須照第一次報到流程重新輸入，存檔只負責接續故事進度與身分資訊，避免序號檔案外流被冒用的疑慮。
- 已用 Playwright 以 `DataTransfer` 建構真實 `File` 物件指派給 `input.files` 並手動 `dispatchEvent('change')` 完整驗證這條路徑（無法用真實檔案選取對話框自動化測試，這是目前最接近真實使用者操作的驗證方式）。

## 動物夥伴選角（2026-09-11 應使用者要求新增）

報到閘門新增「選擇你的動物夥伴」步驟：`CHARACTERS` 常數（5 筆，`{id,emoji,name,role,color}`，對應故事裡的兔子/狐狸/小熊/海獺/貓頭鷹）定義在主程式 IIFE、透過 `window.__animalAdventure.CHARACTERS` 讓報到閘門 IIFE 讀取（避免兩個 IIFE 各自維護一份重複資料）。`#charGrid` 由 `buildCharGrid()` 動態產生 5 張可點選卡片，選中的角色**以 DOM 上 `.char-card.selected` 為唯一事實來源**（`getSelectedCharacterId()` 現場查詢，不另外存一份 JS 閉包變數）——這是刻意的設計，因為「匯入進度存檔」會直接改寫 DOM class 來還原選角狀態，如果另外維護一個閉包變數，兩邊值容易不同步（畫面看起來已選、實際送出卻是空的）。

**選角是報到的必要步驟，不是純裝飾**：`btnGateConfirm` 點擊時若 `getSelectedCharacterId()` 為空，會顯示 `#charHint` 提示文字並捲動到選角區塊、直接 `return`，不會繼續送出序號驗證——這是使用者本次明確要求「進來要選擇一個角色」的行為，不是選填。選中的角色 id 與「幫動物夥伴取的名字」（沿用既有的 `姓名` 欄位，不另開一個重複的命名欄位）一起存進 `animalAdventureIdentity`（新增 `character` 欄位）。

角色資訊會出現在三個地方：
- **`#playerChip`**（topbar，`renderPlayerChip()`）：顯示「🦊 小明」樣式的小徽章，背景色用 `--char-color` 走 `color-mix()`（有 `var(--surface-2)` 純色 fallback，見下）。
- **`#teamRoster`**（hero 區塊下方，`renderTeamRoster()`）：5 個動物頭像橫排展示團隊成員，選中的角色疊加 `.active` 樣式（邊框變成該角色代表色）。
- **證書與寶箱識別資訊列**：`downloadCertificate()` 在姓名前加上角色 emoji、`identityLine()`／`#treasureIdentity` 在最前面加一行「動物夥伴：🦊 狐狸（市場分析師）」。

`saveIdentity()`（報到閘門 IIFE）存完之後會呼叫 `window.__animalAdventure.render()`（不是只呼叫 `renderPlayerChip()`）——因為 `render()` 內部才會一併重繪 `teamRoster` 的 `.active` 狀態，只呼叫 `renderPlayerChip()` 會讓 topbar 徽章更新但團隊橫幅的高亮沒跟著變，兩者曾經不同步過一次，修好後統一走 `render()`。

**`color-mix()` 相容性**：`.char-card.selected`／`.player-chip` 的背景色用到 `color-mix(in srgb, var(--char-color) N%, var(--surface-2))`（較新的 CSS 函式，Safari 16.2+/Chrome 111+/Firefox 113+ 才支援）。寫法是**同一個屬性寫兩次**——先寫一個純色 fallback（`background:var(--surface-2)` 等），再寫一次 `color-mix()` 版本；不支援的瀏覽器會整條 `color-mix()` 宣告視為不可解析而被忽略，保留前一條 fallback 的值，支援的瀏覽器則以後者覆蓋，不需要 `@supports` 判斷。

## 視覺強化（2026-09-11 應使用者要求「畫面太素淨，強化動物朋友的感覺，但不要過於凌亂」）

- **背景爪印紋理**：`body` 的 `background` 疊加一層極低透明度（`fill-opacity:0.05`）的 inline SVG 爪印圖案（`data:image/svg+xml,...`），`background-size:140px 140px` 平鋪，刻意壓得很淡、只當作紙感紋理，不與內容搶視覺。
- **對話泡泡式引言**：`.stage-quote` 從純文字引言改成「圓形動物頭像 + 泡泡」佈局（`.quote-avatar` 用該關 `stage.animal` 的 emoji），左上角直角模擬對話框尖角（`border-top-left-radius:4px`），呼應「動物朋友在說話」的故事感。
- **團隊成員橫幅**：見上方「動物夥伴選角」一節的 `#teamRoster`。
- 三者都刻意維持低飽和度、小尺寸、不佔用太多版面，避免使用者原本擔心的「凌亂」——沒有新增跑馬燈以外的動態效果、沒有額外彈窗或強制引導。

## 視覺主題

淺色「森林小徑」主題（`--bg:#f5ecd7` 暖米色羊皮紙底＋`--accent:#4d8b31` 森林綠＋`--accent-2:#d97706` 琥珀橘），刻意選淺色底與工作區多數深色系姊妹工具區隔，貼近故事書/兒童冒險地圖的氣氛。關卡地圖用 `.trail::before` 中央虛線＋左右交錯卡片（`.stage.left`/`.stage.right`）呈現一條蜿蜒小徑，斷點 680px 以下收成單欄、虛線移到左側（`.trail::before{left:26px}`）。

**踩坑記錄——`.gate-id-grid` 曾經橫向溢出且未套樣式**：組別/學號/系所這三個欄位的 `<input>` 因為不在 `.gate-row` 裡也沒有 `#gateName` 這個 id，一開始完全沒吃到任何自訂樣式（背景/邊框/圓角），維持瀏覽器原生外觀，且原生 `<input>` 的預設最小寬度（約 170px）撐爆了 grid 的 1fr 欄位，導致 `.gate-box` 出現水平捲軸。修法：`.gate-row input,#gateName,.gate-id-grid input` 三個選擇器合併成同一條規則統一套樣式，並加 `min-width:0` 讓 grid track 可以縮到比 input 預設最小寬度更窄。**日後在既有 `.gate-row`／`#gateName` 樣式規則之外新增任何欄位輸入框時，記得把新的 selector 一併加進這條規則，不要只顧著加版面（grid/flex）卻忘記樣式跟最小寬度。**

## 第六關與故事調整（2026-09-11 應使用者要求新增）

原始腳本只有 5 個任務、以「Amazon 成本分析計算機」收尾。使用者要求追加第 6 關「內容工作流工作室」（`content-workflow-studio-cf`，一直沒有分配到台詞的**海獺**在這關首次開口），網址放在最後一關，並調整故事讓「成本利潤」不再是終點，改為「把所有技能串成一條工作流程」才是真正的終局：

- 第 5 關（`cost`）的 `story`／`task` 移除了原本「最後」「最終任務」等暗示故事結束的字眼，改為承先啟後的過渡。
- 第 6 關（`workflow`）的 `kicker` 為「第六關 · 跨境電商指揮塔」，`story` 由海獺提出「把整條流程串成一部自動運轉的機器」的觀點，呼應 `content-workflow-studio-cf` 本身「串接多個工具成一條工作流」的產品定位（見其 `CLAUDE.md`）；`task` 標示為「🎯 最終任務」。
- 全域文案（hero 信件、寶箱結語 blockquote、證書內文、確認彈窗、進度計數）已全數從「5」改為「6」，`downloadCertificate()` 的關卡摘要行改用精簡的六段式「流量排行榜 → 跨境物流 → 商品標題 → 產品說明 → 成本利潤 → 整合工作流」避免證書 1000px 寬度換行溢出。
- 第 6 關卡片獨有 `note` 欄位（`STAGES[5].note`），`render()` 會多渲染一個 `.stage-note` 提示區塊，說明序號不會自動代入（見上方跨網域說明）。

## 使用警語與創作者資訊（2026-09-11 應使用者要求新增）

`index.html` 的 `<footer>` 新增 `.warn-box`（使用警語，涵蓋「本頁不提供電商功能」「完成勾選為自我回報」「各關卡內容以該工具自身條款為準」「中途休息島不計分不列入進度」「第六關序號不會自動代入」「資料只存本機不回傳」「僅供教學個人使用」）與 `.creator-box`（創作者資訊：Mark Tsai（蔡豐全）），版面與配色（`rgba(var(--bad-rgb),...)`／`rgba(var(--accent-rgb),...)`）比照 `traffic-rank-estimator` 的 `.warn-box`/`.creator-box`/`.footer-meta` 樣式但套用本專案的 CSS token 命名。`footer-links` 內新增 `製作：Mark Tsai｜tsaimark@gmail.com` 一行（`mailto:` 連結）。`manual.html` 同步更新頂部警語文字與底部創作者資訊（補上全名與 email），工具總數隨休息島加入從 6→7 一併更新。

## 功能配套範圍

- ✅ 頂部跑馬燈（沿用工作區共用 Google Apps Script 端點，逐字複製 `traffic-rank-estimator` 版本，`localStorage` key `animalAdventureMarquee`）
- ✅ PWA 加入主畫面（`manifest.json`+`service-worker.js`+`icons/`，逐字複製九專案共用已驗證版本；icons 用 PIL 手繪深色森林綠圓角方塊＋米色爪印剪影，非外部素材，見 `manifest.json` 內 icon 清單）
- ✅ 訪客計數器（`visitor-badge.laobi.icu`，`page_id=m255525.animalecommerceadventure`）
- ✅ `manual.html`（說明整體機制＋序號代入原理＋常見問題，給主辦單位與學員兩種讀者）
- ✅ 使用警語＋創作者資訊（`index.html` footer 的 `.warn-box`/`.creator-box`，見上）
- ❌ 不做桌面版 exe（教學導覽入口定位，比照 `bowling-game`／`amazon-logistics-game` 從簡）
- ❌ 不做跨分頁真實完成偵測（BroadcastChannel/postMessage 需要同時修改另外 5 個同源獨立專案才能雙向溝通，第 6 關更是跨網域完全無法溝通，因此完成勾選是使用者自我回報）

## Port

固定用 **8816**（工作區下一個可用埠；8815 已被同日新建的 `product-keyword-cloud` 佔用）。`.claude/launch.json` 已加入對應設定。

## 指令

無建置/測試指令。修改 `index.html` 後直接用瀏覽器開啟驗證，或用 Preview MCP／`python -m http.server 8816 --directory 互動遊戲/animal-ecommerce-adventure` 暫起伺服器測完關閉。

驗證序號代入邏輯不需要真正的後端：在瀏覽器 console 執行 `localStorage.setItem('animalAdventureSerial','test123')` 後重新整理，確認 `#gateSerial` 欄位帶入該值且觸發一次 silent 重驗；驗證通過（或用 fetch mock 偽造 `{valid:true,expiresAt:...}`）後檢查 `localStorage.getItem('trafficRankSerial')` 等 5 個 key 是否也已寫入相同值。

驗證闖關進度：`localStorage.setItem('animalAdventureProgress', JSON.stringify({traffic:true,logistics:true,title:true,listing:true,cost:true,workflow:true}))` 後呼叫 `window.__animalAdventure.render()`，確認 `#treasureSection` 移除 `hidden`、`#treasureIdentity` 正確組合身分欄位、`confetti()` 觸發、`#btnCertificate`／`#btnScreenshot` 皆可正常產出並觸發下載。另外確認 `STAGES[5].note`（第六關跨網域提示）有正確渲染成 `.stage-note`。

驗證進度存檔／讀檔：用 `new File([JSON.stringify(payload)], 'x.json')` + `DataTransfer` 建構真實 File 物件指派給 `#loadProgressInput.files`，手動 `dispatchEvent(new Event('change',{bubbles:true}))`（真實檔案選取對話框無法自動化），確認 `alert('進度已匯入！')` 觸發、`animalAdventureProgress`／`animalAdventureIdentity` 與四個報到欄位皆正確還原。

## 部署

本機開發完成，尚未推公開 GitHub Pages（比照 `pref-confirm-before-deploy-new-experimental-tool` 記憶，新專案上線前先與使用者確認）。上線流程比照工作區慣例：`gh repo create` → push → `.github/workflows/deploy-pages.yml`（Actions 部署模式）→ `gh api repos/M255525/animal-ecommerce-adventure/pages -X POST -f build_type=workflow` 啟用 Pages。
