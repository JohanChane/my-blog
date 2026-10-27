# Johan Chane's Blog

[博客](https://johanchane.github.io/my-blog/) · [English](README.en.md)

中文技术博客，基于 [Hugo FixIt starter1](https://github.com/hugo-fixit/hugo-fixit-starter1)，通过 Git 子模块管理主题。

## 本次迁移基线

2026-10-28 对齐上游模板和主题。FixIt 当前采用 v1 预发布版本；子模块固定到下表中的提交。

| Component | Pinned revision |
| --- | --- |
| starter1 | `0d602a8b75e46f74eca895e352c5e30eaa1b5c60` |
| FixIt | `v1.0.0-beta.5` / `3f506619006d740b86e3e699012329bd6303ed27` |
| shortcode-docs-bookmark | `76ed8abf64b706f6d195ebcf7bd744653a983248` |
| Hugo Extended | `0.168.0` |
| Dart Sass | `1.106.1` |

## 本地开发

安装 Git、Hugo Extended 和原生 Dart Sass，并将 `hugo`、`sass` 加入 PATH。主题最低要求 Hugo 0.166.0，建议使用与 CI 相同的上述版本。

已有仓库首次运行时，初始化子模块：

```bash
git submodule update --init --recursive
hugo server -D --disableFastRender
```

访问 `http://localhost:1313/my-blog/`。`-D` 会显示草稿；正式构建不发布草稿。

```bash
hugo new content posts/my-post.md
hugo --gc --minify --logLevel info
```

## 项目配置

- `content/`：文章和 Projects 页面。
- `config/_default/`：新版主题默认配置和站点共用设置。
- `config/production/`：正式网址、标题、作者、Giscus 和菜单。
- `config/development/`：本地开发标题、中文和图标设置。
- `assets/scss/custom.scss`：自定义样式；主题变量通过 `params.appearance` 配置。
- `static/`：图片等静态资源。

文章路径保持 `posts/:slugorcontentbasename`。Giscus 使用 `pathname` 匹配讨论，修改文章路径前要考虑已有评论。`social.RSS = false` 仅隐藏社交栏中的 RSS 图标，RSS 输出仍保留。

FixIt v1 将主题参数改为 snake_case，并调整评论、代码块等配置结构。升级时应对照 [官方迁移指南](https://fixit.lruihao.cn/zh-cn/guides/upgrade-to-v1/)，保留本站生产配置，不要直接覆盖为模板作者的配置。

## 部署

GitHub Pages 的 Source 选择 **GitHub Actions**。推送到 `main` 的相关站点变更会触发部署，也可以手动运行 **Hugo build and deploy**。

正式地址在 `config/production/hugo.toml` 中配置。工作流已经声明 `contents: read`、`pages: write` 和 `id-token: write` 权限，无需为自动提交配置仓库写权限。

## 手动升级

`Update theme` 工作流已移除；主题和配置不会再由定时任务直接更新。Dependabot 仍可提出依赖升级 PR，合并由维护者决定。

以后升级时：

1. 检查工作区和子模块内是否存在未提交修改。
2. 获取上游更新，选择明确的主题标签或提交。
3. 对比 starter1 和主题配置变更，迁移本站覆盖值。
4. 验证生产构建、文章 URL、中文、搜索和 Giscus。
5. 审查差异后，提交配置及子模块指针。推送到 `main` 会发布。

```bash
git -C themes/FixIt fetch origin --tags
# 将 TAG_OR_SHA 替换为已审查的标签或提交。
git -C themes/FixIt switch --detach TAG_OR_SHA
hugo --gc --minify --logLevel info
git diff --submodule=log
```

模板和主题应作为同一次兼容性迁移处理，不要单独降级主题后继续使用另一版本的配置。
