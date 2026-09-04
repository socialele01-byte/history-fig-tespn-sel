# 品質基準

三條硬底線：**對比**、**鍵盤焦點**、**行動裝置**。每一條都附了可執行的驗證方式——改完請實測，不要目測。

---

## 一、對比

| 文字類型 | 門檻 |
|---|---|
| 內文（< 24px 一般，或 < 18.66px 粗體） | WCAG **4.5:1** |
| 大字（≥ 24px，或 ≥ 18.66px 粗體） | WCAG **3:1** |
| 焦點框、圖示、UI 元件 | **3:1** |

**目前狀態：七頁、952 個元素、0 失敗。**

### 常見陷阱

- **亮暖色配白字。** `--amber` 配白只有 2.14、`--gold` 只有 2.19，連大字門檻都不到。用 `.sw-amber` / `.sw-gold`（深墨字）。
- **小尺寸膠囊。** `.chip`、`.fmt`、`.tag` 這類 0.7rem 標籤算「內文」，需要 4.5:1，不能用 3:1 蒙混。
- **半透明疊色。** `rgba(255,255,255,.25)` 疊在珊瑚上約 3.3:1。要反白就用實色反白（淺底 + 該段深色字）。
- **`--ink-soft` 落在最深的淺底上。** `#7C6758` 對 `--tint-sand` 是 4.52，剛好過。再深的底就不行了。

### 怎麼驗

用瀏覽器開頁面，貼進 console。它會走訪 DOM、逐一計算實際疊出來的背景色，回報所有未達門檻的元素：

```js
(function(){
  const px=c=>{const m=c.match(/[\d.]+/g);return m?m.map(Number):null};
  const lin=v=>{v/=255;return v<=0.03928?v/12.92:Math.pow((v+0.055)/1.055,2.4)};
  const L=a=>0.2126*lin(a[0])+0.7152*lin(a[1])+0.0722*lin(a[2]);
  const CR=(a,b)=>{const x=L(a),y=L(b);return (Math.max(x,y)+0.05)/(Math.min(x,y)+0.05)};
  const over=(f,b)=>{const a=f[3]===undefined?1:f[3];return [0,1,2].map(i=>f[i]*a+b[i]*(1-a))};
  function bgOf(el){let n=el,st=[];while(n){const c=px(getComputedStyle(n).backgroundColor);
    if(c&&(c[3]===undefined||c[3]>0))st.push(c);n=n.parentElement}
    let o=[255,255,255];for(let i=st.length-1;i>=0;i--)o=over(st[i],o);return o}
  document.querySelectorAll('details').forEach(d=>d.open=true);
  document.querySelectorAll('.stage').forEach(d=>d.classList.add('active'));
  const out=[];
  document.querySelectorAll('*').forEach(el=>{
    const t=[...el.childNodes].filter(n=>n.nodeType===3&&n.textContent.trim())
              .map(n=>n.textContent.trim()).join('').slice(0,18);
    if(!t)return;
    const cs=getComputedStyle(el);
    if(cs.visibility==='hidden'||cs.display==='none'||+cs.opacity===0)return;
    const fs=parseFloat(cs.fontSize),fw=parseInt(cs.fontWeight)||400;
    const need=(fs>=24||(fs>=18.66&&fw>=700))?3:4.5;
    const bg=bgOf(el), r=CR(over(px(cs.color),bg),bg);
    if(r<need-0.005)out.push({el:el.tagName+'.'+(el.className||''),txt:t,got:+r.toFixed(2),need});
  });
  return {checked:document.querySelectorAll('*').length,fails:out.length,list:out};
})()
```

> 展開 `<details>` 與啟用所有 `.stage` 是必要的——隱藏內容也要檢查。

---

## 二、鍵盤焦點

**每一個可聚焦元素都必須有可見的焦點框。** 目前四個有互動的檔、58 個可聚焦元素，覆蓋率 100%。

統一寫法（勿自創）：

```css
X:focus-visible{outline:3px solid var(--sky);outline-offset:2px;}
```

`--sky` 在本系統所有淺色底上都達 3:1（3.34–3.67）。

`outline-offset:2px` 不只是留白——它讓焦點框長在元素**外面**、貼著頁面底色。所以即使元素本身是金色（例如鷹架切換成 `--gold` 的按鈕，`--sky` 對 `--gold` 只有 1.71），焦點框仍然清晰。**不要把 offset 改成 0 或負值。**

### 特別注意

拿掉了原生記號的 `<summary>`（`list-style:none` + `::-webkit-details-marker{display:none}`）**更需要**焦點框——使用者除此之外沒有任何線索。

