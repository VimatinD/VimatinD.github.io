# Hugo + Toha 本地部署指南（Windows / PowerShell · Hugo Module 版）

> 目标：在 `F:\hugo\D-Blog` 搭建基于 **Toha v4** 主题的 Hugo 个人博客，走 **Hugo Module** 路线，长期维护省心。

## 0. 路线说明

经过对比（详见对话历史），选 Hugo Module，理由：

- ✅ Toha 官方文档唯一支持的方式
- ✅ 主题升级一条命令搞定
- ✅ 新增 npm 依赖自动同步到 package.json
- ✅ 同事/未来的你 clone 仓库后 `hugo mod tidy` 就齐全
- ✅ 出问题搜资料容易（全网都默认这条路）

代价：多装一个 Go（~150MB），多两个文件 `go.mod` / `go.sum`。可接受。

---

## 1. 安装 Hugo Extended / Go / Node

你机器上有 **winget**，管理员 PowerShell 一次到位：

```powershell
winget install Hugo.Hugo.Extended -s winget
winget install GoLang.Go -s winget
winget install OpenJS.NodeJS.LTS -s winget
```

新开 PowerShell 窗口验证：

```powershell
hugo version
# 必须含 "extended"，例如：hugo v0.1xx.x+extended windows/amd64 BuildDate=...

go version
# go version go1.xx.x windows/amd64

node --version
# v18.x 或 v20.x

npm --version
# 8.x 或更高
```

**任何一项看不到对应版本号**：
- Hugo 没装好：重新装 `Hugo.Hugo.Extended`（不是 `Hugo.Hugo`）
- Go 没装好：检查 PATH 是否包含 `%LOCALAPPDATA%\Programs\Go\bin`（winget 默认装这里）
- Node 没装好：检查 `%ProgramFiles%\nodejs\`

---

## 2. 准备仓库

工作区 `F:\hugo\D-Blog` 已经有 Hugo 骨架（`hugo.toml` + 空目录），**不要重新 `hugo new site`**，直接用。

初始化 git（方便 Hugo Module 工作）：

```powershell
cd F:\hugo\D-Blog
git init
```

如果要部署到 GitHub Pages，先在远端建空仓库，再：

```powershell
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
```

---

## 3. 改 `hugo.toml`

工作区已有 `hugo.toml`，**覆盖**为：

```toml
baseURL = "https://example.org/"
defaultContentLanguage = "en"
title = "我的博客"

# Hugo Module 配置（唯一主题来源，不再写 theme = "toha"）
[module]
  [[module.imports]]
    # ⚠️ 注意：Toha 仓库 GitHub URL 是 hugo-themes/toha，
    # 但 module path 用的是旧的组织名 hugo-toha/toha/v4（go.mod 里声明的）
    path = "github.com/hugo-toha/toha/v4"

  # 把 node_modules 里的字体/图标挂到 Hugo 的 static 目录
  [[module.mounts]]
    source = "static/files"
    target = "static/files"
  [[module.mounts]]
    source = "./node_modules/flag-icons/flags"
    target = "static/flags"
  [[module.mounts]]
    source = "./node_modules/@fontsource/mulish/files"
    target = "static/files"
  [[module.mounts]]
    source = "./node_modules/katex/dist/fonts"
    target = "static/fonts"

