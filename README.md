# 營隊資訊網頁 — 維護說明

營隊的報名資訊與公告頁面。公告與首頁輪播照片由工作人員在後台（`admin.html`）更新；錄取名單由 Google 試算表發布；其餘內容（基本資訊、報名注意事項）寫在 `index.html`，由技術維護者更新。

完整需求與決策紀錄見 [`營隊網頁規格書.md`](營隊網頁規格書.md)。

## 檔案結構

| 檔案 | 用途 |
|---|---|
| `index.html` | 整個網頁（HTML + CSS + JS） |
| `admin.html` | 工作人員後台：管理公告、輪播照片（見第 3 節） |
| `data/notices.json` | 公告資料，由後台修改（目前為 demo 虛擬公告，涵蓋所有驗收測試情境） |
| `data/carousel.json` | 首頁輪播照片清單（順序、描述、文字），由後台修改 |
| `files/checkin-form.jpg` | 報到單圖片（網頁直接顯示＋下載），目前為範例圖 |
| `files/logo.svg` | 左上角 Logo（兼回首頁連結），目前為範例圖 |
| `files/carousel/` | 首頁輪播的照片檔，由後台上傳；目前為範例圖（`slide-1.svg`～`slide-4.svg`） |
| `vendor/papaparse.min.js` | CSV 解析套件 PapaParse 5.4.1（MIT 授權，錄取名單用），放在 repo 中不依賴 CDN |
| `demo-admitted.csv` | demo 用的虛擬錄取名單，954 筆、4 組（已遮蔽；列順序刻意打亂，含 2／3／4 字姓名、全形字、小寫、「男生」寫法、空白列） |
| `營隊網頁規格書.md` | 規格書 |
| `prototypes/` | 版面比較用的原型（A／B／A+B），已選 A；不影響正式網頁，可刪除 |

---

## 1. 日常維護：要改什麼、去哪裡改

### 1.1 設定區（`index.html` 底部 `<script>` 開頭）

```js
var NOTICES_URL  = 'data/notices.json';          // 公告資料檔（後台修改，一般不用動）
var CAROUSEL_URL = 'data/carousel.json';         // 輪播照片清單（後台修改，一般不用動）
var FORM_URL     = 'https://example.com/form';   // 報名 Google 表單網址
var REG_OPEN     = '2026-11-01T00:00:00+08:00';  // 報名開放時間
var REG_CLOSE    = '2026-12-15T23:59:59+08:00';  // 報名截止時間
var CONTACT_TEXT = 'example@example.com';        // 公告載入失敗時顯示的聯絡方式
var DIGEST_COUNT = 3;                            // 首頁「最新公告」摘要顯示幾則
var NOTICE_VISIBLE_COUNT = 5;                    // 公告分頁先顯示幾則，其餘收在「看更早的公告」
var ADMIT_CSV_URL    = 'demo-admitted.csv';      // 錄取名單 CSV 網址；留空 '' = 顯示「尚未公布」
var ADMIT_SHOW_UNTIL = '2027-01-25T00:00:00+08:00'; // 名單顯示到何時（營隊第一天 00:00）
```

- 時間**一定要保留 `+08:00`**（台灣時間），否則會依訪客裝置時區解讀。
- 報名區塊會依時間自動切換：開放前顯示「報名將於…開放」、開放中顯示按鈕、截止後顯示「報名已截止」。頁面開著時每分鐘重新判斷一次。
- 網頁按鈕只負責顯示，**Google 表單本身也要在截止時關閉「接受回應」**。
- 錄取名單到了 `ADMIT_SHOW_UNTIL` 會自動不顯示，但**試算表仍要手動取消發布並清空**（見 3.6）。

### 1.2 其他內容

