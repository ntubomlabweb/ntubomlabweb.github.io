# AGENTS.md — BOmLab 實驗室網站維護指引（給 AI Agent）

> 本檔案是給 VS Code 內 AI agent（GitHub Copilot Agent、Claude Code、Cursor 等）讀取的專案說明。
> 使用者（實驗室網管）會用中文下指令，例如「新增碩一成員王小明」「把這張照片加到花絮」。
> 你的任務：依本檔規則直接修改檔案、驗證、commit 並 push，讓網站自動上線。

---

## 0. 專案概要

- 網站：國立臺灣大學 生物機電工程學系「生物光機電實驗室 BOmLab」
- 正式網址：https://ntubomlabweb.github.io
- Repo：`ntubomlabweb.github.io`，部署分支 **`main`**（GitHub Pages，push 後 1–2 分鐘自動上線）
- **無建置步驟**：沒有 npm、沒有 bundler、沒有 build。改 `index.html` 存檔即是成品。

### 檔案結構

```
/
├── index.html        ← 唯一需要編輯的檔案（版面 + 所有資料）
├── support.js        ← 執行環境，禁止修改
├── image-slot.js     ← 執行環境，禁止修改
├── AGENTS.md         ← 本檔
└── assets/
    ├── bomlab-logo.png   ← 實驗室 LOGO / favicon
    ├── ntu-crest.png     ← 台大校徽
    └── photos/           ← 所有照片（.jpg）
```

---

## 1. 標準工作流程（每個指令都照做）

1. `git pull origin main` — 先同步。
2. 依第 3 節找到對應區塊，進行**最小幅度修改**。不要重排、不要重新格式化、不要「順便改善」其他地方。
3. 若有新照片：依第 4 節壓縮、命名、放入 `assets/photos/`。
4. 執行第 5 節驗證。
5. `git add -A && git commit -m "<中文描述，例：新增碩一成員王小明>" && git push origin main`
6. 回報使用者：改了什麼、哪些資料缺漏（例如沒給照片、沒給研究主題）。

**若使用者沒提供必要資訊**（例如沒給英文姓名、學期、日期），先詢問；不要自行杜撰內容。
**照片檔若使用者只放在對話中**，請使用者把檔案放進 repo 的任意位置（例如 `uploads/`）並告訴你路徑。

---

## 2. index.html 技術架構（必讀，否則會改壞）

`index.html` 是一個「Design Component」檔案，由 `support.js` 在瀏覽器中執行（React 底層）。結構：

```
<x-dc>
  ...模板 HTML（版面）...
</x-dc>
<script type="text/x-dc" data-dc-script data-props="...">
class Component extends DCLogic {
  ...
  renderVals() {
    const homeSlides = [...];   ← 資料清單都在這裡
    ...
    return { ... };             ← 傳給模板的值
  }
}
</script>
```

### 模板語法規則（修改 `<x-dc>` 內的 HTML 時）

- `{{ path }}` 只能放**點路徑**（`{{ mb.name }}`），**不能放運算式**（`{{ a + b }}`、`{{ !x }}`、`{{ fn() }}` 都會失效）。需要計算的值請在 `renderVals()` 算好再 return。
- 迴圈：`<sc-for list="{{ items }}" as="item">…</sc-for>`；條件：`<sc-if value="{{ flag }}">…</sc-if>`。
- **只使用 inline style**（`style="..."`）。不要新增 `<style>` 區塊、CSS class 或外部 CSS。
- hover 效果用 `style-hover="..."` 屬性。
- 事件寫成 `onClick="{{ handler }}"`，handler 必須在 `renderVals()` return 中。
- 所有非 void 元素都要明確關閉；屬性值一律雙引號。
- 不要在 `<x-dc>` 內加 `<script>`。

### 邏輯區（`class Component`）規則

- 純 JavaScript，無 TypeScript、無 `import`/`export`。
- class 名必須是 `Component`。
- 資料清單是 JS 陣列，字串用**單引號**。字串內的單引號要跳脫 `\'`。論文標題請用雙引號 `"…"` 包在單引號字串內。
- 每筆物件結尾保留逗號。

### 頁面路由

網址 hash 控制頁面：`#home`、`#prof`、`#research`、`#members`、`#member/<slot>`、`#courses`、`#photos`、`#pubs`、`#contact`。瀏覽器上一頁可用。不需更動。

---

## 3. 內容位置與資料格式

用搜尋（grep）定位，**不要依賴行號**（行號會變）。

