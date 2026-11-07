# Spike 个人博客 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在阿里云上线 Hugo + Blowfish 个人博客「Spike」：空站、中文、跟随系统深浅色、页脚仅 GitHub，本地 Markdown 推送后 GitHub Actions 自动部署到 sslip.io HTTPS。

**Architecture:** 本地仓库 `/Users/dongzhiwei/spike-blog` 用 Hugo（≥0.162）+ Blowfish git submodule；push 到 `DDDong-11/spike-blog` 后 Actions 构建并 rsync 到服务器 `/var/www/spike-blog/`；nginx 独立 `server_name` + Let’s Encrypt，与「小猴教练」并存。

**Tech Stack:** Hugo extended ≥0.162.0、Blowfish（git submodule）、GitHub Actions、nginx、certbot、sslip.io、SSH deploy key

## Global Constraints

- 站点名：Spike；作者展示可用 Spike；GitHub：`https://github.com/DDDong-11`
- 语言：中文为主（`locale = "zh-cn"`）；无评论；无示例文章
- 首页：Blowfish 默认演示站气质；背景为星际牛仔 Spike 形象——**必须**用用户提供图或原创生成图，禁止直接抓取未授权动画截图上线
- 深浅色：跟随系统 + 手动切换（`defaultAppearance = "auto"`）
- 公网：`https://spike.139.224.252.156.sslip.io/`（可微调但需全计划一致）
- 密钥：部署密钥只进 GitHub Secrets / 服务器 authorized_keys，永不写入聊天或仓库明文
- 不改动「小猴教练」应用代码与其既有 server 逻辑；仅在 nginx 增加并列站点
- 仓库已有设计规格：`docs/superpowers/specs/2026-11-07-spike-blog-design.md`，保留勿删

## File map（将创建/修改）

| 路径 | 职责 |
| --- | --- |
| `config/_default/hugo.toml` | baseURL、locale、theme |
| `config/_default/languages.zh-cn.toml` | 站点标题、作者、GitHub 链接 |
| `config/_default/params.toml` | 配色、外观 auto、主页布局、背景图 |
| `config/_default/menus.zh-cn.toml` | 主导航（可极简） |
| `config/_default/markup.toml` | 从主题复制 |
| `themes/blowfish/` | Blowfish submodule |
| `content/_index.md` | 首页（空站最小内容） |
| `assets/img/` | 作者图、背景图 |
| `.github/workflows/deploy.yml` | 构建 + 部署 |
| 服务器 `/etc/nginx/sites-available/spike-blog` | HTTPS 静态站 |
| 服务器 `/var/www/spike-blog/` | 静态产物 |

---

### Task 1: 安装 Hugo 并脚手架站点骨架

**Files:**
- Create: Hugo 默认目录结构于 `/Users/dongzhiwei/spike-blog`（避开已有 `docs/` 与 `.git`）
- Modify: 无（本任务不改设计文档）

**Interfaces:**
- Consumes: 已有 git 仓库 `main` @ `/Users/dongzhiwei/spike-blog`
- Produces: 可运行的空 Hugo 项目目录（尚无主题）；`hugo version` ≥ 0.162.0；`hugo.toml` 临时文件稍后删除

- [ ] **Step 1: 安装 Hugo extended（Homebrew）**

```bash
brew install hugo
hugo version
```

Expected: 输出版本号 ≥ `0.162.0`，且含 `extended` 或 `+extended`。

- [ ] **Step 2: 在已有仓库内生成站点文件（不覆盖 docs）**

```bash
cd /Users/dongzhiwei/spike-blog
hugo new site . --force
```

Expected: 出现 `archetypes/`、`content/`、`layouts/`、`static/`、`hugo.toml` 等；`docs/superpowers/` 仍在。

- [ ] **Step 3: 确认 git 状态干净可跟踪**

```bash
cd /Users/dongzhiwei/spike-blog
git status
```

