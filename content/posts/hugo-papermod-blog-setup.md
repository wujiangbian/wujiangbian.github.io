---
title: "Hugo + PaperMod 博客UI定制记录"
date: 2026-11-22T05:28:00+08:00
draft: false
---

本文记录博客（Hugo + PaperMod，https://wujiangbian.github.io ）的全部 UI 定制：改了哪些效果、每项写在哪个文件、关键代码是什么，照着可以复刻。全站以毛玻璃为统一语言：文章卡片、顶栏、评论区、浮动键共用同一套参数（背景色 60% 不透明度 + blur 10px），配低透明度全站背景图（浅色 0.22、深色 0.10）；主页用个人卡片模式并补回最新文章列表与浮现动画；顶栏改成悬浮胶囊，手机端收成一行；文章页做了图片限高与点击灯箱、日期中文化、中文阅读时间、giscus 评论透明主题适配。目录、文章封面、顶栏搜索框、标签菜单四项试过后弃用，记在文末。搭建、迁移、发布流程不在本文范围。

## 一、定制入口

全部定制只动三个地方，不改主题源码，主题升级不受影响：

- `assets/css/extended/custom.css`：全部样式集中在这一个文件（下文简称 custom.css）
- `layouts/` 覆盖模板：`_partials/header.html`（顶栏）、`_partials/footer.html`（页脚）、`_partials/comments.html`（评论）、`_partials/extend_footer.html`（页尾注入的 HTML 和脚本，下文简称 extend_footer）、`list.html`（首页）、`archives.html`（归档页）、`_default/about.html`（关于页）
- `hugo.toml`：少量开关型配置，随项说明

毛玻璃的共用写法（custom.css，后文多处复用这组值）：

```css
background: color-mix(in srgb, var(--entry) 60%, transparent);
backdrop-filter: blur(10px) saturate(1.25);
-webkit-backdrop-filter: blur(10px) saturate(1.25);
```

## 二、整体外观

### 2.1 全站背景图

绿字图放 `static/bg.jpg`，用 `body::before` 固定垫底（custom.css）：

```css
body::before {
  content: "";
  position: fixed;
  inset: 0;
  background: url("/bg.jpg") center / cover no-repeat;
  opacity: 0.22;
  z-index: -1;
  pointer-events: none;
}
html[data-theme="dark"] body::before { opacity: 0.10; }
```

要点：深色模式靠 `html` 的 `data-theme` 属性判断，不是 class。背景图从 323KB 压到 158KB（宽 900、质量 62），低透明度垫底看不出画质损失。

### 2.2 毛玻璃卡片

```css
.post-single, .post-entry {
  background: color-mix(in srgb, var(--entry) 60%, transparent);
  backdrop-filter: blur(10px) saturate(1.25);
  -webkit-backdrop-filter: blur(10px) saturate(1.25);
}
.post-single { border-radius: 12px; padding: 24px; }
```

浓度从 76% 试到 60% 定版。

### 2.3 排版

主题里博文标题带斜体，去掉只留粗体，并定字号（custom.css）：

```css
.post-content h1, .post-content h2, .post-content h3,
.post-content h4, .post-content h5, .post-content h6 { font-style: normal; }
.post-content h3 { font-size: 28px; }
.post-content h4 { font-size: 24px; }
.post-content h5 { font-size: 20px; }
```

正文字号 18px 是主题默认值，未改动。

## 三、主页

### 3.1 个人卡片

开启主题自带的 profileMode（hugo.toml）：

```toml
[params.profileMode]
  enabled = true
  title = "wujiangbian"
  subtitle = "猫老吴在中国工作的天"
  imageUrl = "/avatar.jpg"
  imageWidth = 120
  imageHeight = 120
  [[params.profileMode.buttons]]
    name = "归档"
    url = "/archives/"
  [[params.profileMode.buttons]]
    name = "搜索"
    url = "/search/"
```

社交图标走主题的 `params.socialIcons` 配置（本文只放了 GitHub）。

### 3.2 最新文章列表

