# Spike 个人博客设计规格

日期：2026-11-07  
状态：待用户确认后进入实施计划

## 目标

在阿里云服务器上搭建个人博客站点「Spike」：视觉与交互对齐 Hugo 主题 Blowfish 默认演示站；本地 Markdown 写作，推送到 GitHub 后自动构建并部署。与现有「小猴教练」应用并存，互不影响。

## 已确认决策

| 项 | 选择 |
| --- | --- |
| 气质 / 主题 | Hugo + Blowfish 官方主题 |
| 站点名 | Spike（后续可改） |
| 内容 | 混合，以技术为主；中文为主 |
| 首页 | Blowfish 默认演示站布局 |
| 背景 | 星际牛仔 Spike 形象（素材需可合法使用或由用户提供） |
| 深浅色 | 跟随系统，可手动切换 |
| 评论 | 不要 |
| 社交 | 页脚仅 GitHub：DDDong-11 |
| 访问 | 免费临时域名 sslip.io + HTTPS |
| 发文流程 | 本地 Markdown → push GitHub → 自动部署 |
| 首发内容 | 空站，不放示例文章 |

## 架构

```
本地 Hugo 站点 (Blowfish)
        │ git push
        ▼
GitHub 仓库 DDDong-11/spike-blog (可改名)
        │ GitHub Actions 构建静态文件
        ▼
阿里云 139.224.252.156
  nginx 独立 server_name
  文档根目录托管 public/
  Let’s Encrypt (sslip.io)
```

- 临时域名（可微调）：`spike.139.224.252.156.sslip.io`
- 与「小猴教练」：不同 `server_name`、不同站点根目录；不改动其进程与端口绑定策略以外的既有配置，除非为共用 nginx 增加一条 server 块
- 服务器只对外开 80/443；部署用 SSH 密钥放在 GitHub Secrets，不进仓库、不进聊天

## 外观与内容

- 主题：Blowfish；首页布局跟随官方默认演示站
- 背景图：星际牛仔 Spike；优先使用用户自提供文件，否则选用可明确授权的素材并在上线前请用户确认；不使用来源不明的抓取图
- 语言：界面与内容默认中文
- 颜色模式：`auto`（跟随系统）+ 页面内切换
- 页脚：GitHub 链接 `https://github.com/DDDong-11`
- 无评论系统；无分析脚本（除非后续单独要求）
- 仓库初始无示例文章；可保留 About 等主题必需的最小页面结构（若主题要求），但不塞入演示博文

## 部署与发布

1. 本机安装 Hugo（扩展版，满足 Blowfish 版本要求）与主题（git submodule 或官方推荐方式）
2. 初始化站点配置：`title=Spike`、中文、`colorScheme`/暗色跟随系统、社交仅 GitHub
3. 创建 GitHub 仓库并推送
4. 编写 Actions：`hugo` 构建 → rsync/scp 到服务器约定目录 → 可选 reload nginx
5. 服务器：nginx server 块 + certbot 为 sslip.io 域名签证书
6. 验收：HTTPS 可打开空首页、深浅色可切换、页脚 GitHub 正确、小猴教练原站点仍正常

## 明确不做（本期）

- 正式域名、评论、多语言切换 UI、示例文章、动态 CMS、从零仿 Blowfish 的自研主题

## 风险与注意

- **版权**：动画角色形象可能涉及版权；上线前必须确认图片授权或改用用户自有图
- **SSH/CI**：部署密钥最小权限，仅用于博客发布目录
- **与小猴教练共存**：改 nginx 前备份现有站点配置

## 成功标准

- 公网 HTTPS 打开 Spike 博客首页（Blowfish 默认布局气质）
- 背景为已确认的 Spike 形象图
- push 一篇新 Markdown 后 Actions 自动更新线上站点
- 小猴教练原访问方式仍可用
