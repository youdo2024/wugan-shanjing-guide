# 《五感山徑》逛展攻略

桃園市立圖書館《五感山徑：臺灣森林的感官之旅》逛展攻略一頁式網站，由 **佑在幹嘛** 製作。

**正式網址：<https://wugan.zeabur.app>**

- 展期：2026/07/01（三）～ 08/31（一），免費入場
- 地點：桃園市立圖書館總館 1 樓享讀拾光、3 樓兒藝展示區
- 官方活動頁：<https://www.typl.gov.tw/zh-tw/Activity/Content/9716>

## 技術

單一 `index.html`，內嵌 CSS 與 JS，無建置流程、**零外部請求**（字體用系統字，不載 Google Fonts）。
全站不使用展場實拍照，視覺以 SVG 與 CSS 完成。

## 手機（iPhone 原生手感）

以下是刻意的決定，改動前請先看理由：

| 做法 | 理由 |
|---|---|
| 字體用 `-apple-system` → `PingFang TC` | iPhone 直接吃 SF Pro＋蘋方，第一次繪製就是對的樣子，也省掉一次外部字體請求 |
| **不在 `html` 設 `scroll-behavior:smooth`** | Safari 15.4 起它會攔截 JS 的捲動指令，也會跟 iOS 慣性捲動打架。要平滑的地方由 JS 帶 `behavior:"smooth"` |
| `:hover` 全部包在 `@media (hover:hover) and (pointer:fine)` | iOS 點過元素後 `:hover` 會卡住不放，是 WebKit 已知行為 |
| 改用 `:active` 縮放回饋（壓下 60ms／放開 340ms） | `-webkit-tap-highlight-color` 已關，需要自己補回按壓感，這個節奏接近 UIKit 按鈕 |
| `touch-action:manipulation` | WebKit 官方建議，取消雙擊縮放判定讓點擊即時反應 |
| `env(safe-area-inset-*)` + `viewport-fit=cover` | 瀏海與 Home Indicator 讓位；加到主畫面時才不會被切到 |
| Hero 用 `100svh` 不用 `100dvh` | `dvh` 會隨網址列伸縮不停重排；`svh` 穩定且內容一定看得到 |
| `html` 與 `body` 都設底色 | iOS 26 根層透明會退回白色，上下會出現白條 |
| fixed 元素（進度條／回頂鈕／toast）本體透明，顏色放在 `::before`／`::after` | Safari 26 的狀態列與工具列會取樣 fixed 元素的底色，直接上色會把系統列染成奇怪的顏色 |
| **保留**直向橡皮筋回彈 | 那是 iOS 原生手感。要擋的是橫向捲動區傳遞手勢，已用 `overscroll-behavior-x:contain` 各自處理 |
| 沒有做觸覺回饋（haptics） | iOS Safari 沒有 Vibration API；社群流傳的 `<input type="checkbox" switch>` 觸發法已被 iOS 26.5 修掉，現在多數使用者用不到 |
| 支援深色模式 | 跟著 iPhone 系統外觀自動切換，見下一節 |

## 深色模式

跟著系統設定自動切換，不做手動切換鈕（iOS 使用者預期跟系統走）。

所有顏色都收斂成 `:root` 的 CSS 變數，深色版只覆寫變數，不重寫任何版面規則。
要調色請改變數，**不要在規則裡硬寫色碼**，否則深色模式會漏掉。

幾個容易踩到的點：

- `--head`（標題字）與 `--brown-d`（深色面）刻意拆開。深色時標題要翻白，但頁尾、結束橫幅那些深色底不能跟著翻。
- `--green` / `--moss` 是**背景**用色，`--green-txt` / `--moss-txt` / `--ok-txt` 是**文字**用色。深色時文字版要提亮，背景版不能動，否則白字會糊掉。
- 山稜插圖的三層與樹用 `--mt1`~`--mt4` 控制，CSS 的 `fill` 會蓋掉 SVG 的 `fill` 屬性。
- Hero 花粉粒子顏色由 `--dust` 提供，JS 讀取變數並監聽 `prefers-color-scheme` 變化。
- `theme-color` 有 light / dark 兩個 meta（給 Android Chrome；Safari 26 已不看這個）。