卡片模式下首页默认不列文章。覆盖 `layouts/list.html`，在卡片下方加一段：遍历 posts 分区、按主题的条目卡片结构渲染（`layouts/list.html`）：

```html
<div class="home-posts">
  <h2 class="home-posts-title reveal">最新文章</h2>
  {{- $pages := where site.RegularPages "Section" "eq" "posts" }}
  {{- range $page := $pages }}
  <article class="post-entry reveal">
    <header class="entry-header">
      <h2 class="entry-hint-parent">{{ .Title }}</h2>
    </header>
    <div class="entry-content">
      <p>{{ .Summary | plainify | htmlUnescape }}{{ if .Truncated }}...{{ end }}</p>
    </div>
    <footer class="entry-footer">
      {{- partial "post_meta.html" . -}}
    </footer>
    <a class="entry-link" aria-label="post link to {{ .Title | plainify }}" href="{{ .Permalink }}"></a>
  </article>
  {{- end }}
</div>
```

条目上的 `reveal` 类配合 3.3 的浮现动画。

### 3.3 动画

CSS 在 custom.css：页面淡入、头像弹入、条目待浮现的初始状态：

```css
html { scroll-behavior: smooth; }
.main { animation: fadeUp 0.6s ease both; }
.profile_inner img { animation: pop 0.7s ease both; }
@keyframes fadeUp { from { opacity: 0; transform: translateY(14px); } to { opacity: 1; transform: none; } }
@keyframes pop { from { opacity: 0; transform: scale(0.85); } to { opacity: 1; transform: scale(1); } }
.reveal { opacity: 0; transform: translateY(26px); transition: opacity 0.7s ease, transform 0.7s ease; }
.reveal.visible { opacity: 1; transform: none; }
.post-entry { transition: transform 0.25s ease, box-shadow 0.25s ease; }
.post-entry:hover { transform: translateY(-3px); }
@media (prefers-reduced-motion: reduce) {
  .main, .profile_inner img { animation: none; }
  .reveal { opacity: 1; transform: none; transition: none; }
}
```

JS 在 extend_footer：IntersectionObserver 监听 `.reveal`，进入可视区加 `.visible`：

```html
<script>
(function () {
  var els = document.querySelectorAll(".reveal");
  if (!("IntersectionObserver" in window)) {
    els.forEach(function (e) { e.classList.add("visible"); });
    return;
  }
  var io = new IntersectionObserver(function (entries) {
    entries.forEach(function (en) {
      if (en.isIntersecting) { en.target.classList.add("visible"); io.unobserve(en.target); }
    });
  }, { threshold: 0.1 });
  els.forEach(function (e) { io.observe(e); });
})();
</script>
```

## 四、顶栏

覆盖 `layouts/_partials/header.html` 重排结构：左侧 logo（头像+标题），右侧一个 `.header-right` 容器依次放菜单 `ul.menu`、社交图标 `.social-icons`、主题切换按钮 `#theme-toggle`。样式分四步（custom.css）：

**第一步，胶囊本体** — sticky 悬浮、限宽居中、圆角边框阴影、毛玻璃底：

```css
.header {
  position: sticky;
  top: 12px;
  z-index: 100;
  width: min(1180px, calc(100% - 24px));
  margin: 14px auto 0;
  border: 1px solid color-mix(in srgb, var(--border) 65%, transparent);
  border-radius: 18px;
  box-shadow: 0 10px 34px rgba(0, 0, 0, 0.10);
  background: color-mix(in srgb, var(--theme) 62%, transparent);
  backdrop-filter: blur(10px) saturate(1.25);
  -webkit-backdrop-filter: blur(10px) saturate(1.25);
}
.header-nav { max-width: none; padding: 8px 12px 8px 18px; gap: 20px; }
.header-right { display: flex; align-items: center; gap: 22px; margin-left: auto; }
```

**第二步，logo 与图标**：