| 內容 | 搜尋字串 | 位置 |
|---|---|---|
| 最新消息 | `<!-- news -->` | 模板 |
| 首頁幻燈片 | `const homeSlides = [` | 邏輯 |
| 教授經歷學歷 | `const timeline = [` | 邏輯 |
| 研究領域 | `const researchTopics = [` | 邏輯 |
| 成員（在學） | `const allGroups = [` | 邏輯 |
| 成員（畢業） | `allGroups.push(` / `GRADUATED` | 邏輯 |
| 開設課程 | `const courses = [` | 邏輯 |
| 活動花絮 | `const events = [` | 邏輯 |
| 期刊論文 | `const pubJournals = [` | 邏輯 |
| 專書章節 | `const pubBooks = [` | 邏輯 |
| 研討會論文 | `const pubConfs = [` | 邏輯 |
| 聯絡資訊/地圖 | `<!-- ======= CONTACT ======= -->` | 模板 |
| 頁尾 | `<!-- ======= FOOTER ======= -->` | 模板 |

### 3-1. 成員 `allGroups`

```js
const WIX = 'assets/photos/';   // 照片路徑前綴（名稱是歷史遺留，勿改）
const allGroups = [
  { title: '博士班', en: 'PH.D. STUDENTS', items: [ ... ]},
  { title: '碩士班・碩二', en: 'GRADUATE — SECOND YEAR', items: [ ... ]},
  { title: '碩士班・碩一', en: 'GRADUATE — FIRST YEAR', items: [ ... ]},
  { title: '大學部專題生', en: 'B.S. STUDENTS', items: [ ... ]},
];
if (showGraduated) {
  allGroups.push({ title: '畢業成員', en: 'GRADUATED MEMBERS', items: [ ... ]});
  allGroups.push({ title: '畢業大學部', en: 'GRADUATED B.S. STUDENTS', items: [ ... ]});
}
```

成員物件：

```js
{ name: '王小明', en: 'Xiao-Ming Wang', topic: 'Fluorescence Lifetime', slot: 'mb-xmwang', img: WIX + 'mb-xmwang.jpg' },
```

- `slot`：**全站唯一**，格式 `mb-<名縮寫><姓>` 全小寫（例：Bo-Yi Liu → `mb-byliu`）。同時是個人頁網址，**已存在成員的 slot 不要改**（會讓舊連結失效）。新增前 grep 確認不重複。
- `img`：無照片時填 `''`，網站會自動顯示姓名首字方塊。
- `topic`：未知就填 `''`。
- 每位成員自動擁有個人介紹頁（`#member/<slot>`），不需另外建立。
- 例外：分組設有 `noPage: true`（目前為 `畢業大學部`）時，該組成員只列在成員頁、卡片不可點、沒有個人頁（直接開網址會導回 `#members`）。學年升級時把大學部成員搬進此組即自動套用，不要刪掉這個屬性。

**學年升級（常見指令「學年更新」「XX 畢業了」）**：用「剪下→貼上」整行搬移，不要改內容：
- 碩二 → `畢業成員` 的 items 開頭
- 碩一 → 碩二
- 大學部 → `畢業大學部` 的 items 開頭
- 某分組清空時，刪除整個 `{ title: ..., items: [ ]},`；有新成員再加回（保持上表順序）。

### 3-2. 活動花絮 `events`

```js
{ sem: '115-1', label: '迎新活動', slot: 'ev-115-1-welcome', img: 'assets/photos/115-1-welcome.jpg' },
```

- 新增放**陣列最上面**（新→舊排序）。
- `sem`：學期（`115-1`）或學年（`115`），依使用者給的。
- `slot`：唯一字串。
- 花絮頁為三欄瀑布流，照片保持原比例，不會裁切。

### 3-3. 首頁幻燈片 `homeSlides`

```js
{ slot: 'home-slide-11', cap: '115-1 迎新活動', img: 'assets/photos/115-1-welcome.jpg' },
```

- **張數必須與模板的寬度設定一致**。目前 10 張，模板中有：
  - `width:1000%`（= 張數 × 100%）
  - `flex:0 0 10%;width:10%`（= 100 ÷ 張數 %）
  - `hint-placeholder-count="10"`
- **規則：首頁幻燈片 = `events`（花絮）最上面（最新）的 10 張，順序相同。**
  - 每次 `events` 有新增或刪除，都要重新同步 `homeSlides`：新照片依花絮順序插入，超出 10 張的最舊幾張從 `homeSlides` 移除（花絮頁保留不動）。
  - 仍留在幻燈片中的照片**保留原本的 `slot`**；新加入的照片給新的唯一 slot（`home-slide-<下一個未用過的數字>`，grep 確認不重複）。
  - 只想加首頁、不加花絮的照片不存在；若使用者要求「放首頁」，就先加到 `events` 最上面，再同步。
