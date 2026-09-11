# CLAUDE.md — animal-ecommerce-adventure（動物朋友的跨境電商大冒險）

單檔前端「故事化學習地圖」：把工作區內 5 個既有、各自獨立的跨境電商實作工具串成一段動物朋友闖關故事，依序（流量排名 → 跨境物流 → 商品標題 → 五點式產品說明 → 成本利潤）帶學員走過完整的跨境電商知識脈絡。**本專案本身不含任何跨境電商計算/生成邏輯**，純粹是敘事＋進度追蹤＋導覽入口；每個關卡的「🚪 前往關卡練習」按鈕在新分頁開啟對應工具的真實 GitHub Pages 網址，任務在那邊完成。無建置步驟、無框架、無 package.json，直接開啟 `index.html`（`file://`）或以靜態伺服器託管即可。此資料夾本身是獨立 git 儲存庫，本地身分 `Mark Tsai <tsaimark@gmail.com>`。

2026-09-11 依使用者提供的故事腳本建置，關卡順序為使用者指定的「1→5」固定順序（流量排名比較工具 → 跨境物流大挑戰 → 商品標題產生器 → 5點式產品說明產生器 → Amazon成本分析計算機），非本專案自行決定。

## 五個關卡對應表

| # | 主題 | 對應工具 | GitHub Pages 網址 | 該工具 localStorage 序號 key |
|---|------|---------|-------------------|------------------------------|
| 1 | 流量排行榜森林 | `行銷內容工具/traffic-rank-estimator` | <https://m255525.github.io/traffic-rank-estimator/> | `trafficRankSerial` |
| 2 | 跨境物流大河 | `互動遊戲/amazon-logistics-game` | <https://m255525.github.io/amazon-logistics-game/> | `logisticsGameSerial` |
| 3 | 商品標題工坊 | `行銷內容工具/product-title-generator` | <https://m255525.github.io/product-title-generator/> | `ptgSerial` |
| 4 | 商品魔法屋 | `行銷內容工具/amazon-listing-generator` | <https://m255525.github.io/amazon-listing-generator/> | `alGenSerial` |
| 5 | 老貓店長的帳本 | `資料儀表板/amazon-cost-calculator` | <https://m255525.github.io/amazon-cost-calculator/> | `amazonCostCalcSerial` |

這 5 個網址與 localStorage key 名稱是**寫死在 `index.html` 的 `STAGES`／`DOWNSTREAM_KEYS` 常數裡**，若任一來源工具改名、換網址、或改了自己的 `STORAGE_KEY` 變數名稱，本頁對應項目要跟著手動更新（沒有共用設定檔或自動同步機制）。

## 核心機制：一次登入序號、自動代入 5 個關卡（使用者本次明確要求）

首次開啟會出現「森林動物探索社」全螢幕報到閘門（`#licenseGate`），要求輸入**探索隊序號**（選填探險家名字，用於最後證書個人化）。這是**本專案自己新建的第 6 個獨立序號授權後端**（見下），不是沿用任一既有工具的序號池。

**同來源 localStorage 共享是這個機制成立的關鍵**：因為 5 個關卡工具與本頁面全部部署在同一個 GitHub Pages 使用者網域 `m255525.github.io` 底下（只是 path 不同：`/animal-ecommerce-adventure/`、`/traffic-rank-estimator/`……），瀏覽器的 `localStorage` 是以 origin（scheme+host+port）為界線，**與 path 無關**——因此本頁閘門驗證通過後，`saveSerial()` 會同時把同一組序號寫進 `DOWNSTREAM_KEYS` 陣列列出的 5 個 key，使用者點開任一關卡工具時，該工具自己的序號閘門邏輯（`input.value = loadSerial(); if (input.value) runCheck(input.value, {silent:true})`，5 個工具皆用此寫法）會讀到這組預先寫入的值並自動靜默重驗——**如果**主辦單位也把同一組序號值加進該工具自己綁定的 Google Sheet 分頁，就會直接驗證通過、不需使用者再輸入一次；如果沒有，該工具仍會顯示「查無此序號」，但欄位已預先帶入這組序號，使用者通常只需要修正而非整個重打。

