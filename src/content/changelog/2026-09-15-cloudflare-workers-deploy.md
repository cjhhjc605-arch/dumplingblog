---
version: "v1.37.0"
date: 2026-09-15
type: feature
description: 新增 Cloudflare Workers 部署链路（GitHub Actions 构建 + wrangler 直传），不再依赖平台方构建环境变量
---

## Cloudflare Workers 部署

- **新增部署工作流**：`.github/workflows/deploy-cloudflare.yml` 在 GitHub Actions 上完成 `pnpm build`（含图标生成与 Pagefind 索引），再用 `cloudflare/wrangler-action` 把 `dist/` 推送到 Cloudflare Workers。构建环境变量统一由 GitHub Secrets 提供，不依赖 Cloudflare 构建面板的变量配置。
- **新增静态资源配置**：仓库根目录 `wrangler.jsonc` 声明 `assets.directory = ./dist` 与 `not_found_handling = 404-page`（命中 Astro 生成的 `dist/404.html`）。项目为纯静态站点，无需 `@astrojs/cloudflare` 适配器，也没有 Worker 运行时脚本。
- **移除 `public/CNAME`**：该文件是 GitHub Pages 的自定义域名机制，内容为旧域名，在 Cloudflare 上会被当作普通静态文件对外提供，已删除。
- **环境变量本地化**：新增本地 `.env`（已被 `.gitignore` 的 `.env*` 规则忽略、从未被 git 跟踪），填好 `GATE_PASSWORD` 后 `pnpm build` 可自动读取，无需每次在命令行前缀注入。

> ⚠️ `GATE_PASSWORD` 是构建必填项：`src/components/security/EncryptGate.astro:22` 在 `securityConfig.enabled` 为 `true` 而密码缺失时直接中断构建，防止「以为加密了实际是明文」。密码决定密文内容，换密码必须重新构建，且丢失后无法恢复。