- 若使用者明確要改張數，三處都要一起改（例：12 張 → `1200%`、`8.3333%`、`12`），並改為同步最新 N 張。
- `cap` 格式：`<sem> <label>`，直接沿用該照片在 `events` 中的學期與活動名（含英文活動名）。label 含單引號時，該字串改用雙引號包住。
- 同步後驗證：`homeSlides` 的照片順序與 `events` 前 10 筆一致。

### 3-4. 最新消息（模板，`<!-- news -->` 之後）

兩種格式，新消息插在最上方（`<div style="display:flex;flex-direction:column">` 的第一個子元素位置）：

**圖文消息**（複製既有的圖文區塊改內容）：

```html
<div style="display:grid;grid-template-columns:400px 1fr;gap:32px;padding:26px 0 30px;border-top:2px solid #1c2735;align-items:start">
  <div>
    <div style="aspect-ratio:3/2;border-radius:8px;overflow:hidden;box-shadow:0 12px 30px rgba(22,40,62,.16)">
      <img loading="lazy" src="assets/photos/xxx.jpg" alt="標題" style="width:100%;height:100%;object-fit:cover;display:block">
    </div>
    <div style="font-family:'IBM Plex Mono',monospace;font-size:11px;letter-spacing:.14em;color:#a3813a;margin-top:10px">英文分類標籤</div>
  </div>
  <div>
    <span style="font-family:'IBM Plex Mono',monospace;font-size:13px;color:#a3813a">2026.10.03</span>
    <div style="font-family:'Noto Serif TC',serif;font-weight:900;font-size:26px;line-height:1.45;margin:8px 0 12px">標題</div>
    <p style="font-size:15px;line-height:2;color:#4d5a6b;margin:0 0 18px">內文</p>
  </div>
</div>
```

**純文字消息**：

```html
<div style="display:grid;grid-template-columns:100px 1fr auto;gap:20px;align-items:baseline;padding:18px 0;border-top:1px solid #cbc8b8">
  <span style="font-family:'IBM Plex Mono',monospace;font-size:13px">2026.10</span>
  <span style="font-size:15px;font-weight:500;line-height:1.7;color:#4d5a6b">消息內容</span>
  <span></span>
</div>
```

- 日期格式：`YYYY.MM.DD` 或 `YYYY.MM`。
- 第一則若為圖文消息，`border-top` 用 `2px solid #1c2735`；其餘純文字用 `1px solid #cbc8b8`。
- 消息內文**照使用者給的文字原樣放入**，不要改寫或潤飾。
- 刪除舊消息時確保 `<div>` 與 `</div>` 成對。

### 3-5. 論文 `pubJournals` / `pubBooks` / `pubConfs`

```js
{ num: '01', text: 'A. Author and H.-M. Wu*, "Title," Journal, vol. 1, pp. 1–10, 2026.', note: '' },
```

- 新增放**最上面**。
- `pubJournals`、`pubBooks`：`num` 需手動重編（新增後全部 +1，格式兩位數 `'01'`）。
- `pubConfs`：`num` 一律填 `'00'`，程式會自動編號。
- `text`：**完全照使用者提供的引用格式**，不改大小寫、不改標點。
- `note`：標籤（如 `'通訊作者'`），沒有就 `''`。

### 3-6. 其他

- `courses`：`{ num, zh, en, sem, url, slot, extra, extraUrl, img }`，複製既有一筆修改。
- `researchTopics`：`{ num, zh, en, slot, ph, img }`。
- `timeline`：`{ year, role, org }`。
- 聯絡頁 Google 地圖為 `<iframe src="https://www.google.com/maps/embed?pb=...">`；更換地點時只換 `src`。若使用者給的是一般 Google Maps 連結（非 embed），可用 `https://www.google.com/maps?q=<緯度>,<經度>&z=17&output=embed`。

---

## 4. 照片處理規則

所有新照片**必須**處理後才加入：

| 項目 | 規格 |
|---|---|
| 格式 | JPG（品質約 82） |
| 尺寸 | 最長邊 ≤ 1600px（成員大頭照 ≤ 800px） |
| 大小 | 目標 ≤ 400 KB |
| 檔名 | 小寫英數 + 連字號，無中文、空格、括號。例：`115-1-welcome.jpg`、`mb-byliu.jpg` |
| 位置 | `assets/photos/` |

依環境擇一執行：

```bash
# macOS 內建
sips -s format jpeg -s formatOptions 82 -Z 1600 input.png --out assets/photos/115-1-welcome.jpg

# ImageMagick（Windows / Linux / macOS）
magick input.png -resize "1600x1600>" -quality 82 assets/photos/115-1-welcome.jpg

# Python Pillow
python -c "from PIL import Image; im=Image.open('input.png').convert('RGB'); im.thumbnail((1600,1600)); im.save('assets/photos/115-1-welcome.jpg', quality=82, optimize=True)"
```