Expected: 新文件为 untracked；`docs/superpowers/specs/2026-11-07-spike-blog-design.md` 仍被跟踪。

- [ ] **Step 4: Commit 骨架**

```bash
cd /Users/dongzhiwei/spike-blog
git add archetypes content layouts static hugo.toml
# 若生成了其他默认文件一并 add，但不要 add 密钥
git commit -m "$(cat <<'EOF'
chore: scaffold Hugo site skeleton

Keep existing design docs; prepare for Blowfish theme install.
EOF
)"
```

---

### Task 2: 安装 Blowfish 主题并复制配置

**Files:**
- Create: `themes/blowfish` (submodule)
- Create: `config/_default/*.toml`（从主题复制）
- Delete: 根目录 `hugo.toml`（主题要求改用 config/_default）

**Interfaces:**
- Consumes: Task 1 的 Hugo 骨架
- Produces: `theme = "blowfish"` 生效；`config/_default/` 完整；本地 `hugo server` 可起空白站

- [ ] **Step 1: 添加 Blowfish submodule**

```bash
cd /Users/dongzhiwei/spike-blog
git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish
```

Expected: `themes/blowfish` 存在且为 submodule。

- [ ] **Step 2: 复制主题默认配置**

```bash
cd /Users/dongzhiwei/spike-blog
mkdir -p config/_default
rm -f hugo.toml
cp themes/blowfish/config/_default/*.toml config/_default/
ls config/_default/
```

Expected: 至少含 `hugo.toml`、`params.toml`、`markup.toml`、`languages.en.toml`、`menus.en.toml`。

- [ ] **Step 3: 在 `config/_default/hugo.toml` 顶部启用主题并设 baseURL/locale**

将 `config/_default/hugo.toml` 调整为至少包含（保留主题其它默认项，按需合并）：

```toml
theme = "blowfish"
baseURL = "https://spike.139.224.252.156.sslip.io/"
locale = "zh-cn"
# 其余保持从主题复制的合理默认；defaultContentLanguage 若存在则设为 zh-cn
defaultContentLanguage = "zh-cn"
```

- [ ] **Step 4: 本地启动验证主题加载**

```bash
cd /Users/dongzhiwei/spike-blog
hugo server -D --bind 127.0.0.1 --port 1313
```

Expected: 无 fatal；浏览器可开 `http://127.0.0.1:1313/`（可能仍是英文默认文案，下任务改）。验证后 Ctrl+C。

- [ ] **Step 5: Commit**

```bash
cd /Users/dongzhiwei/spike-blog
git add .gitmodules themes/blowfish config
git commit -m "$(cat <<'EOF'
feat: add Blowfish theme submodule and default config

EOF
)"
```

---

### Task 3: 中文、站点身份、外观与空首页

**Files:**
- Create: `config/_default/languages.zh-cn.toml`
- Create: `config/_default/menus.zh-cn.toml`
- Modify: `config/_default/params.toml`
- Create: `content/_index.md`
- Optional remove/ignore: `languages.en.toml` / `menus.en.toml`（若保留，确保 defaultContentLanguage 为 zh-cn 且不造成双语言切换 UI；本期不做双语，优先重命名为 zh-cn 并删除 en 文件以免混淆）

**Interfaces:**
- Consumes: Blowfish 配置文件结构
- Produces: 标题 Spike；作者链接仅 GitHub；`defaultAppearance = "auto"`；空首页无 posts

- [ ] **Step 1: 写入中文语言配置**

创建 `config/_default/languages.zh-cn.toml`：

```toml
title = "Spike"

[params.author]
name = "Spike"
# image 有头像后再填，例如 "img/author.jpg"
headline = "技术为主的个人笔记"
bio = ""
links = [
  { github = "https://github.com/DDDong-11" },
]
```

删除或移走 `config/_default/languages.en.toml`，避免误用英文为主语言。

