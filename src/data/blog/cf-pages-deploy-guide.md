---
description: 从 Astro 到 Cloudflare Pages 的踩坑实录。

pubDatetime: 2026-05-10
tags:
- Website
- Astro
- Cloudflare
- 笔记
title: 踩坑指南：如何把 Astro 博客部署到 Cloudflare Pages
---

## 前言

刚把博客从 Hugo 迁移到 Astro，准备顺手部署到 Cloudflare Pages。本以为是个 `git push` 就能解决的流程，结果硬生生折腾了半天。为了后人（主要是为了自己以后不再踩同样的坑），特此记录这篇《避坑指南》。

---

## 1. 包管理器冲突：pnpm vs npm

**现象**：构建日志中提示找不到依赖或命令失败，Cloudflare 环境默认使用 `npm`，但如果仓库中存在 `pnpm-lock.yaml`，构建系统会尝试使用 `pnpm` 进行安装，导致依赖版本错乱。

**解决**：删除 `pnpm-lock.yaml`，保持仓库中只有 `package-lock.json`，强制 Cloudflare 使用标准的 `npm` 流程。

```bash
rm pnpm-lock.yaml
```

---

## 2. OG 图片生成报错：Rollup 把 WASM 当 JS 解析

**现象**：构建过程中突然崩溃，报错信息非常诡异：
```
Unexpected character '\u{7f}'
at getRollupError (file:///opt/buildhome/repo/node_modules/rollup/dist/es/shared/parseAst.js)
```

**原因**：Astro 的 OG 图片生成依赖 `@resvg/resvg-js`，其中包含 `.wasm` 二进制文件。Vite 在 SSR 构建阶段试图把这些二进制文件当成 JS 模块打包，遇到文件头的魔数 `\x7F`（即 `{7f}`）直接懵了。

**解决**：在 `astro.config.ts` 中配置 `vite`，将该库标记为外部依赖：

```typescript
export default defineConfig({
  vite: {
    ssr: {
      external: ["@resvg/resvg-js"],
    },
  },
});
```

---

## 3. 部署命令之殇：Workers 还是 Pages？

**现象**：使用 `npx wrangler deploy` 部署时，报以下错误：
- `Missing entry-point to Worker script`
- `workers-site/index.js was not found`

**原因**：`wrangler deploy` 默认是部署 Cloudflare **Workers**（后端代码）。而 Astro 博客是纯静态资源（HTML/CSS/JS），不需要 Worker。如果 `wrangler.toml` 配置不对，它会去寻找入口文件而不是上传静态目录。

**解决**：我们需要告诉 Wrangler，这是一个静态资产部署。在 `wrangler.toml` 中明确指定 `[assets]`：

```toml
name = "ric-blog"
compatibility_date = "2025-01-01"

[assets]
directory = "./dist"
```

配合 Cloudflare Pages 控制台中的命令：
- **构建命令**：`npx astro build`
- **部署命令**：`npx wrangler deploy`

---

## 4. 权限错误：Authentication Error (code: 10000)

**现象**：部署脚本跑通了，但到了最后一步报错：
```
Authentication error [code: 10000]
Please ensure it has the correct permissions for this operation.
```

**原因**：在 Cloudflare 环境变量中配置的 `CLOUDFLARE_API_TOKEN` 权限不足。

**解决**：
1. 去 Cloudflare 后台重新生成一个 API Token。
2. 确保勾选 **Edit Cloudflare Pages** 权限。
3. 在项目 **Variables and secrets** 中更新该 Token。

---

## 5. 总结：最终可用的配置清单

如果你也想用 Astro + Cloudflare Pages，请抄这份作业：

### `wrangler.toml`
```toml
name = "ric-blog"
compatibility_date = "2025-01-01"

[assets]
directory = "./dist"
```

### Cloudflare Pages 控制台设置
| 配置项 | 值 |
|--------|-----|
| **构建命令** | `npx astro build` |
| **部署命令** | `npx wrangler deploy` |
| **版本命令** | `npx wrangler versions upload` |
| **根目录** | `/` |

### 环境变量
- `CLOUDFLARE_API_TOKEN`: 必须具有 **Pages: Edit** 权限。

---

希望这篇指南能帮你少掉几根头发。Happy Deploying! 🚀