- 有透明背景的 PNG 轉 JPG 時，背景填白。
- 處理完刪除暫存的原始檔（不要把原始大檔 commit 進 repo）。
- GitHub Pages 檔名**區分大小寫**，`index.html` 中的路徑必須與檔名完全一致。
- LOGO 類（`bomlab-logo.png`、`ntu-crest.png`）保持 PNG，不要轉檔。

---

## 5. 修改後驗證（commit 前必做）

1. **路徑檢查**：`index.html` 中每個 `assets/photos/...` 路徑都存在對應檔案。
   ```bash
   grep -oE "assets/photos/[A-Za-z0-9._-]+" index.html | sort -u | while read f; do [ -f "$f" ] || echo "MISSING: $f"; done
   ```
2. **slot 唯一性**：
   ```bash
   grep -oE "slot: '[^']+'" index.html | sort | uniq -d
   ```
   應無輸出。
3. **JS 語法檢查**（有 Node.js 時）：
   ```bash
   node -e "const s=require('fs').readFileSync('index.html','utf8');const m=s.match(/<script type=\"text\/x-dc\"[^>]*>([\s\S]*?)<\/script>/);new Function('DCLogic','React',m[1]);console.log('JS OK')"
   ```
4. **幻燈片張數**：`homeSlides` 筆數與模板 `width:1000%` 等三處一致。
5. 若可行，用本機伺服器預覽（`python -m http.server 8000` 或 VS Code Live Server），打開 `http://localhost:8000/#home`、`#members`、`#photos`、`#pubs` 確認不是空白頁。

任何一項失敗都先修正再 commit。

---

## 6. 禁止事項

- ❌ 修改或刪除 `support.js`、`image-slot.js`
- ❌ 新增 build 工具、npm 套件、框架、外部 CSS
- ❌ 新增 `<style>` 類別樣式或 CSS class（只用 inline style）
- ❌ 在 `{{ }}` 中寫運算式
- ❌ 改寫使用者提供的消息文字或論文引用
- ❌ 杜撰成員資料、研究主題、日期
- ❌ 更改既有成員的 `slot`
- ❌ 大幅重排版面、更換配色字型（除非使用者明確要求重新設計）
- ❌ commit 未壓縮的大照片
- ❌ push 到 `main` 以外的分支（除非使用者要求）

---

## 7. 設計規範（新增元素時沿用）

| 用途 | 值 |
|---|---|
| 主色 海軍藍（標題區/導覽列背景） | `#16283e` |
| 頁尾背景 | `#101a29` |
| 頁面背景 米白 | `#f1efe8` |
| 主文字 | `#1c2735` |
| 次要文字 | `#4d5a6b` |
| 金色強調（日期、標籤） | `#a3813a`（淺：`#d9a86b`） |
| 連結藍 | `#2c4a6e` |
| 分隔線 | `#cbc8b8` |
| 中文標題字型 | `'Noto Serif TC', serif`（weight 900） |
| 內文字型 | `'Noto Sans TC'` |
| 日期/標籤字型 | `'IBM Plex Mono', monospace` |

---

## 8. 指令範例與預期動作

| 使用者指令 | 你應該做的 |
|---|---|
| 「新增碩一成員 王小明 Xiao-Ming Wang」 | `allGroups` 碩一 items 加一筆，`slot: 'mb-xmwang'`，`img: ''`；回報缺照片與研究主題 |
| 「王小明的照片是 uploads/wang.png」 | 壓縮為 `assets/photos/mb-xmwang.jpg`，更新 `img: WIX + 'mb-xmwang.jpg'` |
| 「學年更新：碩二畢業、碩一升碩二，新碩一有 A、B」 | 依 3-1 搬移，再新增 A、B |
| 「把 uploads/party.jpg 加到花絮，115-1 聖誕派對」 | 壓縮→`115-1-christmas.jpg`；`events` 最上面新增；`homeSlides` 同步為花絮最新 10 張 |
| 「這張也放首頁幻燈片」 | 確認已在 `events`（沒有就加到最上面），再將 `homeSlides` 同步為花絮最新 10 張 |
| 「新增最新消息：2026.12 恭喜 XXX 獲獎」 | news 區最上方插入純文字消息，文字原樣 |
| 「新增研討會論文：<引用>」 | `pubConfs` 最上面新增，`num: '00'` |
| 「新增期刊論文：<引用>」 | `pubJournals` 最上面新增 `'01'`，其餘重編號 |
| 「網站變空白了」 | 跑第 5 節驗證找錯；必要時 `git diff` / `git log` 比對上次正常版本 |
| 「改回上一版」 | `git revert HEAD`（不要用 `reset --hard` + force push） |
