# Cloudflare Random Image API

一个基于 Cloudflare Pages 和 Transform Rules 实现的**无限流量、零成本、多分类**随机图片 API。

## 🌟 原理

利用 Cloudflare 的边缘重写能力（Rewrite URL），将用户的分类请求（如 `/h`）动态映射到预生成的静态资源路径（如 `/h/a1b.jpg`）。整个过程在边缘节点完成，无需服务器后端，无需 Worker 调用额度。

## 📂 目录结构

```text
├── oriImg/           # 原始图片素材目录
│   ├── h/            # 示例：横屏图片分类
│   └── v/            # 示例：竖屏图片分类
├── dist/             # 生成的静态资源目录（部署此目录）
├── gen_img.py        # 资源生成脚本
└── README.md         # 说明文档
```

## 🚀 部署指南

### 1. 准备素材
在 `oriImg` 目录下建立你的分类文件夹（例如 `h`, `pc`, `mobile` 等），并将对应的图片放入其中。
> 支持 `.jpg`, `.png`, `.webp` 等常见格式。

### 2. 生成静态库
运行 Python 脚本，它会将图片扩充并重命名为十六进制哈希文件名（`000.jpg` ~ `fff.jpg`），以适配 Cloudflare 的随机逻辑。

```bash
python gen_img.py
```
*脚本会在 `dist/` 目录下生成处理好的文件，每个分类包含 4096 个文件（16^3）。*

### 3. 部署到 Cloudflare Pages
将 `dist` 目录下的内容部署到 Cloudflare Pages。
* 如果使用 Git 集成，确保 Build output directory 设置为 `dist`（如果你把 dist 提交了）或者在构建命令中运行生成脚本。
* **推荐**：直接在本地运行脚本后，将 `dist` 目录作为静态站点上传，或者仅提交 `dist` 目录内容。

### 4. 配置 Cloudflare Rules (关键)

进入你的 Cloudflare 域名管理面板：
1. 导航到 **Rules** > **Transform Rules**。
2. 点击 **Create rule**，选择 **Rewrite URL**。
3. 配置如下：

* **Rule name**: Random Image
* **Filter Expression**: 
  * 建议匹配你的 API 路径，例如：
  * `URI Path` matches `^/[a-zA-Z0-9_-]+$` 
  * *(这会匹配 /h, /v 等一级路径)*
* **Path Rewrite** (选择 **Dynamic**):
  * 在输入框中填入以下表达式：
  ```text
  concat(http.request.uri.path, "/", substring(uuidv4(cf.random_seed), 0, 3), ".jpg")
  ```

> **⚠️ 注意**：此规则假设你的请求路径（如 `/h`）直接对应 `dist` 下的文件夹名。规则会自动拼接路径，生成如 `/h/1a2.jpg` 的重写地址。

## 🔗 使用示例

假设你的域名是 `img.yourdomain.com`，且你在 `oriImg/h` 中放入了图片。

* **访问地址**: `https://img.yourdomain.com/h`
* **效果**: Cloudflare 内部重写为 `dist/h/xxx.jpg`，返回一张随机横屏图片。

同理，如果你有 `oriImg/acg` 目录：
* **访问地址**: `https://img.yourdomain.com/acg`