### 怎麼驗

`:focus-visible` 只在鍵盤操作時生效，`el.focus()` 測不出來。用這段反查覆蓋率：

```js
(function(){
  const rules=[];
  for(const ss of document.styleSheets){let rs;try{rs=ss.cssRules}catch(e){continue}
    for(const r of rs){if(r.selectorText&&r.selectorText.includes(':focus-visible'))rules.push(r.selectorText)}}
  const f=[...document.querySelectorAll('a[href],button,input,select,textarea,summary,[tabindex]:not([tabindex="-1"])')];
  const un=f.filter(el=>!rules.some(sel=>sel.split(',').some(p=>{
    try{return el.matches(p.trim().replace(/:focus-visible/g,''))}catch(e){return false}})));
  return {rules, focusable:f.length, uncovered:un.map(e=>e.tagName+'.'+e.className)};
})()
```

再實按幾次 Tab，確認 `document.activeElement` 真的有 outline。

---

## 三、行動裝置

**必須在 320 / 375 / 414 / 768 px 都不橫向溢出。** 320px 是硬底線。

各檔斷點：

| 檔案 | 斷點 | 主要處理 |
|---|---|---|
| `index.html` | 780px | 側欄轉為上方橫幅 |
| `00_教學包總覽` | 600px | 情境列改直式、檔案卡欄名改上方標籤 |
| `思考鷹架` | 640px | 反思雙欄收合、徽章縮小 |
| `骰子抉擇` | 600px | 標題縮字、版本徽章改靜態 |
| `教師引導提問單` | 600px | 標題縮字、SEL 膠囊改左對齊 |
| `工作坊流程小抄` | 640px | **表格改堆疊卡片** |
| `工作坊講稿` | 無 | 單欄長文，實測 320px 0 溢出 |

### 已知的坑

- **`overflow-x` 用 `clip` 不要用 `hidden`。** `hidden` 會讓子元素的 `position:sticky` / `fixed` 失效。
- **`flex:1 0 100%` 不能再加 `margin-left`。** margin 在盒外，會溢出。用 `padding-left`（本專案有 `box-sizing:border-box`）。
- **`white-space:nowrap` 會鎖死表格最小寬度。** 小抄的時刻表原本因此在 320px 溢出 37px。窄螢幕要解除。
- **flex 子元素要 `flex:none` 才不會被壓扁。** 固定尺寸的方形徽章少了它，會被擠成長條。
- **`<span>` 加 `margin-top` 無效。** 要堆疊成多行就得 `display:block`。

### 怎麼驗

```js
({w:innerWidth,
  overflow:document.documentElement.scrollWidth-innerWidth,
  bad:[...document.querySelectorAll('*')]
        .filter(e=>e.getBoundingClientRect().right>innerWidth+1)
        .map(e=>e.tagName+'.'+(e.className||''))})
```

`overflow` 要是 0，`bad` 要是空陣列。四個寬度都跑。

---

## 四、Token 紀律

`:root` 與 `@media print` 之外**不得出現任何字面色值或字體堆疊**。目前殘留 0。

```bash
# 檢查全部七個檔
python - <<'PY'
import glob,re
for f in sorted(glob.glob('*.html')):
    s=open(f,encoding='utf-8').read()
    head=s[:s.index('</style>')]
    rest='\n'.join(l for l in head.split('\n') if not re.match(r'\s*--[a-z-]+:',l))+s[s.index('</style>'):]
    rest=re.sub(r'@media print\{(?:[^{}]|\{[^{}]*\})*\}','',rest)
    lit=len(re.findall(r'#[0-9A-Fa-f]{3,8}\b',rest))+len(re.findall(r'rgba?\([^)]*\)',rest))
    d=set(re.findall(r'(--[A-Za-z][\w-]*)\s*:',s)); u=set(re.findall(r'var\((--[A-Za-z][\w-]*)\)',s))
    print(f'{"OK" if lit==0 and not u-d else "FAIL"}  {f}: 字面色值 {lit} · 未定義 token {sorted(u-d)}')
PY
```

> **反向檢查很重要**（`u-d`：被 `var()` 引用但沒定義）。只查「定義了沒用到」會漏掉真正的 bug——曾經因此漏掉 `--sidebar-w` 和 `--fig`／`--tespn` 被誤刪，側欄與分組識別色直接失效。

---

## 本機驗證環境

`file://` 開啟時瀏覽器可能給快取的舊版本，且部分 API 受限。起一個本機 server：

```bash
python -m http.server 8731 --bind 127.0.0.1
```

然後開 `http://127.0.0.1:8731/`。驗完記得關掉。