```css
.logo a { font-weight: 700; font-size: 19px; }
.logo img { border-radius: 50%; width: 30px; height: 30px; object-fit: cover;
  border: 2px solid color-mix(in srgb, var(--border) 75%, transparent); }
.header .social-icons { display: flex; align-items: center; gap: 14px; padding: 0; }
.header .social-icons a svg { width: 22px; height: 22px; transition: transform 0.2s ease; }
.header .social-icons a:hover svg { transform: translateY(-2px); }
```

**第三步，菜单下划线动画** — 给菜单文字的 span 加伪元素，悬停时宽度从 0 展开：

```css
.menu { gap: 20px; }
.menu li a span { position: relative; }
.menu li a span::after {
  content: "";
  position: absolute; left: 0; bottom: -5px;
  height: 2px; width: 0; border-radius: 2px;
  background: var(--primary);
  transition: width 0.25s ease;
}
.menu li a:hover span::after { width: 100%; }
```

**第四步，滚动阴影** — extend_footer 里监听滚动，超过 8px 给顶栏加 `.scrolled`：

```html
<script>
window.addEventListener("scroll", function () {
  var h = document.querySelector(".header");
  if (h) { h.classList.toggle("scrolled", window.scrollY > 8); }
}, { passive: true });
</script>
```

```css
.header { transition: box-shadow 0.3s ease; }
.header.scrolled {
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.10);
  border-bottom: 1px solid color-mix(in srgb, var(--border) 60%, transparent);
}
```

**手机端**（`@media (max-width: 640px)`）：标题文字用 `font-size: 0` 隐藏、只留头像（logo 的链接本来就包着头像，点头像即回首页），其余元素收紧间距挤进一行：

```css
@media (max-width: 640px) {
  .header { width: calc(100% - 16px); margin-top: 8px; top: 8px; border-radius: 14px; }
  .header-nav { flex-wrap: nowrap; padding: 8px 10px; }
  .logo a { font-size: 0; white-space: nowrap; }
  .logo img { width: 26px; height: 26px; }
  .header-right { gap: 10px; flex-wrap: nowrap; }
  .menu { gap: 13px; flex-wrap: nowrap; padding: 0; }
  .menu li a { font-size: 14px; white-space: nowrap; }
  .header .social-icons { gap: 8px; }
  .header .social-icons a svg { width: 18px; height: 18px; }
}
```

初版曾让菜单独占第二行居中，太占高度，弃用。

## 五、文章页

### 5.1 正文图片限高 + 点击灯箱

限高（custom.css）：

```css
.post-content img {
  max-height: 330px;
  width: auto;
  max-width: 100%;
  height: auto;
  display: block;
  margin: 1em auto;
  cursor: zoom-in;
}
```

灯箱层写在 extend_footer（HTML + CSS + JS 三件套）。HTML 是一个全屏遮罩，里面放图片、说明文字和关闭键：

```html
<div class="lightbox" id="lightbox" role="dialog" aria-label="图片预览">
  <button class="lb-close" id="lb-close" aria-label="关闭">✕</button>
  <img id="lb-img" alt="">
  <div class="lb-caption" id="lb-caption"></div>
</div>
```

```css
.lightbox {
  position: fixed; inset: 0; z-index: 200;
  display: none; align-items: center; justify-content: center; flex-direction: column; gap: 12px;
  background: rgba(10, 10, 12, 0.88);
  backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px);
}
.lightbox.open { display: flex; }
.lightbox img { max-width: 92vw; max-height: 86vh; border-radius: 10px; cursor: zoom-out; }
.lightbox .lb-caption { color: #e8e8e8; font-size: 14px; max-width: 88vw; text-align: center; }
```

JS 给所有正文图片绑点击：打开时把图片地址和 alt（配图说明）填进灯箱并锁定页面滚动；点遮罩、点 ✕、按 Esc 都能关：

