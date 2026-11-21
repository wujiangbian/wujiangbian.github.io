---
title: "Hugo + PaperMod 搭建博客记录"
date: 2026-11-22T05:28:00+08:00
draft: false
---

本文记录 2026 年 11 月将博客从 xLog 迁移到 Hugo + PaperMod + GitHub Pages 的完整过程：双分支仓库结构、从零搭建步骤、内容迁移方法、发布流程，以及个人卡片主页、全站背景、毛玻璃卡片、悬浮胶囊顶栏、动画、浮动导航键、giscus 评论、日期中文化、图片灯箱、手机端适配等全部 UI 定制的实现方式与关键代码。文末附每三个月一次的源码备份方案和踩坑清单。照本文可从零重建出一个一模一样的站。

## 一、总览

- 框架：Hugo v0.170.1（静态站点生成器，单个二进制文件，不需要 Node 环境）
- 主题：PaperMod，主题文件直接内嵌在仓库里（不走 git submodule，避免拉取时缺文件）
- 托管：GitHub Pages，仓库 `wujiangbian/wujiangbian.github.io`
- 站点地址：https://wujiangbian.github.io
- 评论：giscus（基于 GitHub Discussions）
- 备份：每三个月把源码打包存一份到 Google Drive，最多保留 4 份

仓库用两个分支：

- `main` 分支：放 Hugo 构建好的成品网页，GitHub Pages 直接发布这个分支
- `hugo-source` 分支：放 Hugo 源码（文章、配置、主题、图片源文件）

分工：Pages 发布成品，源码单独存一个分支。改站时改源码、构建、把成品推回 main，互不干扰。

## 二、从零搭建

### 1. 建仓库

在 GitHub 新建仓库，名字必须是 `你的用户名.github.io`（我的是 `wujiangbian.github.io`），这是 GitHub Pages 的固定规则。仓库设置里 Pages 的发布来源选 main 分支根目录。

### 2. 准备 Hugo 源码

本地建一个站点目录，结构如下（这就是 hugo-source 分支的内容）：

```
site/
├── hugo.toml            # 全站配置，最重要的一个文件
├── content/             # 文章（Markdown）
│   ├── posts/           # 博文
│   ├── archives.md      # 归档页
│   ├── search.md        # 搜索页
│   └── about.md         # 关于页
├── layouts/             # 对主题的覆盖文件（见第五节）
├── assets/css/extended/custom.css   # 全部自定义样式都写这里
├── static/              # 静态文件，原样发布到网站根目录
│   ├── avatar.jpg       # 头像
│   ├── bg.jpg           # 全站背景图
│   └── posts/           # 博文配图
└── themes/PaperMod/     # 主题本体，整份拷进来
```

构建命令：

```
hugo --minify --cleanDestinationDir
```

`--cleanDestinationDir` 必须加。不加的话，每次改样式生成的带指纹的 CSS 文件会一代代攒在输出目录里，曾经攒下 20 多个无用文件。

### 3. 基础配置（hugo.toml 要点）

```toml
baseURL = "https://wujiangbian.github.io/"
languageCode = "zh-cn"
title = "wujiangbian 的博客"
copyright = "© 2026 wujiangbian"
theme = "PaperMod"

[params]
  dateFormat = "2006年1月2日"     # 日期显示格式，见 5.9
  comments = true                  # 开启评论（giscus）
  disableScrollToTop = true        # 关掉主题自带的回到顶部键，见 5.7
  ShowReadingTime = true
  ShowShareButtons = false
  ShowPostNavLinks = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true

[languages."zh-cn"]
  hasCJKLanguage = true            # 中文按字数统计阅读时间，见 5.9
```

## 三、内容迁移（从 xLog 搬家）

老博客在 xLog 上，共 3 篇文章。官方导出工具当时不可用，最后直接读 xLog 的公开数据库导出，得到每篇的 Markdown 全文。图片存在 IPFS 上大多已失效：能找回原图的替换，找不回的删掉坏图。

迁移后的 3 篇：

- 《2023-2024 wujiangbian的电子产品年度总结》— 失效的 33 张图后来用留存的照片逐张对位替换
- 《如何使用git拉取项目并部署到自己的仓库（自存）》
- 《让程序以管理员权限启动且不弹出 UAC提示窗口》

## 四、发布流程

每次改完站都走一遍：

1. 改源码（文章、配置、样式都在源码目录改）
2. 本地构建：`hugo --minify --cleanDestinationDir`
3. 把构建成品推到 main 分支
4. 改过的源码文件推到 hugo-source 分支
5. 等 40~60 秒（Pages 部署有延迟），打开网站验证

注意点：