- [ ] **Step 2: 写入中文菜单（极简，无评论）**

创建 `config/_default/menus.zh-cn.toml`：

```toml
[[main]]
  name = "首页"
  pageRef = "/"
  weight = 10

[[main]]
  pre = "github"
  name = "GitHub"
  url = "https://github.com/DDDong-11"
  weight = 20
```

删除 `menus.en.toml`（若存在）。不要添加评论相关菜单。

- [ ] **Step 3: 调整 `params.toml` 关键项**

在 `config/_default/params.toml` 中设置（保留文件内其它注释默认，确保下列生效）：

```toml
colorScheme = "blowfish"
defaultAppearance = "auto"
autoSwitchAppearance = true
# 主页布局：对齐官方默认演示站；若主题默认已是 profile/background，保持与 blowfish.page 演示一致的 homepage 布局键
# 背景图路径在 Task 4 放入 assets 后取消注释，例如：
# defaultBackgroundImage = "img/spike-bg.jpg"
# 关闭不需要的功能：不启用 comments 相关 provider
```

查阅当前 Blowfish `params.toml` 注释，确认演示站默认的 `homepage` / layout 相关键名并与官方 demo 对齐（勿臆造键名；以主题文件注释为准）。

- [ ] **Step 4: 空首页，不创建 posts 文章**

创建 `content/_index.md`：

```markdown
---
title: "Spike"
---
```

不要创建 `content/posts/` 下的示例 md。若 `hugo new site` 生成了占位内容且含示例文，删除示例文，仅保留空结构。

- [ ] **Step 5: 本地验证**

```bash
cd /Users/dongzhiwei/spike-blog
hugo server -D --bind 127.0.0.1 --port 1313
```

Expected: 标题为 Spike；可见 GitHub 链接；深浅色可随系统/手动切换；无文章列表条目（或空列表）；无评论组件。

- [ ] **Step 6: Commit**

```bash
git add config content
git commit -m "$(cat <<'EOF'
feat: configure Chinese Spike identity and empty homepage

EOF
)"
```

---

### Task 4: Spike 背景图（授权安全）

**Files:**
- Create: `assets/img/spike-bg.jpg`（或 `.png`）
- Modify: `config/_default/params.toml`（`defaultBackgroundImage`）

**Interfaces:**
- Consumes: 用户提供的图片文件，或本会话用图像生成工具生成的**原创**插画（星际牛仔气质，非复制官方帧）
- Produces: 首页背景生效；规格中的版权约束满足

- [ ] **Step 1: 获取图片**

优先：若用户已提供文件，复制到 `assets/img/spike-bg.jpg`。  
否则：用图像生成生成一张原创「黑衣烟枪、冷色赛博、侧脸剪影」风格插画（不复制具体官方剧照构图），保存到 `assets/img/spike-bg.jpg`。  
上线前用一句话告知用户「当前为原创生成图；若要换成你的图随时替换同路径文件」。

- [ ] **Step 2: 配置背景**

在 `params.toml` 设置（键名以主题实际支持为准，常见为）：

```toml
defaultBackgroundImage = "img/spike-bg.jpg"
```

若主题要求放在 `assets/` 且 homepage layout 另有 `backgroundImage` 字段，按主题注释补全，保证首页可见。

- [ ] **Step 3: 本地验证后 Commit**

```bash
cd /Users/dongzhiwei/spike-blog
hugo server -D --bind 127.0.0.1 --port 1313
# 目视确认背景
git add assets/img config/_default/params.toml
git commit -m "$(cat <<'EOF'
feat: add Spike-inspired homepage background image

EOF
)"
```

---

### Task 5: 创建 GitHub 仓库并推送

**Files:**
- Remote: `https://github.com/DDDong-11/spike-blog.git`
- Modify: git remote

**Interfaces:**
- Consumes: 本地 `main` 含主题 submodule
- Produces: 远程仓库可读；submodule 可被 Actions checkout