**這個機制不能反向運作**：學員若直接先打開某個關卡工具、在那邊輸入序號，不會回頭同步到本頁面或其他關卡（只有本頁的閘門邏輯有寫入 `DOWNSTREAM_KEYS` 的程式碼，其他 5 個工具的閘門邏輯完全沒有修改，也不應該修改——它們是各自獨立、各自維運的專案，不因為本專案而增加耦合）。

**給主辦單位的操作提醒**（也寫在 `manual.html`）：如果要讓「一組序號、5 關全部免重複輸入」真的生效，必須把同一組序號值**分別新增到 6 個地方**——本專案自己的「AnimalAdventure序號」分頁，加上 5 個關卡工具各自的授權分頁（TrafficRank序號／AmazonLogistics序號／product-title-generator 專屬分頁／AmazonListing序號／AmazonCost序號）。只加在本專案自己的分頁，只會讓「森林動物探索社報到」這一關通過，不會讓後面 5 關自動解鎖。

## 序號授權後端（本專案專屬，12 個月，比照工作區既有骨架）

- **綁定的 Google Sheet**：沿用 `product-title-generator`／`amazon-listing-generator`／`traffic-rank-estimator`／`amazon-cost-calculator`／`amazon-logistics-game` 共用的既有表 <https://docs.google.com/spreadsheets/d/1pqGlCvUstowBzZh7J4xEa0jy3KoK4UeHUiyMTzcSGo4/edit>，`Code.gs` 固定操作獨立分頁「AnimalAdventure序號」（`SHEET_NAME` 常數），與其餘 5 個分頁互不干擾。分頁不存在時 `getLicenseSheet_()` 會自動 `insertSheet()` 並寫入表頭（序號／開始日期／結束日期）。
- **部署方式**：`clasp create --parentId <SheetID>`（不加 `--type`，在本專案 `.gas-deploy/` 內操作，該資料夾已加入 `.gitignore` 不進版控）→ 寫入客製化 `Code.gs`（`SHEET_NAME="AnimalAdventure序號"`，其餘邏輯逐字沿用 `traffic-rank-estimator` 已驗證過的骨架）→ `appsscript.json` 加 `webapp:{executeAs:"USER_DEPLOYING",access:"ANYONE_ANONYMOUS"}` → `clasp push --force` → `clasp deploy`。
- 已部署：`LICENSE_CHECK_URL = https://script.google.com/macros/s/AKfycbxM07ltAJairQi5oRlbbRC_cn4KNx-FIdXY-OZnE3bRqDRlQtRF3mqjUE1fd0ILbaRs6A/exec`，Apps Script 編輯器：<https://script.google.com/d/1Lut1ZcRXwv7agZMKpzk412OJjptSnoJUGjmQFJpQQ1h_JwEj0dFKsNw5/edit>。
- **⚠️ 尚待使用者完成一次性 OAuth 授權**：部署後 `curl -sL` 測試 `doGet` 回傳 Google 的「需要存取權」頁面（`title:"存取遭拒"`），這是正常現象（自己寫的私人腳本沒有送 Google 審查），需使用者親自打開上面的 Apps Script 編輯器網址，執行一次 `doGet` 或直接部署管理頁面跑過同意畫面，之後前端 `licenseGate` 才能正常驗證序號。此前使用者打開 `index.html` 會看到「無法連線授權伺服器」。
- 授權完成後，需請使用者（或用 `SN-maker`）在「AnimalAdventure序號」分頁新增至少一筆序號才能真正開放使用——分頁目前只有表頭、無任何序號列，比照 `license-gate-rollout-amazon-tools-traffic-rank` 記載的同類收尾步驟。

## 進度追蹤與證書（純前端、無後端）

