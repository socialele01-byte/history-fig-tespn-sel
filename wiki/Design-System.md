# 設計系統

暖陽紙色系。七個 HTML 檔共用同一組 token，**`:root` 區塊必須逐字相同**。

---

## 核心觀念：顏色分兩層

這套系統把「顏色」拆成兩種角色。搞混這兩層，就會做出白字落在琥珀色上（對比 2.14:1，實質不可讀）那種錯誤。

| 命名 | 角色 | 用在哪 |
|---|---|---|
| `--coral` | **純填色**，不承載文字 | 色條、進度條、圓點、小色塊 |
| `--coral-deep` | **承載文字**：當紙上文字色，或當底色配 `--on-accent` | 內文強調、連結、有字的徽章底 |

`--deep`（藍）原值就已達標，所以沒有 `--deep-deep`。

### 為什麼要有 `-deep`

原本的 accent 色是為「填色好看」挑的，拿來當小字的文字色時對比不足（3.2–3.7:1，門檻是 4.5:1）。`-deep` 是同色相、僅降明度的版本，對**最深的淺色底**（`--tint-sand #FBEAD0`）仍達 4.5:1，因此在任何淺底上都安全。

---

## `.sw-*` 配色對

**填色與其文字色永遠綁在一起**，不要分開指定。這是 gate 41「按鈕文字≈按鈕填色」的結構性防呆。

```css
.sw-coral{background:var(--coral-deep);color:var(--on-accent);}
.sw-amber{background:var(--amber);     color:var(--on-warm);}
.sw-gold {background:var(--gold);      color:var(--on-warm);}
.sw-green{background:var(--green-deep);color:var(--on-accent);}
.sw-teal {background:var(--teal-deep); color:var(--on-accent);}
.sw-sky  {background:var(--sky-deep);  color:var(--on-accent);}
.sw-deep {background:var(--deep);      color:var(--on-accent);}
.sw-close{background:var(--close-deep);color:var(--on-accent);}
```

實測對比（瀏覽器 computed style）：

| class | 底色 | 文字 | 對比 |
|---|---|---|---|
| `.sw-coral` | `#C5371E` | 白 | 5.33 |
| `.sw-amber` | `#F0A03B` | **墨** | 6.39 |
| `.sw-gold` | `#D9A82E` | **墨** | 6.26 |
| `.sw-green` | `#437646` | 白 | 5.36 |
| `.sw-teal` | `#2F766A` | 白 | 5.36 |
| `.sw-sky` | `#327193` | 白 | 5.36 |
| `.sw-deep` | `#356E9E` | 白 | 5.42 |
| `.sw-close` | `#9F592D` | 白 | 5.33 |

**琥珀與金是例外**：它們保留原本的亮色（教學包的識別色，FIG-TESPN 的 I 與 G 兩枚徽章就是這兩色），改用深墨字達標。其餘色相改用加深版配白字。

### 用法

```html
<div class="medal sw-amber">I</div>
```

JS 動態指定時傳 class 名稱，不要傳色值：

```js
{k:"I", cn:"界定問題", sw:"sw-amber", …}
// 樣板：<div class="medal ${s.sw}">${s.k}</div>
```

---

## 完整 token 清單