- [ ] **Step 1: 创建公开仓库**

使用已连接的 GitHub 能力（`gh` 或 cursor-github）创建：

```bash
cd /Users/dongzhiwei/spike-blog
gh repo create DDDong-11/spike-blog --public --source=. --remote=origin --description "Spike personal blog (Hugo + Blowfish)"
```

若 `origin` 已存在则先检查 URL。不要 `--push` 若 submodule 未就绪；下一步再 push。

- [ ] **Step 2: 推送含 submodule**

```bash
cd /Users/dongzhiwei/spike-blog
git push -u origin main
```

Expected: GitHub 上可见代码与 `themes/blowfish` submodule 指针。

- [ ] **Step 3: 记录仓库 URL 供用户**

仓库：`https://github.com/DDDong-11/spike-blog`

---

### Task 6: 服务器静态目录与 nginx + HTTPS

**Files:**
- Create on server: `/var/www/spike-blog/`
- Create: `/etc/nginx/sites-available/spike-blog`
- Enable: `/etc/nginx/sites-enabled/spike-blog`
- Cert: Let’s Encrypt for `spike.139.224.252.156.sslip.io`

**Interfaces:**
- Consumes: SSH `Lab-Server` → `root@139.224.252.156`
- Produces: HTTP 可访问占位页；证书签发后 HTTPS 可用；小猴教练站点仍 200

- [ ] **Step 1: 创建目录与占位**

```bash
ssh Lab-Server 'mkdir -p /var/www/spike-blog && echo "Spike blog deploy pending" > /var/www/spike-blog/index.html && chown -R www-data:www-data /var/www/spike-blog || chown -R nginx:nginx /var/www/spike-blog'
```

- [ ] **Step 2: 写入 nginx server（先 HTTP 便于 certbot）**

`/etc/nginx/sites-available/spike-blog`：

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name spike.139.224.252.156.sslip.io;

    root /var/www/spike-blog;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
ssh Lab-Server 'ln -sf /etc/nginx/sites-available/spike-blog /etc/nginx/sites-enabled/spike-blog && nginx -t && systemctl reload nginx'
```

- [ ] **Step 3: 签发证书**

```bash
ssh Lab-Server 'certbot --nginx -d spike.139.224.252.156.sslip.io --non-interactive --agree-tos --register-unsafely-without-email || certbot --nginx -d spike.139.224.252.156.sslip.io'
```

Expected: HTTPS 打开占位页。若 sslip.io 校验失败，排查 80 端口与云安全组，修好后重试。

- [ ] **Step 4: 回归小猴教练**

```bash
curl -sI https://xiaohou.139.224.252.156.sslip.io/ | head -5
curl -sI http://139.224.252.156/ | head -5
```

Expected: 仍为成功响应（非长时间超时/502）。

---

### Task 7: 部署密钥与 GitHub Actions

**Files:**
- Create: `.github/workflows/deploy.yml`
- Server: deploy 用户或 root 的 authorized_keys 中**专用**公钥
- GitHub Secrets: `DEPLOY_HOST`、`DEPLOY_USER`、`DEPLOY_SSH_KEY`、`DEPLOY_PATH`

**Interfaces:**
- Consumes: Task 5 仓库；Task 6 目录
- Produces: push `main` 后自动更新 `/var/www/spike-blog/`

- [ ] **Step 1: 在本机生成专用部署密钥（勿打印私钥到聊天）**

```bash
ssh-keygen -t ed25519 -f /Users/dongzhiwei/.ssh/spike_blog_deploy -N "" -C "spike-blog-deploy"
```

把**公钥**追加到服务器 `authorized_keys`；把**私钥**写入 GitHub Secret `DEPLOY_SSH_KEY`（用 `gh secret set`，不要 cat 私钥到对话）。

```bash
# 公钥上服务器（示例）
ssh Lab-Server 'mkdir -p /root/.ssh && chmod 700 /root/.ssh'
# 使用 ssh-copy-id 或安全追加 spike_blog_deploy.pub
gh secret set DEPLOY_SSH_KEY < /Users/dongzhiwei/.ssh/spike_blog_deploy
gh secret set DEPLOY_HOST -b "139.224.252.156"
gh secret set DEPLOY_USER -b "root"
gh secret set DEPLOY_PATH -b "/var/www/spike-blog"
```

（在仓库目录执行 `gh secret set`，确保作用域为 `spike-blog`。）

- [ ] **Step 2: 写入 workflow**

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy Spike blog

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "0.162.0"
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy via rsync
        uses: burnett01/rsync-deployments@7.0.1
        with:
          switches: -avz --delete
          path: public/
          remote_path: ${{ secrets.DEPLOY_PATH }}/
          remote_host: ${{ secrets.DEPLOY_HOST }}
          remote_user: ${{ secrets.DEPLOY_USER }}
          remote_key: ${{ secrets.DEPLOY_SSH_KEY }}
```

