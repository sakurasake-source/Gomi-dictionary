# 🗑️ Gomi-dictionary｜糸島市ごみ分別辞典

[中文](#-中文) | [日本語](#-日本語) | [English](#-english)

Multilingual garbage sorting dictionary for **Itoshima City, Fukuoka, Japan** — built for international students.

> 🔗 **Live Demo**: https://sakurasake-source.github.io/Gomi-dictionary/

---

## 🇨🇳 中文

面向留学生的多语言垃圾分类查询工具（日/中/英/韩）。

### 功能特点

- 🔍 **模糊搜索**：四语言关键词，平假名・片假名自动互通（搜「ぺっとぼとる」也能命中「ペットボトル」）
- 🏷️ **别名匹配**：收录口语叫法（充电宝、快递盒、保温杯…），不必知道标准日语名称
- 📦 **205+ 常见物品**：覆盖厨余、家电、家具、化妆品、搬家场景
- 🎨 **袋色对应实物**：可燃（白袋红字）/ 不可燃（黄袋）/ 资源回收（绿袋）
- 🧭 **材质兜底提示**：搜不到时按材质给出通用判断规则
- ⚠️ **安全警告**：锂电池、充电宝等严禁入袋物品有醒目提示
- 📱 **零安装**：单 HTML 文件，浏览器直接打开，可离线使用

### 使用方法

直接访问上方在线链接，或下载 `index.html` 用浏览器打开。

### 数据来源与免责声明

分类规则依据糸岛市官方资料整理（见下方 Data Source）。本工具为个人整理的非官方参考，分类规则可能随年度更新，不确定时请以市官方资料为准，或咨询糸島市環境政策課（092-332-2068）。

---

## 🇯🇵 日本語

留学生向けの多言語ごみ分別検索ツール（日・中・英・韓）。

### 特徴

- 🔍 **あいまい検索**：4言語対応、ひらがな・カタカナ自動変換
- 🏷️ **別名対応**：日常的な呼び方でも検索可能（正式名称を知らなくてもOK）
- 📦 **205品目以上**：生ごみ・家電・家具・化粧品・引越しでよく出るものを収録
- 🎨 **袋の色が実物と対応**：もえるごみ（白袋・赤文字）／もえないごみ（黄色袋）／リサイクル（緑袋）
- 🧭 **素材別の判断ガイド**：見つからない場合は素材から判断できるヒントを表示
- ⚠️ **安全警告**：リチウムイオン電池・モバイルバッテリー等の袋入れ禁止品目を明示
- 📱 **インストール不要**：HTMLファイル1つ、ブラウザで開くだけ・オフライン利用可

### 使い方

上記のリンクにアクセスするか、`index.html` をダウンロードしてブラウザで開いてください。

### データ出典・免責事項

糸島市の公式資料をもとに作成した非公式の参考ツールです。分別ルールは年度により変更される場合があります。ご不明な点は市の公式資料をご確認いただくか、糸島市環境政策課（092-332-2068）へお問い合わせください。

---

## 🇬🇧 English

A multilingual garbage sorting dictionary (JA/ZH/EN/KO) for international students living in Itoshima City.

### Features

- 🔍 **Fuzzy search** in 4 languages, with automatic hiragana ↔ katakana normalization
- 🏷️ **Alias matching** — search by everyday names, no need to know the official Japanese term
- 📦 **205+ items** covering kitchen waste, appliances, furniture, cosmetics, and moving-out scenarios
- 🎨 **Bag colors match reality**: burnable (white bag, red print) / non-burnable (yellow) / recycling (green)
- 🧭 **Material-based fallback** — generic sorting rules shown when no item matches
- ⚠️ **Safety warnings** for items banned from bags (lithium-ion batteries, power banks)
- 📱 **Zero install** — a single HTML file, works offline in any browser

### How to Use

Visit the live link above, or download `index.html` and open it in a browser.

### Data Source & Disclaimer

Sorting rules are compiled from official Itoshima City materials. This is an unofficial personal reference; rules may change each fiscal year. When in doubt, consult official city resources or the Environment Policy Division (092-332-2068).

---

## 📖 Data Source

- [糸島市 家庭ごみの分け方一覧表（50音）](https://www.city.itoshima.lg.jp/s011/010/020/020/070/20210628224354.html)
- [糸島市 家庭ごみの正しい出し方（ごみカレンダー）](https://www.city.itoshima.lg.jp/s011/010/020/020/050/20210305144610.html)

## 🛠️ Tech

Pure HTML + CSS + Vanilla JavaScript. No frameworks, no dependencies, no build step. See [ARCHITECTURE.md](ARCHITECTURE.md) for details.

## 🤝 Contributing

Found a sorting error or want to add items? Open an [Issue](../../issues) or a Pull Request.

```js
{name:"日本語名",zh:"中文",en:"English",ko:"한국어",cat:"moeru",
 alias:["别名1","alias2"],
 note_ja:"...",note_zh:"...",note_en:"...",note_ko:"..."}
```

`cat`: `moeru` / `moenai` / `recycle` / `sodai` / `not_collected` / `clean_center`

## 📄 License

[MIT](LICENSE)