**對比度**：淺色與深色各 16 項文字實測全數通過 WCAG AA。過程中發現淺色模式的苔綠標籤字原本只有 3.4:1
（規格書 §1 要求 AA），已把 `--moss-txt` 從 `#7A8C6B` 壓深到 `#5F7154`，變成 4.8:1。

`manifest.json` + `apple-touch-icon.png` 讓「加入主畫面」能用（`display:standalone`）。
副檔名刻意用 `.json` 不用 `.webmanifest`，因為 Zeabur 的靜態伺服器把 `.webmanifest` 回成 `text/plain`，Chrome 會拒收。
`apple-mobile-web-app-capable` 這個舊 meta 已被官方標為過時，故意不加。

## 維護

所有可能異動的資料都集中在 `index.html` `<script>` 開頭的常數區：

| 常數 | 用途 |
|---|---|
| `EXHIBITION_END` | 展期結束日。超過這天，頁面自動切換為「展覽已結束」狀態 |
| `CLOSED_DATES` | 清館日。程式已能自動算出「當月最後一個週四」，這裡只是額外保險 |
| `HOLIDAYS` | 國定假日休館日，**需手動維護**，格式 `"2026-09-28"` |
| `EVENTS` | 活動場次。過期場次會自動隱藏，衝堂會依實際時間重疊自動偵測 |
| `LINKS` | 各類型活動的官方報名頁連結 |

改場次只要動 `EVENTS` 陣列，衝堂標記與類型篩選的數字都會自己重算。

## 數據追蹤

`index.html` 的 `<script>` 開頭有一行：

```js
const GA_ID = "";
```

把 Google Analytics 4 的評估 ID（`G-` 開頭）貼進去就開始收數字。留空時整段追蹤不會載入，
網站照常運作、外部請求維持為零，頁尾的隱私告知也不會出現。

**追蹤的事件**

| 事件 | 參數 | 這個數字能回答什麼 |
|---|---|---|
| `page_view` | GA4 自動 | 總共多少人看過（結案報告的主數字） |
| `library_status` | `status` | 有多少人是在**休館日**打開這頁的，也就是這頁擋掉多少次白跑一趟 |
| `signup_click` | `event_id` | 官方報名頁的導流次數，逐場次分開算 |
| `intent_click` | `action` | 按了導航到總館／撥打電話／官方活動頁，高意圖行為 |
| `cta_click` | `id` | 導回展覽任務活動貼文與佑在幹嘛官網的次數 |
| `scroll_depth` | `percent` | 讀到 25／50／75／100% 的比例，看內容有沒有被讀完 |
| `engagement_summary` | `max_scroll_percent`, `seconds_on_page` | 離開時補送，單次瀏覽的深度與停留秒數 |
| `checklist_complete` | — | 出發前三件事全部勾完的人數 |
| `trail_open` / `flip_open` / `route_switch` / `event_filter` / `share` | 各自的識別 | 哪些互動真的有人用 |

同一個事件在一次瀏覽裡只送一次（`scroll_depth`、`library_status`、`checklist_complete`），不會重複灌數字。

**驗證埋點**：網址加 `?debug=1`，操作頁面時主控台會印出每一個事件。

## 測試

網址後面加參數可以模擬任何日期時間（需用 http server 開，直接開檔案不吃參數）：

```
?date=2026-08-27          # 清館日休館
?date=2026-09-01          # 展覽已結束
?date=2026-08-10          # 確認 8/01、8/09 場次已消失
?date=2026-07-29&time=22:00   # 已閉館，且會跳過隔天的清館日
```

本機起一個 server：

```bash
python3 -m http.server 4173
```

## 部署

推到 GitHub 後由 Zeabur 自動部署（靜態網站），網址 <https://wugan.zeabur.app>。

社群分享卡是 `og-image.png`（1200×630，程式繪製，非展場實拍）。
`index.html` 的 `og:url` / `og:image` / `canonical` 都寫死絕對網址，**換網域時這三處要一起改**。

## 資料來源與免責

資訊整理自主辦單位公開資料與媒體報導，資料查證日 2026/07/28。
展區內容、活動場次與名額可能調整，實際狀況以桃園市立圖書館官方公告為準。