若 `burnett01/rsync-deployments` 版本需调整，选用当时仍维护的 major；或改为 `appleboy/scp-action` + ssh，但须同样不把密钥写入日志。

- [ ] **Step 3: Commit 并推送触发首次部署**

```bash
cd /Users/dongzhiwei/spike-blog
git add .github/workflows/deploy.yml
git commit -m "$(cat <<'EOF'
ci: add Hugo build and rsync deploy workflow

EOF
)"
git push origin main
```

- [ ] **Step 4: 验证 Actions 与线上**

```bash
gh run list --limit 3
curl -sI https://spike.139.224.252.156.sslip.io/ | head -10
```

Expected: workflow 成功；HTTPS 返回 Blowfish 首页而非占位句。

---

### Task 8: 端到端验收与交付说明

**Files:**
- Modify: 无代码要求；可更新本计划 checkboxes
- Optional: `README.md` 简短发文说明（中文）

**Interfaces:**
- Consumes: 已部署站点
- Produces: 给用户的三句话：仓库 URL、线上 URL、如何新建文章

- [ ] **Step 1: 验收清单（全部手动确认）**

1. `https://spike.139.224.252.156.sslip.io/` HTTPS 正常  
2. 标题 Spike；中文；GitHub 页脚/菜单正确  
3. 深浅色跟随系统且可切换  
4. 背景为 Spike 形象图  
5. 无示例文章、无评论  
6. 小猴教练原 URL 仍可用  
7. 本地新建测试文 → push → 线上出现 → 再删测试文 push（保持空站约定）**或**仅用 `workflow_dispatch` 验证构建，不留测试文

- [ ] **Step 2: 写最短 README（可选但推荐）**

`README.md`：

```markdown
# Spike

个人博客：Hugo + Blowfish。

- 线上：https://spike.139.224.252.156.sslip.io/
- 本地预览：`hugo server -D`
- 新文章：`hugo new content posts/my-post/index.md` 后编辑并 `git push`
```

- [ ] **Step 3: Commit README（若创建）并最终向用户汇报上述链接与发文步骤**

```bash
git add README.md
git commit -m "docs: add short usage README"
git push origin main
```

---

## Spec coverage（自检）

| 规格要求 | 对应任务 |
| --- | --- |
| Hugo + Blowfish | Task 1–2 |
| 名 Spike、中文、空站、无评论 | Task 3 |
| 默认站气质 + Spike 背景 | Task 3–4 |
| 深浅色 auto | Task 3 |
| GitHub 仅 DDDong-11 | Task 3 |
| sslip.io HTTPS | Task 6 |
| Markdown → GitHub → 自动部署 | Task 5、7 |
| 与小猴教练并存 | Task 6 回归 |
| 密钥不进聊天 | Task 7 |

## Placeholder scan

无 TBD/TODO；图片授权路径已写明优先用户文件/原创生成。