- 图片等二进制文件与文本文件的推送方式不同。图片要单独传到 main 的对应路径和 hugo-source 的 `static/` 对应路径，各一份，缺一份重建时就会丢图。
- 每次推送后要打开线上页面核对，不要只看推送成功。样式类改动还要核对线上 CSS 文件里确实有新规则。

## 五、UI 定制全记录

以下每一项记录效果与实现方法，文件路径均相对源码根目录。

### 5.1 主页：个人卡片 + 最新文章

PaperMod 自带 profileMode（个人卡片模式），在 hugo.toml 里开启：

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

卡片模式下首页默认不列文章。覆盖 `layouts/list.html`，在卡片下方加"最新文章"区块：遍历 posts 分区的文章，按条目卡片渲染（标题、摘要、日期）。条目带 `reveal` 类，配合 5.6 的动画下滑浮现。

### 5.2 全站背景图

背景是一张"感谢你的视奸"绿字图，放在 `static/bg.jpg`。给 body 加垫底伪元素，半透明显示（写在 custom.css）：

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
html[data-theme="dark"] body::before { opacity: 0.10; }  /* 深色模式单独调淡 */
```

要点：PaperMod 切换深色模式是改 `html` 上的 `data-theme` 属性，不是加 class。背景图原图 323KB，后压到 158KB（宽 900、质量 62）：它只是 22% 透明度垫底，画质损失不可见，首页加载更快。

### 5.3 毛玻璃卡片

文章页和文章条目统一毛玻璃，与顶栏、评论区、浮动键共用同一套参数：

```css
.post-single, .post-entry {
  background: color-mix(in srgb, var(--entry) 60%, transparent);
  backdrop-filter: blur(10px) saturate(1.25);
  -webkit-backdrop-filter: blur(10px) saturate(1.25);
}
.post-single { border-radius: 12px; padding: 28px 30px; }
```

浓度从 76% 试到 60%，定版 60%。

### 5.4 正文排版

- 博文标题去斜体、只留粗体
- 标题字号：一级 28px、二级 24px、三级 20px；正文 18px
- 正文图片限高 330px、等比例缩小、居中：

```css
.post-content img {
  max-height: 330px;
  width: auto;
  margin: 0 auto;
  cursor: zoom-in;   /* 配合 5.10 的灯箱 */
}
```

试过给文章加左侧目录（TOC），效果不好，已回滚，不再加。

### 5.5 顶栏：悬浮胶囊导航

覆盖 `layouts/_partials/header.html`，按 Blowfish 风格重排：

- 顶栏是悬浮的圆角胶囊：sticky 定位、距顶 12px、宽 `min(1180px, 100% - 24px)`、圆角 18px、毛玻璃底（`var(--theme)` 62% + blur 10px）、带边框和阴影
- 左边：圆形头像（30×30，2px 描边）+ 加粗小字标题（19px）
- 右边一组（间距 22px）：菜单、GitHub 图标、主题切换按钮
- 菜单悬停时有下划线展开动画
- 页面滚动超过 8px 后，JS 给顶栏加 `.scrolled` 类，阴影加深（JS 写在 extend_footer 里）
- 顶栏原有的搜索框已删除（连同 Ctrl+K 快捷键脚本），搜索入口只保留主页卡片上的"搜索"按钮
- 菜单内容：关于、归档、搜索。"标签"曾经有，因文章都没打标签、标签页为空，已撤掉

### 5.6 动画

写在 `layouts/_partials/extend_footer.html` 和 custom.css 里：

- 页面载入时整体淡入
- 头像弹入（带回弹的缩放）
- 卡片悬停时微微上浮
- 带 `.reveal` 类的元素用 IntersectionObserver 监听，进入可视区时加 `.visible` 类浮现（首页"最新文章"用此动画）

### 5.7 右下角浮动导航键

一对自制键（extend_footer 注入）：↑ 回到顶部、↓ 回到底部。54×54px 圆形、箭头 23px、两键间距 16px、离右和底各 30px，平滑滚动。毛玻璃参数与卡片统一，阴影用轻版（试过加深，已退回）。

曾出现三个圆叠在一起的 bug：PaperMod 自带一个"回到顶部"键（滚动后出现），与这对键挤在一起。解决办法是 hugo.toml 里 `disableScrollToTop = true` 关掉自带的。

### 5.8 评论区（giscus）

接入步骤：

1. 仓库设置里开启 Discussions
2. 安装 giscus 应用到这个仓库（只授权这一个仓库，权限：Discussions 读写）
3. 覆盖 `layouts/_partials/comments.html`，填入仓库的 repo-id 和分类 id（Announcements 分类），mapping 用 pathname，语言 zh-CN

giscus 是外嵌 iframe，不能直接改内部 CSS，样式适配用它支持的自定义主题：

- 网站根目录自托管两个主题文件 `giscus-light.css`、`giscus-dark.css`（在官方主题基础上把背景色改成透明）
- 评论区外包一层毛玻璃卡片（`.comments-card`，参数与博文卡片一致），iframe 透明底透出卡片的毛玻璃
- 评论脚本改为动态创建：按当前 `html[data-theme]` 选明/暗主题；用 MutationObserver 监听主题切换，通过 postMessage 通知 giscus 换主题，不刷新页面

### 5.9 日期和月份中文化

分两处：

1. 文章日期：hugo.toml 里 `dateFormat = "2006年1月2日"`，显示为"2025年1月8日"
2. 归档页月份：PaperMod 归档模板用 Go 时间格式分组，月份名恒为英文，改配置无效。覆盖 `layouts/archives.html`，分组格式从 `"January"` 改为 `"01"`，显示时输出 `{{ int .Key }}月`，变为"12月""1月"

中文阅读时间（年度总结显示 12 分钟）靠 hugo.toml 的 `hasCJKLanguage = true`。不开时中文按单词数统计，阅读时间会明显偏短。

### 5.10 图片灯箱

博文图片点击放大：extend_footer 注入全屏灯箱层 — 深色模糊背景、图片最大 92vw×86vh、显示图片的 alt 文字（配图说明）、点任意处或按 Esc 关闭、打开时锁定页面滚动。正文图片的 `cursor: zoom-in` 配合灯箱。

### 5.11 关于页

`content/about.md`，菜单第一位。使用自定义模板 `layouts/_default/about.html`，只渲染标题和正文：普通文章模板带日期、阅读时间和评论区，关于页不需要。front matter 里要写日期，否则搜索引擎的结构化数据里会出现 0001 年的假日期（页面不可见）。

关于我文案："wujiangbian，梦想是成为快乐火星猫，在地球艰难生存中。"

### 5.12 手机端适配

写在 custom.css 的 `@media (max-width: 640px)` 里：

- 顶栏胶囊变窄变矮（左右各留 8px、圆角 14px）
- 导航改两行：第一行头像标题 + 图标 + 主题键；菜单独占第二行、居中、字号 15px、项间距 26px
- 右下角浮动键往里收（离边 16px）
- 评论卡片内边距缩小

### 5.13 页脚

覆盖 `layouts/_partials/footer.html`，删掉 "Powered by Hugo & PaperMod"，只留版权行（文案在 hugo.toml 的 `copyright` 里）。

### 5.14 网站图标（favicon）

`params.assets.favicon` 直接指向头像文件 `/avatar.jpg`，换头像时标签页图标自动同步。另外用头像生成了 favicon.ico、favicon-16x16.png、favicon-32x32.png、apple-touch-icon.png 四个文件放在根目录。此前标签页无图标，原因是主题引用了两个不存在的 png 文件、浏览器优先用它们，补齐即恢复。换图标后本机仍不显示多为浏览器缓存，强刷（Ctrl+F5）即可。

### 5.15 封面图的试错（结论：不要封面）

给年度总结加过一张跳伞牛当封面，前后改了多版：封面图过大，加"最高 330px"限制（与正文图分开设置）；改为只在首页显示、文章内不显示；首页封面贴卡片左侧且不能撑大卡片（用绝对定位实现，卡片大小只由文字决定）。结论：封面是累赘，已彻底删除，图文件也从仓库清除。以后不再给文章加封面。

## 六、备份

- 每三个月备份一次：把 hugo-source 分支的全部源码打包成 zip
- 存放位置：Google Drive 的 `muse/blog/backup/` 下，按日期建文件夹（如 `2026-11-22/`）
- 最多保留 4 份，超出时从最早的一份开始删

有源码备份 + 本文即可随时重建：新建仓库 → 源码放 hugo-source 分支 → 按第二节构建 → 成品推 main 分支 → 二进制图片补传 → 按第五节逐项核对 UI。

## 七、踩坑清单

1. 构建不加 `--cleanDestinationDir`，旧样式文件会越攒越多
2. 图片等二进制文件要传两个位置（main + hugo-source 的 static），只传一处会在重建时丢图
3. 推送成功不等于上线，Pages 有 40~60 秒延迟，验证要等够时间再刷新
4. 主题自带的回到顶部键会与自制浮动键叠加，用 `disableScrollToTop` 关掉
5. 归档页月份中文化改配置无效，必须覆盖 archives 模板
6. giscus 不能直接改样式，要用自定义主题文件 + 透明底 + 外层卡片
7. 关于页这类页面记得手动给日期，避免结构化数据出现 0001 年