| 要改的內容 | 在 `index.html` 中搜尋 |
|---|---|
| 營隊名稱 | 頂部列 `class="brand"`（`aria-label` 與 `class="name"` 兩處）、首頁隱藏標題 `id="camp-title"` |
| Logo | 替換 `files/logo.svg`（建議正方形；改用 PNG 時一併改 `<img src>`） |
| 首頁照片輪播 | **用後台**（見 1.5、第 3 節）；照片清單在 `data/carousel.json` |
| 日期、地點、對象、名額、費用 | `<ul class="facts"`（首頁「報名資訊」卡片） |
| 關於我們 | `data-view="about"`（每段一個 `<div class="rule-group">`，可增減；年表在 `<ul class="timeline">`） |
| 營隊注意事項 | `data-view="camp-rules"`（每組一個 `<div class="rule-group">`，可增減） |
| 報名注意事項 | `data-view="reg-rules"`（最底下的「填寫報名表」大按鈕由程式產生，連到 `FORM_URL`） |
| 報到單說明文字、下載檔名 | `data-view="checkin"` |
| 聯絡信箱、主辦單位 | `<footer class="site-footer">` |
| 主色 | CSS 開頭 `--green`（目前 `#2F6B4F`；改色後確認與白色對比 ≥ 4.5:1） |

> 頁面標題 `<title>` 也有營隊名稱，改名時一起改。

### 1.3 更換報到單

1. 準備**空白**報到單圖片（JPG，建議寬 1200px 以上；repo 是公開的，不可放填好的報到單）
2. 檔名改成 `checkin-form.jpg`，取代 `files/checkin-form.jpg`（GitHub 網頁：進 `files/` → Add file → Upload files，上傳同名檔案即覆蓋）
3. 若圖片比例不同，把 `index.html` 中 `<img class="form-image"` 的 `width`、`height` 改成新圖片的像素尺寸
4. 同一個 `<img>` 的 `alt` 要改成新報到單的內容描述（欄位名稱、要攜帶的物品）
5. 下載後的檔名在 `download="…"` 屬性中修改

### 1.4 分頁

- 網頁上方的選單是分頁，一次只顯示一個分頁。網址 `#xxx` 會顯示所有 `data-view="xxx"` 的區塊，例如 `…/#notices` 顯示公告分頁
- 「首頁」分頁（`data-view="home"`）不在選單中，由左上角 Logo 連回；由三個區塊組成：照片輪播、報名資訊卡片（基本資訊＋報名按鈕，按鈕先連到報名須知分頁）、最新公告摘要
- 新增分頁：複製一個 `<section class="section" data-view="…">` 改成新名稱，並在 `<nav class="site-nav">` 加一個 `href="#新名稱"` 的連結
- 預設分頁是 HTML 中第一個出現的 `data-view`（目前是 `home`）
- 選單文字要短（2～4 個字），手機上才放得下；完整名稱寫在各分頁的標題

### 1.5 公告與輪播照片

公告、輪播照片一律**用後台修改**：開啟 `…/admin.html` 登入（第 3 節）。

- 照片上傳時會自動縮小到寬 1600px、壓成 JPEG，並移除拍攝地點等 EXIF 資訊；橫式照片效果最好（網頁以 16:9、手機 4:3 裁切顯示）
- 輪播張數、圓點、「第 N 張」標示由程式自動產生；只剩一張時不顯示輪播按鈕；沒有照片時整個輪播區不顯示
- **照片中有可辨識的學員時，須先取得家長同意**（repo 與網頁皆公開，刪除後仍留在 git 歷史）

輪播行為：每 5 秒自動換下一張（程式中 `CAROUSEL_INTERVAL_MS`）；滑鼠移入、鍵盤焦點在輪播內或按「暫停輪播」時停止；訪客系統設定「減少動態效果」時預設不自動播放。

### 1.6 用 GitHub 網頁直接修改（不需要裝任何軟體）

1. 到 repo 頁面點 `index.html` → 右上角鉛筆圖示（Edit this file）
2. 用 `Ctrl+F` 找到要改的段落，修改文字
3. 點 **Commit changes…**，寫一句說明（例如「更新報名注意事項第 3 點」）→ **Commit changes**
4. 約 1～2 分鐘後網頁更新（可到 repo 的 **Actions** 分頁看部署進度）

---

## 2. 本機預覽

**不要直接雙擊 `index.html`**：瀏覽器會擋住讀取資料檔，公告區會顯示「公告暫時無法載入」。請開本機伺服器：

```bash
python -m http.server 8790 --bind 127.0.0.1
```

再開啟 <http://127.0.0.1:8790/>。