- 5 關的「我已完成這一關的任務」勾選狀態存在 `localStorage['animalAdventureProgress']`（`{stageId: boolean}`），純自我紀錄，**不會回傳給任何伺服器**，換裝置或清瀏覽器資料會重置。
- 關卡採**軟性循序解鎖**：`render()` 依 `STAGES` 陣列順序判斷前一關是否已勾選完成，未完成則疊加 `.stage-lock-overlay` 視覺鎖定（CSS 遮罩+🔒文字），但**這只是本頁畫面上的提示，不會真的擋住玩家直接開網址使用任一關卡工具**——5 個工具本身是完全獨立、可直接存取的網站。
- 5 關全部勾選完成後，`treasureSection` 淡入、觸發 `confetti()`（逐字沿用 `amazon-logistics-game` 已驗證過的 CSS 彩帶效果），並可按「🎓 下載我的探險家證書」——`downloadCertificate()` 用 Canvas 2D 手繪一張 1000×700 PNG（含探險家名字、五關摘要、結語金句、日期），`canvas.toBlob()+<a download>` 觸發下載，零外部依賴，手法比照 `amazon-logistics-game` 的 `buildResultCanvas()`。
- 「🔄 重新挑戰一次」只清空 `animalAdventureProgress`，不影響已驗證的序號（`animalAdventureSerial`）。

## 視覺主題

淺色「森林小徑」主題（`--bg:#f5ecd7` 暖米色羊皮紙底＋`--accent:#4d8b31` 森林綠＋`--accent-2:#d97706` 琥珀橘），刻意選淺色底與工作區多數深色系姊妹工具區隔，貼近故事書/兒童冒險地圖的氣氛。關卡地圖用 `.trail::before` 中央虛線＋左右交錯卡片（`.stage.left`/`.stage.right`）呈現一條蜿蜒小徑，斷點 680px 以下收成單欄、虛線移到左側（`.trail::before{left:26px}`）。

## 功能配套範圍

- ✅ 頂部跑馬燈（沿用工作區共用 Google Apps Script 端點，逐字複製 `traffic-rank-estimator` 版本，`localStorage` key `animalAdventureMarquee`）
- ✅ PWA 加入主畫面（`manifest.json`+`service-worker.js`+`icons/`，逐字複製九專案共用已驗證版本；icons 用 PIL 手繪深色森林綠圓角方塊＋米色爪印剪影，非外部素材，見 `manifest.json` 內 icon 清單）
- ✅ 訪客計數器（`visitor-badge.laobi.icu`，`page_id=m255525.animalecommerceadventure`）
- ✅ `manual.html`（說明整體機制＋序號代入原理＋常見問題，給主辦單位與學員兩種讀者）
- ❌ 不做桌面版 exe（教學導覽入口定位，比照 `bowling-game`／`amazon-logistics-game` 從簡）
- ❌ 不做跨分頁真實完成偵測（BroadcastChannel/postMessage 需要同時修改另外 5 個獨立專案才能雙向溝通，非本專案能單方面達成，因此完成勾選是使用者自我回報）

## Port

固定用 **8816**（工作區下一個可用埠；8815 已被同日新建的 `product-keyword-cloud` 佔用）。`.claude/launch.json` 已加入對應設定。

## 指令

無建置/測試指令。修改 `index.html` 後直接用瀏覽器開啟驗證，或用 Preview MCP／`python -m http.server 8816 --directory 互動遊戲/animal-ecommerce-adventure` 暫起伺服器測完關閉。

驗證序號代入邏輯不需要真正的後端：在瀏覽器 console 執行 `localStorage.setItem('animalAdventureSerial','test123')` 後重新整理，確認 `#gateSerial` 欄位帶入該值且觸發一次 silent 重驗；驗證通過（或用 fetch mock 偽造 `{valid:true,expiresAt:...}`）後檢查 `localStorage.getItem('trafficRankSerial')` 等 5 個 key 是否也已寫入相同值。

驗證闖關進度：`localStorage.setItem('animalAdventureProgress', JSON.stringify({traffic:true,logistics:true,title:true,listing:true,cost:true}))` 後呼叫 `window.__animalAdventure.render()`，確認 `#treasureSection` 移除 `hidden`、`confetti()` 觸發、`#btnCertificate` 可正常產出 canvas 並觸發下載。

## 部署

本機開發完成，尚未推公開 GitHub Pages（比照 `pref-confirm-before-deploy-new-experimental-tool` 記憶，新專案上線前先與使用者確認）。上線流程比照工作區慣例：`gh repo create` → push → `.github/workflows/deploy-pages.yml`（Actions 部署模式）→ `gh api repos/M255525/animal-ecommerce-adventure/pages -X POST -f build_type=workflow` 啟用 Pages。