[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true

[params]
  background = "/images/site/background.jpg"
  [params.logo]
    main = "/images/site/main-logo.png"
    inverted = "/images/site/inverted-logo.png"
    favicon = "/images/site/favicon.png"
  [params.features]
    [params.features.theme]
      enable = true
      services = { light = true, dark = true, default = "system" }
    [params.features.blog]
      enable = true
      showAuthor = true
  [params.footer]
    enable = true
```

关键字段：
- **`[module.imports]`** —— 告诉 Hugo 去拉哪个主题模块（**注意：不要再写 `theme = "toha"`**，module 模式下主题名由 module 自动识别，写了反而冲突会去 `themes/toha/` 找本地主题然后报错）
- **`[module.mounts]`** —— 把 npm 装的字体/图标挂成 Hugo 的 static 资源，没这段字体和图标会 404

---

## 4. 初始化 Hugo Module + 拉取主题

```powershell
cd F:\hugo\D-Blog
hugo mod init github.com/<你的用户名>/<你的仓库名>
hugo mod tidy
```

这两条命令会：
- 在仓库根创建 `go.mod`（Module 标识）
- 把 Toha 主题从 GitHub 拉下来到 Go module 缓存
- 创建 `go.sum`（依赖校验）

> **Module 路径**填什么？
> - 要部署到 GitHub Pages：`github.com/<用户名>/<仓库名>`
> - 只本地玩玩：`github.com/local/blog`
> - 不打算用远端：`example.com/myblog`

---

## 5. 生成前端 package.json + 装依赖

```powershell
hugo mod npm pack    # 在仓库根生成 package.json
npm install          # 下载 bootstrap、katex、mermaid 等到 node_modules/
```

> 第一次 `npm install` 会下载约 80MB 依赖，1-3 分钟。
>
> 建议把 `node_modules/`、`package-lock.json`、`public/`、`resources/` 加进 `.gitignore`：
> ```powershell
> @"
> node_modules/
> package-lock.json
> public/
> resources/
> .hugo_build.lock
> "@ | Out-File -Encoding utf8 .gitignore
> ```

---

## 6. 跑起来

```powershell
hugo server -w
```

浏览器打开 `http://localhost:1313/`，第一次会看到大量 SCSS 编译日志 + esbuild 输出，**正常**。等页面渲染出来就 OK。

- `-w`：watch，文件改动自动刷新
- `-D`：含 draft 文章（默认不渲染草稿）

---

## 7. 配置站点（数据驱动）

Toha 用 `data/toha/*.yaml` 驱动首页各个 section，**不是 `hugo.toml` 里的 `[params]`**。

最快上手：**fork 官方示例站拷数据**：

```powershell
# 临时克隆示例站
git clone --depth 1 https://github.com/hugo-themes/toha-example-site.git `
  C:\Users\dhq_1\AppData\Local\Temp\toha-example

# 拷数据
New-Item -ItemType Directory -Path data\toha -Force
Copy-Item -Recurse `
  C:\Users\dhq_1\AppData\Local\Temp\toha-example\data\toha\* `
  data\toha\

# 拷示例内容（可选用来看默认效果）
Copy-Item -Recurse -Force `
  C:\Users\dhq_1\AppData\Local\Temp\toha-example\content\* `
  content\
```

要编辑的清单：

| 文件 | 控制什么 |
|---|---|
| `data/toha/site.yaml` | 站点标题、副标题、Logo、社交链接 |
| `data/toha/about.yaml` | "About" 板块 |
| `data/toha/skills.yaml` | "Skills" 卡片 |
| `data/toha/experiences.yaml` | 工作/项目时间线 |
| `data/toha/projects.yaml` | "Projects" 卡片网格 |
| `data/toha/achievements.yaml` | "Achievements" 区块 |
| `data/toha/education.yaml` | "Education" 时间线 |
| `data/toha/social.yaml` | 社交链接 |

字段 schema：[Toha 配置文档](https://toha-docs.netlify.app/posts/configuration/)

---

## 8. 写第一篇博客

```powershell
hugo new content content/posts/my-first-post.md
```

生成的 front matter：

```yaml
---
title: "My First Post"
date: 2026-07-02T19:49:00+08:00
draft: true
---
```

写正文。`hugo server -w -D` 预览草稿；发布把 `draft: true` 删掉或改成 `false`。

---

## 9. 升级主题（Module 路线的核心优势）

```powershell
cd F:\hugo\D-Blog
hugo mod get -u github.com/hugo-toha/toha/v4    # 拉最新版本
hugo mod tidy
hugo mod npm pack                                # 同步 package.json
npm install                                      # 同步 node_modules
hugo server -w
```

如果想**锁定某个版本**（比如团队协作、生产部署）：

```powershell
# 锁定到 v4.16.0
hugo mod get github.com/hugo-toha/toha/v4@v4.16.0
```

---

## 10. 常用命令速查

```powershell
hugo server -w              # 本地预览（监听文件变化）
hugo server -w -D           # 本地预览（含 draft）
hugo                        # 生成静态站点到 public/
hugo --gc --minify          # 生产构建（清理缓存 + 压缩）
hugo mod tidy               # 同步 go.mod / go.sum
hugo mod get -u <module>    # 升级指定模块
hugo mod npm pack           # 重新生成 package.json
npm install                 # 同步 node_modules
```

---

## 11. 故障排查

### Q: `hugo mod tidy` 报 "module not found" / 网络错
**A:** 检查 GitHub 是否可达。Module 是从 `proxy.golang.org` 拉的，国内偶尔抽风，可设环境变量换源：
```powershell
$env:GOPROXY = "https://goproxy.cn,direct"
hugo mod tidy
```

### Q: 启动报 "Cannot find module 'bootstrap'" / 'katex' / 'mermaid'
**A:** 没装 npm 依赖。`npm install`，确认 `node_modules/bootstrap/` 存在。

### Q: 启动报 SCSS 编译失败
**A:** 99% 是 Hugo 不是 Extended 版。`hugo version` 必须有 `extended` 字样。

### Q: 升级主题后页面挂了
**A:** 99% 是忘了跑 `hugo mod npm pack && npm install`。两步缺一不可。

### Q: 改了 `themes/toha/` 里的文件但没生效
**A:** Module 模式下主题在 Go 缓存里，**不要去改**。要改就在站点根的 `layouts/`、`assets/`、`static/` 里加同名文件覆盖（Hugo 优先级：站点 > 主题）。

### Q: 主题想自定义怎么办？
**A:** 在站点根创建同名文件覆盖 Toha 的布局/资源。例如改导航栏颜色：
- 创建 `assets/css/custom.css`，写你的样式
- 创建 `layouts/partials/header.html` 引入 `custom.css`
- 完事，下次升级主题不会冲突

---

## 12. 部署（可选）

`hugo.toml` 里 `baseURL` 改成正式域名：

```toml
baseURL = "https://yourname.github.io/"
```

构建命令：

```powershell
hugo --gc --minify
```

把 `public/` 整个丢到 GitHub Pages / Vercel / Netlify 都能跑。

**Netlify 一键部署**（推荐）：
1. 把代码推到 GitHub
2. Netlify 选 "Import from Git"
3. Build command: `hugo --gc --minify`
4. Build environment: `HUGO_VERSION = 0.xxx.x`（指定 Hugo 版本）
5. 完成

---

## TL;DR（装好三件套后）

```powershell
cd F:\hugo\D-Blog

# hugo.toml 覆盖：theme = "toha" + module.imports 段

git init
hugo mod init github.com/<你的用户名>/<你的仓库名>
hugo mod tidy
hugo mod npm pack
npm install
hugo server -w
```

打开 http://localhost:1313 🎉

---

## 参考链接

- Toha 主题：https://github.com/hugo-themes/toha
- Toha 配置文档：https://toha-docs.netlify.app/posts/configuration/
- Toha 示例站：https://github.com/hugo-themes/toha-example-site
- Hugo Module 文档：https://gohugo.io/hugo-modules/
- Hugo 中文文档：https://gohugo.io/documentation/