```html
<script>
(function () {
  var lb = document.getElementById("lightbox");
  var img = document.getElementById("lb-img");
  var cap = document.getElementById("lb-caption");
  function open(src, alt) {
    img.src = src; img.alt = alt || "";
    cap.textContent = alt || "";
    cap.style.display = alt ? "block" : "none";
    lb.classList.add("open");
    document.body.style.overflow = "hidden";
  }
  function close() {
    lb.classList.remove("open");
    img.src = "";
    document.body.style.overflow = "";
  }
  document.querySelectorAll(".post-content img").forEach(function (el) {
    el.addEventListener("click", function () { open(el.currentSrc || el.src, el.alt); });
  });
  lb.addEventListener("click", close);
  document.getElementById("lb-close").addEventListener("click", close);
  document.addEventListener("keydown", function (e) { if (e.key === "Escape") { close(); } });
})();
</script>
```

### 5.2 日期与月份中文化

文章日期（hugo.toml）：

```toml
[params]
  dateFormat = "2006年1月2日"
```

归档页月份改配置无效：主题模板用 Go 格式分组、月份名恒为英文。覆盖 `layouts/archives.html`，把分组格式从 `"January"` 改成 `"01"`，标题输出改成数字加"月"：

```html
{{- range .Pages.GroupByDate "01" }}
  <h2 class="archive-year-header">
    {{ int .Key }}月
  </h2>
{{- end }}
```

（实际模板里年份、月份是两层分组，这里只示意改动点。）

### 5.3 中文阅读时间

hugo.toml 的语言配置里开 CJK 统计：

```toml
[languages."zh-cn"]
  hasCJKLanguage = true
```

不开时按单词数算，中文文章阅读时间明显偏短；开启后年度总结显示 12 分钟，符合实际字数。

### 5.4 评论（giscus）

前置：仓库 Settings 里开启 Discussions，安装 giscus 应用（只授权本仓库）。然后覆盖 `layouts/_partials/comments.html`：外层包毛玻璃卡片，脚本**动态创建**（不用 giscus 官网给的静态写法），为了能按当前主题选主题文件、并监听切换：

```html
<div class="comments-card">
<script>
(function () {
  var darkTheme = "https://wujiangbian.github.io/giscus-dark.css";
  var lightTheme = "https://wujiangbian.github.io/giscus-light.css";
  function currentTheme() {
    return document.documentElement.dataset.theme === "dark" ? darkTheme : lightTheme;
  }
  var s = document.createElement("script");
  s.src = "https://giscus.app/client.js";
  s.setAttribute("data-repo", "wujiangbian/wujiangbian.github.io");
  s.setAttribute("data-repo-id", "R_kgDOV2zOGg");
  s.setAttribute("data-category", "Announcements");
  s.setAttribute("data-category-id", "DIC_kwDOV2zOGs4DLrJK");
  s.setAttribute("data-mapping", "pathname");
  s.setAttribute("data-theme", currentTheme());
  s.setAttribute("data-lang", "zh-CN");
  s.setAttribute("crossorigin", "anonymous");
  s.async = true;
  document.currentScript.parentNode.appendChild(s);
  new MutationObserver(function () {
    var iframe = document.querySelector("iframe.giscus-frame");
    if (iframe) {
      iframe.contentWindow.postMessage(
        { giscus: { setConfig: { theme: currentTheme() } } },
        "https://giscus.app"
      );
    }
  }).observe(document.documentElement, { attributes: true, attributeFilter: ["data-theme"] });
})();
</script>
</div>
```

其中 repo-id 和 category-id 在 giscus.app 配置页生成配置时获得，换仓库要重新取。两个主题文件放 `static/` 自托管：在 giscus 官方 light / transparent_dark 主题基础上把背景改透明，这样 iframe 透出外层卡片的毛玻璃：

```css
.comments-card {
  margin-top: 36px;
  padding: 20px 22px;
  border-radius: 12px;
  border: 1px solid color-mix(in srgb, var(--border) 55%, transparent);
  background: color-mix(in srgb, var(--entry) 60%, transparent);
  backdrop-filter: blur(10px) saturate(1.25);
  -webkit-backdrop-filter: blur(10px) saturate(1.25);
}
```

## 六、零散部件

### 6.1 浮动导航键