```css
:root{
  /* 字體堆疊 */
  --font-body:"PingFang TC","Heiti TC","Microsoft JhengHei","Noto Sans TC",system-ui,sans-serif;
  --font-display:Georgia,"Times New Roman",serif;
  --font-num:ui-monospace,"SF Mono",Menlo,Consolas,monospace;

  /* 紙張與墨色 */
  --paper:#FBF3E2;      /* 頁面底 */
  --paper-hi:#FFFCF4;   /* 抬頭漸層上緣 */
  --paper-lo:#FFF6E6;   /* 抬頭漸層下緣 */
  --card:#FFFDF8;       /* 卡片面 */
  --ink:#3A2A20;        /* 主文字 */
  --ink-soft:#7C6758;   /* 次要文字 */
  --line:#EAD9BD;       /* 分隔線 */
  --on-accent:#FFFFFF;  /* 置於深填色上的文字 */
  --on-warm:#3A2A20;    /* 置於 amber／gold 亮填色上的文字 */

  /* 色相・純填色 */
  --coral:#E25B43; --amber:#F0A03B; --gold:#D9A82E; --green:#5FA463;
  --teal:#3F9E8E;  --sky:#3E8CB5;   --deep:#356E9E;  --close:#C9743E;

  /* 色相・承載文字用 */
  --coral-deep:#C5371E; --amber-deep:#9C5C0C;
  --green-deep:#437646; --teal-deep:#2F766A;  --sky-deep:#327193;
  --close-deep:#9F592D;

  /* 淺色調底（皆可承載 --ink 與 --ink-soft） */
  --tint-warm:#FFF7EC;  --tint-sand:#FBEAD0; --tint-cream:#FBF6EC;
  --tint-note:#FBF1DE;  --tint-coral:#FDF0E9; --tint-green:#E5F1E6;
  --tint-teal:#EAF5F2;  --tint-sky:#EEF5F8;   --tint-honey:#FFF7E8;
  --surface-hover:#FFF1D6;
  --surface-input:#FFFEFB;

  /* 邊框與陰影（陰影帶墨色相，不用純黑） */
  --line-coral:#F3C9BC; --line-green:#CFE6D4; --line-sky:#BFDDEC;
  --shadow-soft:rgba(58,42,32,.12);
  --shadow-firm:rgba(58,42,32,.16);

  /* 步驟徽章的淡色底（承載 --ink） */
  --pale-coral:#FBD9CF; --pale-amber:#FCE6C6; --pale-gold:#F6E7B6;
  --pale-green:#D7EBD9; --pale-teal:#CFE9E3;  --pale-sky:#CFE3EF;
  --pale-deep:#D0DCEA;  --pale-close:#F1DBC6;

  /* 抬頭光暈（--amber 22%） */
  --bloom-warm:rgba(240,160,59,.22);
}
```

**每個檔只會用到其中一部分，這是正常的。** 共用 `:root` 的用意是七個檔講同一套詞彙，不是每個檔都要用滿。不要為了「清掉沒用到的」而讓七個檔的 `:root` 產生分歧。

### 本檔專用 token

兩個檔在共用 `:root` 之後另有一個小區塊，放該檔獨有的值：

```css
/* index.html */
:root{ --sidebar-w:280px; }
```

新增這類 token 時，放在共用 `:root` **之後**，並註明「本檔專用」。

---

## 字體

三個堆疊，**任一檔案不得超過 3 種**（目前最多 3，多數是 2）。

| token | 用途 |
|---|---|
| `--font-body` | 內文（CJK 無襯線） |
| `--font-display` | 徽章、序號、標號（Georgia 襯線） |
| `--font-num` | **需要對齊的數字**（等寬） |

### 什麼時候用 `--font-num`

**多位數、需要在直欄中對齊時。** Georgia 的數字是舊體（oldstyle），3/4/5/7/9 帶下降部且寬度不一，排成時刻表會參差不齊。且 Georgia **沒有 lining 替代字集**，加 `font-variant-numeric:lining-nums` 無效——只能換字體。

```css
.tclock{font-family:var(--font-num);font-variant-numeric:tabular-nums;}
```

單一數字或字母的徽章（`.medal`、`.snum`）不受影響，繼續用 `--font-display`。

---

## 禁用的版型

以下是常見的 AI 生成版型特徵，本專案已全數移除，請勿加回：

| 特徵 | 為什麼 | 改用 |
|---|---|---|
| **側邊粗色條**（`border-left:4px+`、`box-shadow:-6px 0 0`） | 2018-SaaS 樣板感 | 整圈細框，或標題旁 11px 小色塊 |
| **三欄等寬 icon-above-heading 卡片** | 每個 LLM 都會吐這個 | 直式清單，徽章與標題同列 |
| **卡中卡**（有邊框的卡裝在有邊框的容器裡） | 無語意的視覺巢狀 | 只留一層containment |
| **Emoji 當圖示**（✨🚀🎲⏱） | 各 OS 長相不同、破壞筆畫語彙 | 移除，靠字體排印分層 |
| **多色光暈**（兩層以上 radial-gradient） | accent 不得超過一色 | 單層 `--bloom-warm` |
| **`transition:.2s`** | 等同 `transition:all` | 明列屬性 |
| **斜體標題** | 最可靠的 AI 特徵之一 | 用字重、accent 色或底線 |

小色塊的標準寫法：

```css
.swatch{display:inline-block;width:11px;height:11px;border-radius:3px;
        margin-right:8px;vertical-align:baseline;}
```
