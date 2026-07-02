---
title: "博客操作指南"
date: 2026-07-02T21:43:00+08:00
draft: false
summary: "Toha 主题博客的日常操作手册：写文章、改配置、升级主题、部署"
tags: ["指南", "Toha", "Hugo"]
---

这是一份**写给自己用**的操作手册，把日常要做的所有事都列在这里。忘了怎么操作就翻这篇。

---

## 一、写作相关

### 1. 创建新文章

```powershell
hugo new content content\posts\<文件名>.md
```

会自动生成 front matter 模板。`draft: true` 是草稿，**不会**出现在生产构建里。

预览草稿：

```powershell
hugo server -w -D
```

发布：把 `draft: true` 改成 `false`。

### 2. Front Matter 常用字段

```yaml
---
title: "文章标题"
date: 2026-07-02T21:43:00+08:00
draft: false
summary: "一句话描述（卡片预览用）"
tags: ["标签1", "标签2"]
categories: ["分类"]
author: "你的名字"
hero: /images/posts/<自定义图>.jpg
---
```

完整字段参考：https://toha-docs.netlify.app/posts/writing-posts/front-matter/

### 3. 文章分类与标签

```
content/posts/
├── tech-notes/
│   ├── go-concurrency.md
│   └── rust-ownership.md
└── life/
    └── 2026-reading-list.md
```

- **分类** = 文件夹（一篇文章只能有一个分类）
- **标签** = front matter 里的数组（一篇文章可以有多个）

### 4. 内置 shortcodes

```markdown
{{< alert type="warning" >}}
警告内容
{{< /alert >}}

{{< mermaid >}}
flowchart LR; A-->B-->C
{{< /mermaid >}}

{{< video src="/videos/demo.mp4" >}}

{{< split >}}
左半边内容
<<!--->>
右半边内容
{{< /split >}}
```

完整列表：https://toha-docs.netlify.app/posts/shortcodes/

### 5. 数学公式

Toha 用 KaTeX 自动渲染 markdown 里的公式，**不需要 shortcode**，配置好 delimiters 后直接写 LaTeX 语法就行。

`hugo.toml` 里启用（前面已经配过）：

```toml
[params.features.math]
  enable = true
  [params.features.math.services.katex]
    [[params.features.math.services.katex.delimiters]]
      left = "$$"
      right = "$$"
      display = true
    [[params.features.math.services.katex.delimiters]]
      left = "\\("
      right = "\\)"
      display = false
```

markdown 文件里直接写：

```markdown
行内公式：$E = mc^2$

块级公式：

$$
\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
$$
```

### 6. Mermaid 流程图

```markdown
---
mermaid: true
---

{{< mermaid >}}
graph TD; A-->B; B-->C
{{< /mermaid >}}
```

---

## 二、修改个人信息

所有个人/站点信息都在 `data/zh-cn/` 目录下，**改 yaml 文件即可**，不用动 hugo.toml：

| 文件 | 改什么 |
|---|---|
| `data/zh-cn/author.yaml` | 名字、昵称、头像、邮箱、GitHub、社交链接、一句话列表 |
| `data/zh-cn/site.yaml` | 版权、免责声明、站点描述、OpenGraph、自定义菜单 |
| `data/zh-cn/sections/about.yaml` | 自我介绍正文、社交链接、证书、软技能 |
| `data/zh-cn/sections/skills.yaml` | 技能清单（首页"技能"区） |
| `data/zh-cn/sections/experiences.yaml` | 工作经历（首页时间线） |
| `data/zh-cn/sections/education.yaml` | 教育背景 |
| `data/zh-cn/sections/projects.yaml` | 项目展示 |
| `data/zh-cn/sections/achievements.yaml` | 成就 |
| `data/zh-cn/sections/accomplishments.yaml` | 证书 + 软技能进度条 |

> 字段 schema 详细说明：https://toha-docs.netlify.app/posts/configuration/sections/

---

## 三、添加导航栏菜单

编辑 `data/zh-cn/site.yaml`：