> 本機的 `127.0.0.1:8765` 有其他專案的 service worker 會攔截頁面，請避開這個埠號。

> 在本機開 `admin.html` 也能登入，但**儲存會直接修改 GitHub 上的 repo**（`admin.html` 設定區 `CONFIG` 指定的那個），不是改本機檔案；改完要 `git pull` 才會同步到本機。

---

## 3. 後台（公告、輪播照片）

後台是 `admin.html`，原理是用每個人自己的 GitHub token，透過 GitHub API 直接修改 repo 裡的 `data/*.json` 與照片，GitHub Pages 再自動重新部署。不需要另外的伺服器或服務。規格見規格書 §7.5。

### 3.1 誰能登入

只有**對這個 repo 有寫入權限**的 GitHub 帳號能登入。

- demo 階段（repo 在個人帳號）：只有 Alice。fine-grained token 只能存取「自己」或「所屬 organization」的 repo，其他人即使被加為 collaborator 也建不出可用的 fine-grained token
- 正式上線：repo 轉移到 **organization** 後，把工作人員加入 organization → repo → Settings → **Collaborators and teams** → 給予 **Write** 權限
- 有人離職：從 organization 移除即可，他的 token 會立即失效

### 3.2 工作人員第一次使用（每人做一次）

1. 註冊 GitHub 帳號；Settings → **Emails** → 勾選 **Keep my email addresses private**（否則後台建立的修改紀錄會公開 email）
2. 到 <https://github.com/settings/personal-access-tokens/new> 建立 **fine-grained token**：
   - Token name：例如「營隊後台」
   - Expiration：建議 **90 天**（到期後重做這一步）
   - Resource owner：選 repo 所屬的帳號／organization
   - Repository access：**Only select repositories** → 只選這個 repo
   - Permissions → Repository permissions → **Contents：Read and write**（其他都不用開）
   - organization 若設定 token 需核准，建立後要等管理者在 organization 設定頁核准
3. 複製 `github_pat_` 開頭的 token（只會顯示一次），開啟 `…/admin.html` 貼上登入
4. 自己的電腦可勾選「在這台裝置記住我」；**共用電腦不要勾**，用完按「登出」

> token 等同密碼：不要傳給別人、不要貼在群組。外流或遺失時到 <https://github.com/settings/personal-access-tokens> 刪除（Delete／Revoke），再重新建立。

### 3.3 操作重點（可轉給工作人員）

- 每個動作（儲存、下架、刪除、調整順序）按下就**立即**改到公開網站，約 1～2 分鐘後生效
- 暫時不想顯示的公告用「下架」，之後可「重新上架」；「刪除」才是真的移除
- 上傳照片要填「照片內容描述」，並勾選「已取得家長同意」
- 不要放任何個人資料（電話、身分證字號、住址等）

### 3.4 查看修改紀錄、還原

- 後台頂部「修改紀錄」連到 repo 的 commits 頁面，每筆後台修改都有紀錄（例如「公告：新增「…」（後台）」）與修改者
- 改錯或誤刪：技術維護者在本機 `git revert <commit>` 後推送，或在 GitHub 網頁打開該 commit 之前的 `data/notices.json` 複製內容貼回
- 刪除的公告與照片**仍在 git 歷史中**，任何人都能翻到。真的需要徹底移除（例如照片未經同意），要改寫 git 歷史，並無法保證沒有人下載過

### 3.5 repo 搬移後

`admin.html` 放在 `*.github.io` 時會自動從網址判斷 repo；在本機或自訂網域使用時，改 `admin.html` 設定區的 `CONFIG`（`owner`、`repo`、`branch`）。

---

### 3.6 錄取名單試算表（公開榜單，不在後台）

錄取名單是本網站唯一含個資的內容，規則見規格書 §6.1、§7.3。**原始的姓名、身分證字號絕對不能進入這份試算表，也不能放進 repo。**

**建立（一次）**

