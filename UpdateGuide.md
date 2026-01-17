# MkDocs + GitHub Pages：维护与更新指南（全 Markdown 版）

本指南用于长期维护你的 **GitHub（源）+ MkDocs（站）+ 知乎/小红书（分发）** 内容体系。目标是形成稳定的“写作—预览—发布—传播”流水线。

---

## 1. 维护的核心原则

### 1.1 单一真源（Single Source of Truth）
- **GitHub 仓库**是唯一真源：所有内容都以仓库为准。
- **MkDocs 站点**是展示层：由仓库内容自动构建生成。

### 1.2 内容放置规则
- MkDocs 默认构建 `docs/` 目录下的内容。
- **所有要上站的 Markdown**必须位于 `docs/` 内（或其子目录）。

### 1.3 小步提交、频繁发布
- 每次更新尽量“小步”：一篇文章 / 一次修错 / 一次补充 → 一个 commit。
- 好处：易回溯、易定位问题、易维护节奏。

---

## 2. 每次更新标准流程（SOP）

> 适用场景：新增/修改文档、补充图片、更新导航、修复错误等。

### Step 0：前置条件（一次性准备）
- 已在本地打开仓库根目录（包含 `mkdocs.yml` 与 `docs/`）
- 本地已安装 MkDocs（你使用 conda 环境也可以）
- GitHub Actions 部署已配置：push 后会自动更新 Pages

---

### Step 1：同步远程最新版本
>如果本地已经是最新版本，则无需运行这一步

在 VS Code 终端、仓库根目录运行：

```bash
git pull
```
#### git 如何知道要拉取哪一个 repo？
关键在于你的本地仓库里保存了“远程仓库”的配置，叫 remote。默认通常叫 `origin`。
你可以在仓库根目录运行：
```bash
git remote -v
```
你会看到类似：
- `origin https://github.com/yeahsupercool/research-tools-handbook.git (fetch)`
- `origin https://github.com/yeahsupercool/research-tools-handbook.git (push)`

这就是 git pull 知道“repo 在哪里”的来源。
#### git 如何知道拉哪个分支？
取决于你当前所在分支以及这个分支设置的上游分支（upstream/tracking branch）。
查看你当前分支：
```bash
git branch
```
查看更完整信息（包括 upstream）：
```bash
git status
```
如果你的本地 main 跟踪 origin/main，那么你在 main 上运行：
```bash
git pull
```
Git 就会默认等价于：
```bash
git pull origin main
```
#### 如何明确指定拉取哪个 repo/分支？
```bash
git pull origin main
```
- origin：远程仓库名字
- main：远程分支名字

修改远程仓库的名称
```bash
git remote rename <old> <new>
```

### Step 2: 编辑内容
> 目标：把要发布到站点的内容，放入 `docs/` 并按主题组织，便于长期维护与检索。

#### 2.1 修改已有页面
- 在 VS Code 中直接打开并编辑 `docs/` 下对应的 `.md` 文件
- 保存即可（不需要额外命令）

#### 2.2 新增页面（推荐按主题归类）
1. 在 `docs/` 下选择合适的目录（不存在就新建）
2. 新建一个 `.md` 文件（建议使用 `kebab-case.md` 命名）

### Step 3：更新导航（mkdocs.yml）
> 如果新增页面，确保在 mkdocs.yml 的 nav: 中添加入口。

**注意：**
- nav 里的路径是相对 docs/ 的路径。
- 不要写 docs/xxx.md，应该写 xxx.md 或子目录路径。

### Step 4: 本地预览
在 VS Code 终端运行：
```bash
conda activate mkdocs # 激活虚拟环境
mkdocs serve
```
浏览器打开终端提示的地址（通常是 http://127.0.0.1:8000/ ），检查：
- 页面是否能打开
- 新增页面是否出现在侧边栏导航
- 文内链接是否可点击
- 图片是否显示
- 代码块排版是否正确

>停止预览：终端按 Ctrl + C

### Step 5: 提交到 Git（本地）
```bash
git status
git add .
git commit -m "Add SSH basics guide"
```
建议 commit message 规范：动词开头 + 简短说明，例如
- Add tmux cheatsheet
- Fix SSH config example
- Update remote dev guide

### Step 6: 推送到 GitHub
```bash
git push
```

### Step 7: 验证线上是否更新
#### 7.1 检查 GitHub Actions 是否成功
GitHub 仓库 → Actions：✅ 绿色对勾：部署成功
#### 7.2 打开 GitHub Pages 站点验证
访问 Pages 地址，确认页面内容已更新。