```yaml
customMenus:
- name: 文章
  url: /posts/
  hideFromNavbar: false
  showOnFooter: false
- name: 关于本站
  url: /posts/blog-guide/
  hideFromNavbar: false
  showOnFooter: false
```

⚠️ **url 必须用站内路径**（`/posts/` 这种），不要写 `https://...`，否则会跳到外部站。

---

## 四、升级 Toha 主题

```powershell
# 拉最新版本
hugo mod get -u github.com/hugo-toha/toha/v4

# 同步 go.mod / go.sum
hugo mod tidy

# 重新生成 package.json（主题新增了依赖的话会同步过来）
hugo mod npm pack

# 装新依赖
npm install

# 重启服务验证
hugo server -w
```

锁定某个版本：

```powershell
hugo mod get github.com/hugo-toha/toha/v4@v4.16.0
```

---

## 五、换图片资源

**关键规则**：被 `resources.Get` 引用的图必须放 `assets/`，**不能放 `static/`**。

| 类型 | 放哪 | 例子 |
|---|---|---|
| 被主题用 `resources.Get` 加载的图 | `assets/images/...` | `assets/images/site/favicon.png`、`assets/images/author/john.png` |
| 直接 URL 暴露的图 | `static/...` | `static/files/resume.pdf`、`static/videos/sample.mp4` |

换头像流程：
1. 把新头像放到 `assets/images/author/`（覆盖旧文件）
2. 改 `data/zh-cn/author.yaml` 的 `image` 字段
3. 刷新页面

---

## 六、构建生产版本

```powershell
# 生成静态文件到 public/
hugo --gc --minify

# 部署：把 public/ 整个丢到 GitHub Pages / Vercel / Netlify / 服务器
```

构建前先确认 `hugo.toml` 里 `baseURL` 改成正式域名（不然 CSS/JS 路径会指向 localhost）。

---

## 七、添加评论系统

`hugo.toml` 里：

```toml
[params.features.comment]
  enable = true
  [params.features.comment.services.giscus]
    repo = "VimatinD/D-Blog"
    repoID = "xxx"          # 在 https://giscus.app/ 获取
    category = "Announcements"
    categoryID = "DIC_xxx"
```

支持 Disqus / Giscus / Utterances / Valine。

---

## 八、添加访问统计

```toml
[params.features.analytics]
  enable = true
  [params.features.analytics.services.google]
    id = "G-XXXXXXX"        # Google Analytics ID
```

支持 Google Analytics / GoatCounter / Matomo / Umami。

---

## 九、紧急修复速查

| 症状 | 原因 | 修法 |
|---|---|---|
| 启动报 `module not found` | Go 模块没拉全 | `hugo mod tidy` |
| 报 `Cannot find module 'bootstrap'` | npm 依赖没装 | `npm install` |
| SCSS 编译失败 | Hugo 不是 Extended | 重新装 `Hugo.Hugo.Extended` |
| 图片不显示 | 图放错目录 | 主题用 `resources.Get` 的图必须在 `assets/` |
| `nil pointer evaluating resource.Resource.RelPermalink` | 引用了不存在的资源 | 检查对应 yaml 里的 `image` / `logo` 路径 |
| 主题升级后页面挂了 | 没跑 `hugo mod npm pack && npm install` | 按"四、升级主题"流程再来一遍 |

---

## 十、关键文件清单（出问题先看这些）

```
hugo.toml              # 主配置
go.mod / go.sum        # Hugo Module 锁版本
package.json           # npm 依赖锁版本
data/zh-cn/*.yaml      # 站点数据（个人/各 section）
content/posts/         # 博客文章
assets/images/         # 主题用 resources.Get 读取的图片
static/                # 直接暴露的静态文件
```

---

## 参考链接

- Toha 文档：https://toha-docs.netlify.app/posts/
- Toha GitHub：https://github.com/hugo-themes/toha
- Hugo 中文文档：https://gohugo.io/documentation/
- 项目 GitHub：https://github.com/VimatinD/D-Blog