1. **另開一份新的** Google 試算表（不要和其他資料共用），工作表改名為 `錄取名單`
2. 第一列輸入欄位名稱：`梯次`、`性別`、`錄取編號`、`姓名`、`身分證字號`（名稱要完全一樣），並保護第一列：選取第 1 列 → 資料 → **保護工作表和範圍** → 範圍 `錄取名單!1:1` → 限制只有你能編輯
   - `梯次` 名稱要含數字，例如「第一梯」「第2梯」，網頁靠它排順序
   - `性別` 設資料驗證下拉選單：`男`、`女`
   - 列的順序不用整理，網頁會自動依梯次 → 男女 → 錄取編號排序
3. 共用：只加入負責名單的工作人員
4. 檔案 → 設定 → 時區選 **(GMT+08:00) 台北**

**產生遮蔽資料（在私人的報名回覆試算表中做）**

在報名回覆試算表新增兩欄，假設姓名在 B 欄、身分證字號在 C 欄，第 2 列輸入後往下填滿：

```
遮蔽姓名：     =IF(LEN(B2)<=2, LEFT(B2,1)&"○", LEFT(B2,1)&REPT("○",LEN(B2)-2)&RIGHT(B2,1))
遮蔽身分證：   =UPPER(LEFT(TRIM(C2),1))&"******"&RIGHT(TRIM(C2),3)
```

結果會是 `王○明`、`A******123`。複製錄取者的梯次、性別、錄取編號、遮蔽姓名、遮蔽身分證 → 到錄取名單試算表按右鍵 → **選擇性貼上 → 僅貼上值**。

> 不要用 `IMPORTRANGE` 或公式引用報名回覆試算表：發布出去的雖然只有結果，但一旦有人改錯公式或發布整份文件，原始資料就會外流。

**公布**

1. 檔案 → 共用 → **發布到網路** → 第一個下拉選單選 **錄取名單**（不要選「整份文件」）、第二個選 **逗號分隔值 (.csv)** → 發布；確認網址含 `gid=` 與 `single=true`、結尾是 `output=csv`，貼到 `index.html` 的 `ADMIT_CSV_URL`
2. 開網頁的「錄取名單」分頁確認顯示正常；若顯示「資料整理中，暫停顯示」，按 `F12` 看 Console 指出第幾列有問題
3. **若是有未遮蔽的資料被發布**：先到試算表「發布到網路」→ **停止發布**，修正後再重新發布（發布網址會改變，記得更新 `ADMIT_CSV_URL`）

**下架（營隊開始當天）**

1. 錄取名單試算表 → 發布到網路 → **停止發布**
2. 刪除試算表中的名單資料
3. `index.html` 的 `ADMIT_CSV_URL` 改回 `''`

## 4. 部署到 GitHub Pages（demo，個人帳號）

### 4.1 第一次 commit 之前：隱藏 email

repo 是公開的，commit 會顯示作者 email。

1. GitHub → Settings → **Emails** → 勾選 **Keep my email addresses private**，並勾選 **Block command line pushes that expose my email**
2. 複製該頁顯示的 noreply 信箱（`<id>+<帳號>@users.noreply.github.com`）
3. 在本機 repo 資料夾設定：

   ```bash
   git config user.email "<id>+<帳號>@users.noreply.github.com"
   ```

### 4.2 建立 repo 與啟用 Pages

1. GitHub 建立新 repo，可見性選 **Public**（免費方案的 Pages 需要公開）
2. 推上 `index.html`、`admin.html`、`data/`、`vendor/`、`files/`、`demo-admitted.csv`、`README.md`、規格書
3. repo → Settings → **Pages** → Source 選 **Deploy from a branch** → Branch 選 `main`、資料夾 `/ (root)` → Save
4. 1～2 分鐘後網址會出現在同一頁：`https://<帳號>.github.io/<repo>/`

> demo 網址只供內部檢視，**不要對外散布**。repo 之後會搬到公用帳號，網址會改變，GitHub 不會自動轉址。

---

## 5. 常見問題排除

