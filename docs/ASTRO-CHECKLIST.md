# Astro 主站检查清单

## 已检查并修复

- **构建**：根目录 `package.json` 使用 Astro 脚本；在 Node.js 24 下运行 `npm ci`、`npm run build` 检查当前构建结果。
- **脚本**：`scripts/build-dict.ts` 生成 `public/data/words_*.json`，合并本地与远程词表；远程下载不可用时回退到本地或内置词表。
- **PostCSS**：根目录 `postcss.config.mjs` 已去掉 `@tailwindcss/postcss`，避免 Astro 构建报错。
- **动态路由**：`[...slug].astro` 的 `slug` 为字符串，符合 Astro 要求。
- **404**：`src/pages/404.astro` 已添加，带 `noindex`，避免 404 被收录。
- **SEO**：`BaseHead` 增加 favicon（`/icon.svg`）和 apple-touch-icon（`/apple-icon.png`）；`PageMeta` 支持 `noindex`，404 使用。
- **Sitemap**：`public/robots.txt` 指向 `sitemap.xml`；Astro 生成 sitemap index 和分片，`scripts/ensure-sitemap.mjs` 将首分片复制为 `dist/sitemap.xml`，并在 index 缺失时补生成 sitemap。
- **Netlify**：根目录 `netlify.toml` 设置 `npm run build` 和 `dist`，并配置旧路径的 301 重定向及 sitemap、robots 响应头。Base directory 和 Package directory 使用仓库根目录。
- **PWA**：`public/manifest.json` 图标改为现有资源（`/icon.svg`、`/apple-icon.png`），去掉不存在的 `icon-192.png`、`icon-512.png`、`screenshot-1.png`。
- **文档**：`README.md` 标明主站为 Astro，并说明本地运行与 `words.txt` 用法。
- **UI/UX**：已移除页面内重复的 `<InternalLinks />`（底部已有 SiteFooter 导航）；`words-ending-in` 由专用 hub 页 + `words-ending-in/[letter].astro` 动态页接管（26 字母页 + 1 hub）。

## 已迁移的动态路由

| 路由 | 说明 |
|------|------|
| `words-ending-in` | Hub 页：`src/pages/words-ending-in.astro` |
| `words-ending-in/[letter]` | 已实现：`src/pages/words-ending-in/[letter].astro`，`getStaticPaths` 生成 a–z 共 26 页；词库来自 `src/lib/words-data.ts`（构建时读取 `public/data/words_*.json`）。 |

## 已迁移的 Next 动态路由（全部在 Astro 中实现）

| 路由 | Astro 文件 | 说明 |
|------|------------|------|
| `[length]-letter-words-ending-with/[letter]` | `src/pages/[length]-letter-words-ending-with/[letter].astro` | 如 `/3-letter-words-ending-with/s` |
| `[length]-letter-words-starting-with/[letter]` | `src/pages/[length]-letter-words-starting-with/[letter].astro` | 如 `/3-letter-words-starting-with/b` |
| `{length}-letter-words-with-{letter}` | `src/pages/[slug].astro` | 单段 URL 如 `/10-letter-words-with-x`，仅生成 234 条路径 |

## 体验与质量

- **本地预览**：`npm run build` 后执行 `npm run preview`，在浏览器中逐页检查样式与链接。
- **Lighthouse**：部署后可在 Chrome DevTools → Lighthouse 对首页、/unscramble、/words-ending-in 等做性能与可访问性检查；或本地预览时运行 `npx lighthouse http://localhost:4321 --view`（需先 `npm run preview`）。
- **Netlify**：仓库已关联 Netlify；推送到生产分支 `main` 后自动构建，在控制台确认部署成功，并检查重定向、响应头和正式域名。

## 其他说明

- **词库**：根目录的 `words.txt`（每行一词）可补充词表。生产构建保持 `SKIP_REMOTE_DICT` 未设置；设为 `1` 时会跳过远程下载，缺少完整本地 `words_alpha.txt` 时会使用内置小词表。
- **部署说明**：参见根目录 [DEPLOYMENT-GUIDE.md](../DEPLOYMENT-GUIDE.md)。`astro/` 是历史子项目，主站从仓库根目录部署。
- **Next.js 已移除**：`app/` 目录已删除，项目仅使用 Astro 构建。根目录 `components/`、`lib/` 等为历史遗留，Astro 仅使用 `src/` 下的页面与组件。
