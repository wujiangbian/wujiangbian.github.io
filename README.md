# 博客源码（Hugo + PaperMod）

这是 wujiangbian.github.io 的 Hugo 源码分支。线上站点（main 分支）放的是构建成品，由 GitHub Pages 直接发布。

## 本地重建并更新线上

1. 安装 Hugo（v0.170.1 或更新的普通版即可，不需要 extended）
2. 在本目录运行 `hugo --minify`，成品生成到 `public/`
3. 把 `public/` 里的全部文件推到仓库的 `main` 分支即可上线

主题 PaperMod 已内嵌在 `themes/PaperMod/`，文章放在 `content/posts/`（目前为空，尚未迁入文章）。
