# ARCHITECTURE.md

[中文](#-中文) | [日本語](#-日本語) | [English](#-english)

---

## 🇨🇳 中文

本文档说明 `index.html` 的内部结构，供维护者修改数据、调整搜索逻辑或扩展新城市时参考。

### 1. 总体结构

单文件应用，无外部依赖，无构建步骤：

```
index.html
├── <style>            界面样式（CSS 变量主题，支持深色模式）
└── <script>
    ├── I18N           界面文案（4语言）
    ├── GOMI_DATA      分类规则 + 物品数据库
    ├── 搜索引擎        kanaNorm / fuzzyScore / levenshtein / searchItems
    ├── 渲染层          renderCategories / renderResults / renderHistory / getSuggestions
    └── 事件绑定        搜索框、语言切换、历史记录、快捷标签
```

### 2. 数据结构

#### 2.1 GOMI_DATA

```js
const GOMI_DATA = {
  cities: [{
    id: "itoshima",
    name: "糸島市",
    categories: [ ... ],   // 6 个分类的定义
    items: [ ... ]         // 物品数据库（205+ 条）
  }]
};
```

#### 2.2 categories（分类定义）

| id | 含义 | 指定袋 | 主题色 |
|---|---|---|---|
| `moeru` | 可燃 | 白袋・红字 | `#c0392b` |
| `moenai` | 不可燃 | 黄袋 | `#c7952b` |
| `recycle` | 资源回收 | 绿袋 | `#3a7d4f` |
| `sodai` | 粗大垃圾 | 需预约+贴票 | `#8e44ad` |
| `not_collected` | 市不收集 | 家电回收法/回收协力店 | `#636e72` |
| `clean_center` | 直接搬入 | クリーンセンター | `#d68910` |

每个分类含 `name / name_zh / name_en / name_ko / bag / bag_zh / bag_en / bag_ko / desc_* / color` 字段。分类主题色同时定义在 CSS `:root` 的 `--cat-*` 变量中，两处需保持一致。

#### 2.3 items（物品条目）

```js
{
  name: "ペットボトル",        // 日语正式名称（必填，作为主键显示）
  zh: "塑料瓶", en: "PET bottle", ko: "페트병",
  cat: "recycle",              // 分类 id（必填）
  alias: ["宝特瓶", "petto"],   // 可选：口语别名/俗称，任意语言
  note_ja: "...", note_zh: "...", note_en: "...", note_ko: "..."  // 处理说明
}
```

- `note_*` 缺失时回退显示 `note_ja`。
- 危险品（锂电池等）在 `note_*` 中以 ⚠️ 开头强调。

### 3. 搜索引擎

流程：`用户输入 → kanaNorm 正规化 → 对每条物品的 _kw 关键词组打分 → 按分数排序`

#### 3.1 kanaNorm（假名正规化）

将片假名逐字符映射为平假名（Unicode 偏移 `-0x60`），同时统一小写。查询词与关键词两侧都做同样处理，实现片假名⇔平假名互通。

#### 3.2 关键词组构建（getAllItems）

每条物品在载入时生成 `_kw` 数组：`[name, zh, en, ko, ...alias]` 全部经过 `kanaNorm(lowercase)`。搜索只对 `_kw` 进行，原始数据不变。

#### 3.3 打分规则（fuzzyScore）

按优先级匹配，返回 `[是否命中, 分数]`：

| 匹配方式 | 分数 |
|---|---|
| 完全相等 | 100 |
| 前缀匹配 | 80 |
| 关键词包含查询词 | 60 |
| 查询词包含关键词 | 50 |
| 编辑距离 ≤ 阈值 | 30 |

#### 3.4 编辑距离的 CJK 限制（editDistanceOk）

- 关键词长度 ≤ 2：跳过（噪音过大）
- 关键词含 CJK 汉字且长度 ≤ 3：跳过（如「毛布」↔「財布」距离仅为 1，会误匹配）
- 其余情况：Levenshtein 距离 ≤ 阈值即命中

#### 3.5 无结果兜底

`renderResults` 在零命中时展示按材质判断的通用规则（`I18N` 中 `fallback_title / fallback_rules / fallback_contact`，`fallback_rules` 以 `|` 分隔逐行渲染），并附市政咨询电话。

### 4. 界面与状态

- **I18N**：4 个语言对象，`t(key, vars)` 取文案并做 `{q}` 占位符替换。语言选择持久化于 `localStorage`。
- **搜索历史**：`localStorage` 保存最近若干条；渲染时所有用户输入均经 `esc()` HTML 转义后再插入 `innerHTML`（防 XSS）。
- **建议下拉**：输入 ≥1 字符时用同一打分函数取前若干条候选。
- **深色模式**：`prefers-color-scheme` 媒体查询覆盖 CSS 变量。

### 5. 如何扩展

**新增物品**：在 `items` 数组末尾追加条目（格式见 2.3），建议同时补充 `alias`。

**修改分类规则**：编辑对应条目的 `cat` 与 `note_*`。规则来源以糸岛市官方年度资料为准。

**新增城市**：
1. 在 `GOMI_DATA.cities` 追加 `{id, name, categories, items}`；
2. 分类体系可与糸岛不同（如福冈市塑料为资源垃圾），`categories` 完全独立定义；
3. 城市切换 UI 已预留（`citySelect` 事件监听）。

**修改主题色**：同步修改两处——CSS `:root` 的 `--cat-*` 变量、`categories` 各条目的 `color` 字段。

### 6. 已知限制

- 编辑距离矩阵使用 `Uint8Array`，超长字符串（>255 距离）不适用——对本场景（短词搜索）无影响
- 语音输入依赖浏览器 Web Speech API，部分移动端浏览器不支持时回退键盘输入
- 数据内嵌于 HTML，更新数据需重新发布整个文件

---

## 🇯🇵 日本語

本ドキュメントは `index.html` の内部構造を説明します。データの修正、検索ロジックの調整、新しい市町村への拡張の際にご参照ください。

### 1. 全体構成

単一ファイルのアプリケーション。外部依存なし、ビルド不要：

```
index.html
├── <style>            UIスタイル（CSS変数テーマ、ダークモード対応）
└── <script>
    ├── I18N           UI文言（4言語）
    ├── GOMI_DATA      分別ルール + 品目データベース
    ├── 検索エンジン    kanaNorm / fuzzyScore / levenshtein / searchItems
    ├── レンダリング    renderCategories / renderResults / renderHistory / getSuggestions
    └── イベント処理    検索ボックス、言語切替、履歴、クイックタグ
```

### 2. データ構造

#### 2.1 GOMI_DATA

```js
const GOMI_DATA = {
  cities: [{
    id: "itoshima",
    name: "糸島市",
    categories: [ ... ],   // 6分類の定義
    items: [ ... ]         // 品目データベース（205件以上）
  }]
};
```

#### 2.2 categories（分類定義）

| id | 意味 | 指定袋 | テーマ色 |
|---|---|---|---|
| `moeru` | もえるごみ | 白袋・赤文字 | `#c0392b` |
| `moenai` | もえないごみ | 黄色袋 | `#c7952b` |
| `recycle` | リサイクル | 緑袋 | `#3a7d4f` |
| `sodai` | 粗大ごみ | 要予約・シール | `#8e44ad` |
| `not_collected` | 市で収集不可 | 家電リサイクル法/回収協力店 | `#636e72` |
| `clean_center` | 直接搬入 | クリーンセンター | `#d68910` |

各分類は `name / name_zh / name_en / name_ko / bag / bag_zh / bag_en / bag_ko / desc_* / color` を持ちます。テーマ色は CSS `:root` の `--cat-*` 変数にも定義されており、両者を一致させる必要があります。

#### 2.3 items（品目エントリ）

```js
{
  name: "ペットボトル",        // 日本語の正式名称（必須・表示上の主キー）
  zh: "塑料瓶", en: "PET bottle", ko: "페트병",
  cat: "recycle",              // 分類id（必須）
  alias: ["宝特瓶", "petto"],   // 任意：俗称・別名（言語不問）
  note_ja: "...", note_zh: "...", note_en: "...", note_ko: "..."  // 出し方の説明
}
```

- `note_*` が欠けている場合は `note_ja` を表示。
- 危険品（リチウムイオン電池など）は `note_*` の先頭に ⚠️ を付けて強調。

### 3. 検索エンジン

処理の流れ：`入力 → kanaNorm 正規化 → 各品目の _kw キーワード群にスコアリング → スコア順にソート`

#### 3.1 kanaNorm（かな正規化）

カタカナを1文字ずつひらがなへ変換（Unicodeオフセット `-0x60`）し、小文字に統一。クエリとキーワードの両方に適用し、カタカナ⇔ひらがなの相互検索を実現。

#### 3.2 キーワード群の構築（getAllItems）

読み込み時に各品目へ `_kw` 配列を生成：`[name, zh, en, ko, ...alias]` を全て `kanaNorm(lowercase)` で処理。検索は `_kw` のみを対象とし、元データは変更しません。

#### 3.3 スコアリング（fuzzyScore）

優先度順にマッチし、`[ヒット有無, スコア]` を返します：

| マッチ方式 | スコア |
|---|---|
| 完全一致 | 100 |
| 前方一致 | 80 |
| キーワードがクエリを含む | 60 |
| クエリがキーワードを含む | 50 |
| 編集距離 ≤ しきい値 | 30 |

#### 3.4 編集距離のCJK制限（editDistanceOk）

- キーワード長 ≤ 2：スキップ（ノイズ過多）
- CJK漢字を含み長さ ≤ 3：スキップ（例：「毛布」↔「財布」は距離1で誤マッチ）
- それ以外：Levenshtein距離がしきい値以下ならヒット

#### 3.5 結果ゼロ時のフォールバック

`renderResults` はヒットゼロの場合、素材別の一般ルール（`I18N` の `fallback_title / fallback_rules / fallback_contact`、`fallback_rules` は `|` 区切りで行ごとに描画）と市の問い合わせ先を表示します。

### 4. UIと状態管理

- **I18N**：4言語オブジェクト。`t(key, vars)` で文言を取得し `{q}` プレースホルダを置換。言語設定は `localStorage` に保存。
- **検索履歴**：`localStorage` に直近数件を保存。描画時、ユーザー入力は全て `esc()` でHTMLエスケープしてから `innerHTML` へ挿入（XSS対策）。
- **サジェスト**：1文字以上の入力で同じスコアリング関数により候補を表示。
- **ダークモード**：`prefers-color-scheme` メディアクエリでCSS変数を上書き。

### 5. 拡張方法

**品目の追加**：`items` 配列の末尾にエントリを追加（形式は2.3参照）。`alias` の追加を推奨。

**分別ルールの修正**：該当エントリの `cat` と `note_*` を編集。ルールの根拠は糸島市の公式年度資料に従うこと。

**市町村の追加**：
1. `GOMI_DATA.cities` に `{id, name, categories, items}` を追加；
2. 分類体系は糸島市と異なってもよい（例：福岡市はプラスチックが資源）。`categories` は完全に独立して定義；
3. 市切替UIは実装済み（`citySelect` のイベントリスナー）。

**テーマ色の変更**：CSS `:root` の `--cat-*` 変数と `categories` 各エントリの `color` の2箇所を同期修正。

### 6. 既知の制限

- 編集距離の行列に `Uint8Array` を使用しており、距離が255を超える超長文字列には非対応——本用途（短語検索）では影響なし
- 音声入力はブラウザの Web Speech API に依存。非対応のモバイルブラウザではキーボード入力にフォールバック
- データはHTMLに内蔵されているため、更新にはファイル全体の再公開が必要

---

## 🇬🇧 English

This document describes the internal structure of `index.html`, for maintainers who want to edit data, tune the search logic, or extend the tool to new cities.

### 1. Overall Structure

A single-file application with no external dependencies and no build step:

```
index.html
├── <style>            UI styles (CSS-variable theming, dark mode support)
└── <script>
    ├── I18N           UI strings (4 languages)
    ├── GOMI_DATA      sorting rules + item database
    ├── Search engine  kanaNorm / fuzzyScore / levenshtein / searchItems
    ├── Rendering      renderCategories / renderResults / renderHistory / getSuggestions
    └── Event binding  search box, language switch, history, quick tags
```

### 2. Data Structures

#### 2.1 GOMI_DATA

```js
const GOMI_DATA = {
  cities: [{
    id: "itoshima",
    name: "糸島市",
    categories: [ ... ],   // definitions of the 6 categories
    items: [ ... ]         // item database (205+ entries)
  }]
};
```

#### 2.2 categories

| id | Meaning | Designated bag | Theme color |
|---|---|---|---|
| `moeru` | Burnable | White bag, red print | `#c0392b` |
| `moenai` | Non-burnable | Yellow bag | `#c7952b` |
| `recycle` | Recyclable | Green bag | `#3a7d4f` |
| `sodai` | Bulky waste | Reservation + sticker | `#8e44ad` |
| `not_collected` | Not collected by city | Appliance Recycling Law / collection stores | `#636e72` |
| `clean_center` | Direct delivery | Clean Center | `#d68910` |

Each category has `name / name_zh / name_en / name_ko / bag / bag_zh / bag_en / bag_ko / desc_* / color` fields. Theme colors are also defined as `--cat-*` CSS variables in `:root`; the two must stay in sync.

#### 2.3 items

```js
{
  name: "ペットボトル",        // official Japanese name (required; display key)
  zh: "塑料瓶", en: "PET bottle", ko: "페트병",
  cat: "recycle",              // category id (required)
  alias: ["宝特瓶", "petto"],   // optional: colloquial names, any language
  note_ja: "...", note_zh: "...", note_en: "...", note_ko: "..."  // disposal notes
}
```

- Missing `note_*` fields fall back to `note_ja`.
- Hazardous items (e.g., lithium-ion batteries) are emphasized with a leading ⚠️ in `note_*`.

### 3. Search Engine

Pipeline: `user input → kanaNorm normalization → score each item's _kw keyword set → sort by score`

#### 3.1 kanaNorm

Maps katakana to hiragana character by character (Unicode offset `-0x60`) and lowercases. Applied to both queries and keywords, enabling katakana ⇔ hiragana interoperability.

#### 3.2 Keyword set construction (getAllItems)

On load, each item gets a `_kw` array: `[name, zh, en, ko, ...alias]`, all passed through `kanaNorm(lowercase)`. Search operates only on `_kw`; original data is untouched.

#### 3.3 Scoring (fuzzyScore)

Matches by priority and returns `[hit, score]`:

| Match type | Score |
|---|---|
| Exact match | 100 |
| Prefix match | 80 |
| Keyword contains query | 60 |
| Query contains keyword | 50 |
| Edit distance ≤ threshold | 30 |

#### 3.4 CJK restriction on edit distance (editDistanceOk)

- Keyword length ≤ 2: skipped (too noisy)
- Contains CJK characters and length ≤ 3: skipped (e.g., 「毛布」↔「財布」 differ by distance 1 and would mismatch)
- Otherwise: hit if Levenshtein distance ≤ threshold

#### 3.5 Zero-result fallback

When nothing matches, `renderResults` shows generic material-based rules (`fallback_title / fallback_rules / fallback_contact` in `I18N`; `fallback_rules` is `|`-separated and rendered line by line) plus the city's contact number.

### 4. UI & State

- **I18N**: four language objects; `t(key, vars)` retrieves strings and substitutes `{q}` placeholders. Language choice persists in `localStorage`.
- **Search history**: recent queries stored in `localStorage`; all user input is HTML-escaped via `esc()` before insertion into `innerHTML` (XSS protection).
- **Suggestions dropdown**: candidates ranked by the same scoring function from the first character typed.
- **Dark mode**: CSS variables overridden via the `prefers-color-scheme` media query.

### 5. How to Extend

**Add items**: append entries to the end of the `items` array (format in 2.3); adding `alias` is recommended.

**Change sorting rules**: edit the entry's `cat` and `note_*`. Rules should follow Itoshima City's official annual materials.

**Add a city**:
1. Append `{id, name, categories, items}` to `GOMI_DATA.cities`;
2. The category system may differ from Itoshima's (e.g., Fukuoka City treats plastics as recyclable); `categories` is fully independent;
3. The city-switch UI is already wired (`citySelect` event listener).

**Change theme colors**: update both places in sync — the `--cat-*` CSS variables in `:root` and the `color` field of each `categories` entry.

### 6. Known Limitations

- The edit-distance matrix uses `Uint8Array`, so it doesn't support very long strings (distance > 255) — irrelevant for this short-query use case
- Voice input depends on the browser's Web Speech API; unsupported mobile browsers fall back to keyboard input
- Data is embedded in the HTML, so updating data requires republishing the whole file