extend_footer 注入一对按钮 + 点击脚本（平滑滚动靠 `scroll-behavior: smooth` 或参数里的 behavior）：

```html
<div class="float-nav">
  <button type="button" id="to-top" aria-label="回到顶部" title="回到顶部">↑</button>
  <button type="button" id="to-bottom" aria-label="回到底部" title="回到底部">↓</button>
</div>
<script>
document.getElementById("to-top").addEventListener("click", function () {
  window.scrollTo({ top: 0, behavior: "smooth" });
});
document.getElementById("to-bottom").addEventListener("click", function () {
  window.scrollTo({ top: document.documentElement.scrollHeight, behavior: "smooth" });
});
</script>
```

```css
.float-nav { position: fixed; right: 30px; bottom: 30px; z-index: 99;
  display: flex; flex-direction: column; gap: 16px; }
.float-nav button {
  width: 54px; height: 54px; border-radius: 50%;
  border: 1px solid var(--border);
  background: color-mix(in srgb, var(--entry) 60%, transparent);
  backdrop-filter: blur(10px) saturate(1.25);
  -webkit-backdrop-filter: blur(10px) saturate(1.25);
  color: var(--primary); font-size: 23px; line-height: 1; cursor: pointer;
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.12);
}
.float-nav button:hover { transform: translateY(-2px); }
```

两个配套点：主题自带一个回到顶部键、会与这对键叠成三个圆，在 hugo.toml 里关掉：`disableScrollToTop = true`。手机端浮动键离边收至 16px（写在 640px 媒体查询里）。

### 6.2 关于页

`content/about.md`，菜单第一位。新建 `layouts/_default/about.html`，只渲染标题和正文 — 普通文章模板带日期、阅读时间、评论区，关于页都不要：

```html
{{- define "main" }}
<article class="post-single">
  <header class="post-header">
    <h1 class="post-title">{{ .Title }}</h1>
  </header>
  <div class="post-content md-content">
    {{ .Content }}
  </div>
</article>
{{- end }}
```

front matter 要写日期，否则搜索引擎的结构化数据里会出现 0001 年假日期（页面上看不见）。文案："wujiangbian，梦想是成为快乐火星猫，在地球艰难生存中。"

### 6.3 页脚

覆盖 `layouts/_partials/footer.html`，删掉 "Powered by Hugo & PaperMod" 那段，只留版权行；文案在 hugo.toml 顶层：`copyright = "© 2026 wujiangbian"`。

### 6.4 网站图标（favicon）

hugo.toml：

```toml
[params.assets]
  favicon = "/avatar.jpg"
  favicon16x16 = "/favicon-16x16.png"
  favicon32x32 = "/favicon-32x32.png"
  apple_touch_icon = "/apple-touch-icon.png"
```

主 favicon 直接指向头像文件，换头像自动同步；其余三个用头像生成、放根目录。此前标签页无图标，是因为主题引用了不存在的 16/32 png、浏览器优先用它们，补齐文件即恢复；换图标后本机不显示多为缓存，强刷即可。另外 `params.images = ["/avatar.jpg"]` 让分享预览图也用头像。

## 七、试过又弃用的

- **文章目录（TOC）**：加过左侧目录，效果不好，完整回滚
- **文章封面**：给年度总结加过封面，改了限高、只在首页显示、贴卡片左侧绝对定位多版，结论是累赘，已彻底删除（含仓库里的图文件）
- **顶栏搜索框**：主题自带的已从 header.html 删除（连同 Ctrl+K 快捷键脚本），搜索入口只保留主页卡片上的"搜索"按钮
- **"标签"菜单**：文章都没打标签、标签页为空，已从菜单撤掉

## 八、关键数值速查

背景不透明度 0.22 / 深色 0.10 ｜ 毛玻璃 60% + blur 10px saturate 1.25 ｜ 标题 28/24/20px ｜ 正文图限高 330px ｜ 顶栏距顶 12px、圆角 18px、宽上限 1180px ｜ 浮动键 54px、间距 16px、离边 30px ｜ 手机端断点 640px
