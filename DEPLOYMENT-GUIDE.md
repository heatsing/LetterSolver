# Netlify 部署指南

主站是仓库根目录的 Astro 静态项目，仓库为 [heatsing/LetterSolver](https://github.com/heatsing/LetterSolver)。Netlify 已关联此仓库；将迁移配置提交并推送到生产分支 `main` 后，会触发自动构建和部署。本指南描述部署步骤，不代表代码已经推送或域名已经切换。

## 构建配置

根目录 `netlify.toml` 是构建、重定向和响应头的配置来源。在 Netlify 项目设置中确认生产分支为 `main`，并清除旧的 `astro` 子目录设置：

| 设置 | 值 |
| --- | --- |
| Base directory | 仓库根目录，留空或 `.` |
| Package directory（若已设置） | 仓库根目录，留空或 `.` |
| Build command | `npm run build` |
| Publish directory | `dist` |
| Node.js | `24`，由 `NODE_VERSION` 固定 |
| npm 安装参数 | `--include=dev`，由 `NPM_FLAGS` 指定 |

依赖使用根目录 `package-lock.json`，构建工具包含在开发依赖中。项目保持 `output: 'static'`，无需添加 Netlify adapter。根目录的构建命令会生成词库、执行 Astro 构建，再运行 `scripts/ensure-sitemap.mjs`。

## 本地验证与自动部署

在仓库根目录使用 Node.js 24 执行：

```bash
npm ci
npm run build
npm run preview
```

在预览服务器显示的地址检查首页、工具页面和 `/sitemap.xml`。`npm run preview` 只预览静态文件；Netlify 的重定向和响应头需要在 Netlify 部署地址上确认。

将改动提交并推送到 `main` 后，在 [Netlify 控制台](https://app.netlify.com/) 打开该项目的部署记录，检查构建日志以及发布目录。先通过项目的 `*.netlify.app` 地址检查页面，再切换正式域名。

## 环境变量与词库

在 Netlify 项目的环境变量设置中添加实际需要的构建变量。当前广告组件使用以下可选变量：

- `PUBLIC_ADSENSE_CLIENT`：AdSense 发布商 ID。
- `PUBLIC_ADSENSE_SLOT`：广告位 ID。

`PUBLIC_*` 变量会用于客户端输出，不应放置密钥。无需配置旧文档中的 `NEXT_PUBLIC_*` 或 `SENTRY_DSN`。

正常生产构建保持 `SKIP_REMOTE_DICT` 未设置，以便脚本下载完整远程词表。`SKIP_REMOTE_DICT=1` 会跳过下载，使用本地 `words_alpha.txt` 或内置小词表，并合并 `words.txt`；缺少完整本地词表时会减少可查询单词。远程下载失败时也会回退，需在构建日志中确认词库来源。

## 域名迁移

1. 确认 Netlify 部署成功，预览地址上的工具可以使用。
2. 在 Netlify 域名管理中添加 `lettersolver.net`，如使用 `www`，同时添加 `www.lettersolver.net` 并设置主域名。
3. 在 DNS 服务商处按照该 Netlify 项目显示的记录更新域名解析，替换原来指向 Vercel 的对应记录。
4. 等待 Netlify 完成域名验证和 HTTPS 证书配置，再检查正式域名。
5. 确认流量已切换且功能正常后，可断开旧 Vercel 项目的 Git 自动部署，避免重复构建。

项目 canonical URL 和 sitemap URL 继续使用 `https://lettersolver.net`，因此无需更改网站域名配置。

## 部署后检查

以下为命令行示例；Windows PowerShell 可使用 `curl.exe` 代替 `curl`。

```bash
curl -I https://lettersolver.net/
curl -I https://lettersolver.net/sitemap.xml
curl -I https://lettersolver.net/sitemap-index.xml
curl -I https://lettersolver.net/sitemap-0.xml
curl -I https://lettersolver.net/robots.txt
curl -I https://lettersolver.net/5-letter-words
npm run validate:sitemap -- https://lettersolver.net
```

预期行为：

- 首页和主要工具页面正常打开；不存在的页面返回 404。
- `/sitemap.xml` 和 `/sitemap-index.xml` 返回 200 与 XML 类型；`/robots.txt` 返回文本并声明正式 sitemap 地址。
- `/sitemap-0.xml` 返回 301，跳转至同站点的 `/sitemap.xml`。
- `/2-letter-words` 至 `/10-letter-words` 返回 301，分别跳转至 `/two-letter-words` 至 `/ten-letter-words`。

重定向使用相对路径，便于在 Netlify 预览域名上测试。`force = true` 确保规则优先于同路径的静态文件。

`validate:sitemap` 默认检查 `http://localhost:4321`，生产检查应明确传入上述正式 URL。脚本会跟随 sitemap index 中的绝对分片地址；当前这些地址指向正式域名，因此传入本地或 Netlify 预览地址不能证明所有分片都来自该预览部署。

Google Search Console 可提交 `https://lettersolver.net/sitemap.xml`，也可提交保留的 `/sitemap-index.xml`。`sitemap.xml` 是首个分片的副本；若以后生成多个分片，应提交 index 以覆盖所有分片。详情见 [Sitemap 说明](docs/SITEMAP-GSC-FIX.md)。

## 常见问题

- **构建了旧站或找不到配置**：检查 Base directory、Package directory 是否误设为 `astro`，主站应从仓库根目录构建。
- **依赖安装失败**：检查日志是否使用 Node.js 24 和 npm，确认提交了 `package-lock.json`，并保留 `NPM_FLAGS=--include=dev`。
- **词库明显变小**：检查远程下载日志和 `SKIP_REMOTE_DICT`，必要时准备完整本地 `words_alpha.txt`。
- **sitemap 404**：确认构建命令包含 `node scripts/ensure-sitemap.mjs`，产物中存在 `dist/sitemap.xml`、`dist/sitemap-index.xml`，发布目录是 `dist`。
- **正式域名仍显示旧站**：检查 Netlify 域名验证、DNS 记录和 HTTPS 状态，再确认实际访问的部署版本。

参考：[Astro 部署到 Netlify](https://docs.astro.build/en/guides/deploy/netlify/)、[Netlify 构建依赖管理](https://docs.netlify.com/build/configure-builds/manage-dependencies/)。