| 狀況 | 可能原因 | 處理方式 |
|---|---|---|
| 公告區顯示「公告暫時無法載入」 | `data/notices.json` 被手動改壞（JSON 格式錯誤，例如少了逗號、引號） | 從 commits 找到改壞的那次，還原該檔（3.4）；之後一律用後台修改 |
| 〃 | 本機直接雙擊開檔 | 依第 2 節開本機伺服器 |
| 首頁沒有輪播 | `data/carousel.json` 沒有照片或格式錯誤；照片路徑不在 `files/carousel/` | 用後台上傳；`F12` → Console 看 `[輪播] 載入失敗` |
| 後台改了但網頁沒變 | GitHub Pages 還在部署 | 等 1～2 分鐘，看 repo 的 **Actions** 分頁；**不要重複新增** |
| 後台登入失敗：token 錯誤或已過期 | token 貼錯、到期或已刪除 | 依 3.2 重新建立 |
| 後台登入失敗：沒有寫入權限 | 帳號沒被加進 repo，或 token 的 Contents 不是 Read and write | 依 3.1、3.2 檢查 |
| 後台登入失敗：找不到 repo | token 沒選到這個 repo，或 `CONFIG` 的 repo 名稱不對 | 重建 token 時選對 repo；依 3.5 檢查 |
| 後台儲存失敗：其他人同時在修改 | 兩人幾乎同時儲存 | 再按一次儲存 |
| 某則公告沒出現 | 已下架，或日期、標題空白 | 在後台「重新上架」或補齊 |
| 公告排序怪怪的 | 日期是手動改成無法辨識的格式（例如「下週一」） | 在後台編輯，重新選日期 |
| 錄取名單顯示「資料整理中，暫停顯示」 | 有一列身分證字號或姓名沒有遮蔽、格式不對（例如星號數量不是 6 個）、性別不是男／女，或有欄位空白 | `F12` → Console 會寫第幾列；未遮蔽的話先停止發布再修正（3.6） |
| 錄取名單顯示「尚未公布」 | `ADMIT_CSV_URL` 是空的，或試算表只有欄位名稱 | 依 3.6 公布 |
| 錄取名單顯示「公告期間已結束」 | 已超過 `ADMIT_SHOW_UNTIL` | 正常現象；需延長就改這個時間 |
| 報名按鈕狀態不對 | `REG_OPEN`／`REG_CLOSE` 設錯或少了 `+08:00` | 依 1.1 修正 |
| 網頁整個沒更新 | GitHub Pages 還在部署 | 看 repo 的 **Actions** 分頁；瀏覽器強制重新整理（`Ctrl+F5`） |

想看詳細錯誤：瀏覽器按 `F12` → Console，會有 `[公告] 載入失敗：…`、`[輪播] 載入失敗：…` 或 `[後台] …` 的訊息。

---

## 6. 正式上線

照規格書 **§13.2 正式上線檢查清單** 逐項完成，重點：

1. repo → Settings → 最下方 **Transfer** 轉移到公用帳號（不要用複製另開的方式）
2. 確認舊的 demo 網址已經 404，並在新位置重新啟用 Pages（依 4.2 第 3 點）
3. 後台：把工作人員加入 organization 並給 repo 寫入權限，每人依 3.2 建立 token；用後台刪除 demo 公告與範例照片、換成正式內容
   在公用帳號建立正式表單，更換 `FORM_URL`
4. 設定正式的 `REG_OPEN`／`REG_CLOSE`，並安排表單在截止時關閉回應
5. 錄取名單：依 3.6 建立試算表；`ADMIT_CSV_URL` 先設為 `''`（尚未公布），`ADMIT_SHOW_UNTIL` 設為營隊第一天 00:00；`demo-admitted.csv` 刪除
6. 刪除 `index.html` 中的 `<meta name="robots" content="noindex">`
7. 全站搜尋 `【範例】` 與 `example.com`，必須一筆都沒有（含 `data/*.json`）；報到單圖片換成正式版（範例圖上印有「【範例】」字樣，搜尋不到，要用眼睛確認）
8. 確認最終網址後才製作對外連結與 QR code

---

## 7. 更新 PapaParse

一般不需要更新。若要更新：

1. 到 <https://cdnjs.com/libraries/PapaParse> 找新版本的 `papaparse.min.js`
2. 下載後取代 `vendor/papaparse.min.js`（保留檔頭的授權聲明）
3. 本機預覽確認錄取名單正常顯示，並把本文件「檔案結構」中的版本號一併更新
