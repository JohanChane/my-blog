# Johan Chane's Blog

[Website](https://johanchane.github.io/my-blog/) · [简体中文](README.md)

A Chinese technical blog based on [Hugo FixIt starter1](https://github.com/hugo-fixit/hugo-fixit-starter1), with themes managed as Git submodules.

## Migration baseline

Updated against the upstream template and theme on 2026-10-28. FixIt is currently a v1 prerelease. Submodules are pinned to these revisions:

| Component | Pinned revision |
| --- | --- |
| starter1 | `0d602a8b75e46f74eca895e352c5e30eaa1b5c60` |
| FixIt | `v1.0.0-beta.5` / `3f506619006d740b86e3e699012329bd6303ed27` |
| shortcode-docs-bookmark | `76ed8abf64b706f6d195ebcf7bd744653a983248` |
| Hugo Extended | `0.168.0` |
| Dart Sass | `1.106.1` |

## Local development

Install Git, Hugo Extended, and native Dart Sass. Make `hugo` and `sass` available on PATH. The theme requires Hugo 0.166.0 or newer; use the versions above to match CI.

```bash
git submodule update --init --recursive
hugo server -D --disableFastRender
```

Open `http://localhost:1313/my-blog/`. The `-D` flag includes drafts locally; production builds exclude them.

```bash
hugo new content posts/my-post.md
hugo --gc --minify --logLevel info
```

## Configuration

- `content/`: posts and the Projects page.
- `config/_default/`: current theme defaults and shared site settings.
- `config/production/`: public URL, title, author, Giscus, and navigation.
- `config/development/`: local title, Chinese locale, and icons.
- `assets/scss/custom.scss`: custom styles. Configure theme variables in `params.appearance`.
- `static/`: images and other static resources.

Keep the `posts/:slugorcontentbasename` permalink pattern to preserve article URLs and Giscus discussions, which use `pathname` mapping. `social.RSS = false` hides only the social icon; RSS feeds remain enabled.

FixIt v1 uses snake_case theme parameters and changes the comment and code-block configuration structure. Follow the [migration guide](https://fixit.lruihao.cn/guides/upgrade-to-v1/) and retain this site's overrides instead of copying the template author's settings.

## Deployment

Set GitHub Pages Source to **GitHub Actions**. Relevant changes pushed to `main` trigger deployment. **Hugo build and deploy** can also be run manually.

Set the public URL in `config/production/hugo.toml`. The workflow declares `contents: read`, `pages: write`, and `id-token: write`; repository write permission for automatic commits is unnecessary.

## Manual upgrades

The **Update theme** workflow has been removed. Scheduled tasks no longer update the theme or regenerate configuration. Dependabot can still propose dependency PRs for review.

1. Check for uncommitted changes in the site and submodules.
2. Fetch upstream changes and select a specific tag or commit.
3. Compare starter1 and theme configuration changes; migrate site overrides.
4. Verify the production build, article URLs, Chinese text, search, and Giscus.
5. Review and commit the configuration and submodule pointers. Pushing to `main` publishes the site.

```bash
git -C themes/FixIt fetch origin --tags
# Replace TAG_OR_SHA with the reviewed tag or commit.
git -C themes/FixIt switch --detach TAG_OR_SHA
hugo --gc --minify --logLevel info
git diff --submodule=log
```

Treat the template and theme as one compatibility migration. Do not downgrade the theme alone while retaining configuration from another version.
