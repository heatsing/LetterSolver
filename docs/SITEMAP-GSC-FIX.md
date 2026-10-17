# Sitemap 与 Google Search Console 检查

主站从仓库根目录构建并部署到 Netlify。当前 `robots.txt` 声明 `https://lettersolver.net/sitemap.xml`，该地址直接提供 XML；`sitemap-index.xml` 仍保留。

## 构建产物

根目录 `package.json` 使用：

```json
"build": "npm run build:dict && astro build && node scripts/ensure-sitemap.mjs",
"validate:sitemap": "node scripts/validate-sitemap.mjs"
```

`@astrojs/sitemap` 生成 `dist/sitemap-index.xml` 与分片 `dist/sitemap-0.xml` 等。随后 `scripts/ensure-sitemap.mjs`：

1. 若首分片存在且 `sitemap.xml` 不存在，复制首分片为 `sitemap.xml`。
2. 若 index 已存在，保留它并结束。
3. 若 index 缺失，扫描 `dist` 中的页面生成分片和 index，并在需要时创建首分片副本 `sitemap.xml`。

`sitemap.xml` 是首分片的副本，不是整个 index 的副本。若以后产生多个分片，应向 GSC 提交 `sitemap-index.xml` 以覆盖全部分片。

## Netlify 路径与响应头

根目录 `netlify.toml` 定义：

| 路径 | 预期行为 |
| --- | --- |
| `/sitemap.xml` | 200，`application/xml; charset=utf-8` |
| `/sitemap-index.xml` | 200，`application/xml; charset=utf-8` |
| `/sitemap-0.xml` | 301 到同站点 `/sitemap.xml` |
| `/sitemap-1.xml`、`/sitemap-2.xml` | 若构建生成这些文件，使用配置的 XML 响应头 |
| `/robots.txt` | 200，`text/plain; charset=utf-8` |

首分片重定向使用 `force = true`，因此即使 `dist/sitemap-0.xml` 存在也会重定向。index 中指向首分片的链接会经过该重定向读取相同内容。

当前 `public/robots.txt` 内容为：

```text
User-agent: *
Allow: /
Disallow: /private/

Sitemap: https://lettersolver.net/sitemap.xml
```

## 验证步骤

在仓库根目录运行 `npm ci`、`npm run build`，确认 `dist` 内存在 sitemap 文件。运行 `npm run preview` 可查看本地静态产物，但 Netlify 的重定向和响应头需在部署后检查。

```bash
# 正式部署后检查；PowerShell 可将 curl 改为 curl.exe
curl -I https://lettersolver.net/sitemap.xml
curl -I https://lettersolver.net/sitemap-index.xml
curl -I https://lettersolver.net/sitemap-0.xml
curl -I https://lettersolver.net/robots.txt
npm run validate:sitemap -- https://lettersolver.net

# 本地预览地址以终端输出为准
npm run validate:sitemap -- http://localhost:4321
```

校验脚本默认地址是 `http://localhost:4321`，会请求该地址下的 `/sitemap-index.xml`，并跟随 index 中的分片 URL。它检查状态、index 的 XML Content-Type、XML 声明、URL 数量和 HTTPS URL 等基本条件。

注意：当前 index 中分片使用 `https://lettersolver.net/...` 绝对地址。因此对本地或 Netlify 预览地址运行脚本时，分片仍可能从正式域名读取；这不能作为预览部署全部 sitemap 产物都通过验证的证据。迁移时还应直接访问预览站点自己的 `/sitemap.xml` 并检查内容。

确认正式域名已经切换到 Netlify 后，在 [Google Search Console](https://search.google.com/search-console) 的「站点地图」中提交 `https://lettersolver.net/sitemap.xml` 或保留的 index，随后查看抓取状态。

## 排查无法抓取

- **404**：确认 Netlify 从仓库根目录构建、发布目录为 `dist`，且日志执行了 `ensure-sitemap.mjs`。
- **返回 HTML**：检查该地址是否错误落到首页、404 页面或旧部署；sitemap 应返回 XML 内容。
- **仍访问旧站**：确认域名绑定、DNS 与 HTTPS 状态，使用 Netlify 项目预览地址对照检查。
- **提交后尚未更新**：先通过正式 URL 检查实际响应，再查看 GSC 的抓取时间与错误信息。

静态站由 `src/pages/404.astro` 生成 `404.html`；部署后同时确认未知路径返回 HTTP 404。canonical 继续使用正式域名 `https://lettersolver.net`。
