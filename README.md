# DEMO的小站 - 首页部署指南

## 文件说明

| 文件 | 用途 |
|------|------|
| `index.html` | 完整的首页（需托管后使用） |
| `embed.html` | 简化版（可直接粘贴到 new-api 设置，但样式可能受限） |
| `docker-compose.homepage.yml` | Docker 部署配置 |

---

## 方案一：GitHub Pages（推荐 ⭐）

**优点**：免费、HTTPS、无需 VPS 配置、CDN 加速

### 步骤

1. **登录 GitHub**，点击右上角 `+` → `New repository`

2. **创建仓库**
   - Repository name: `homepage`（或其他名称）
   - 选择 `Public`
   - 点击 `Create repository`

3. **上传文件**
   - 点击 `uploading an existing file`
   - 将 `index.html` 拖入页面
   - 点击 `Commit changes`

4. **启用 GitHub Pages**
   - 进入仓库 → `Settings` → 左侧 `Pages`
   - Source 选择 `Deploy from a branch`
   - Branch 选择 `main` → `/ (root)` → `Save`

5. **获取地址**
   - 等待 1-2 分钟
   - 页面顶部显示：`Your site is live at https://你的用户名.github.io/homepage/`

6. **配置 new-api**
   - 登录 new-api 管理后台
   - 系统设置 → 个性化设置 → 首页内容
   - 填入：`https://你的用户名.github.io/homepage/index.html`
   - 点击「设置首页内容」

---

## 方案二：VPS Nginx 容器

**优点**：完全自主控制，可使用自己的域名

### 步骤

1. **上传文件到 VPS**

```bash
# 在 VPS 上创建目录
mkdir -p ~/homepage
cd ~/homepage

# 方法 A：使用 scp 从本地上传
# 在本地执行：
scp homepage/index.html homepage/docker-compose.homepage.yml root@你的VPS地址:~/homepage/

# 方法 B：直接在 VPS 上创建文件
# 复制 index.html 内容，然后：
nano index.html
# 粘贴内容，Ctrl+O 保存，Ctrl+X 退出
```

2. **创建 docker-compose.homepage.yml**

```yaml
version: '3.8'

services:
  homepage:
    image: nginx:alpine
    container_name: homepage
    restart: always
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html:ro
```

3. **启动容器**

```bash
cd ~/homepage
docker-compose -f docker-compose.homepage.yml up -d
```

4. **测试访问**

```
http://你的VPS地址:8080/
```

5. **配置 new-api**
   - 首页内容填入：`https://api.demokt.me:8080/` 或你的实际地址

---

## 方案三：与 new-api 共用 Nginx（进阶）

如果你已有 Nginx 反向代理，可以添加一个 location：

```nginx
# 在现有 nginx 配置中添加
location /homepage/ {
    alias /path/to/homepage/;
    index index.html;
}
```

然后首页内容填入：`https://api.demokt.me/homepage/index.html`

---

## 常见问题

### Q: 页面显示空白？
检查：
1. URL 是否以 `https://` 开头
2. 是否存在跨域问题（CORS）
3. 浏览器控制台是否有错误

### Q: 样式不对？
new-api 使用 iframe 加载外部页面，确保：
1. 页面内使用完整的 HTML 结构
2. 没有 `X-Frame-Options` 限制

### Q: 如何使用自定义域名？
1. 添加 DNS A 记录指向 VPS
2. 配置 Nginx 反向代理
3. 使用 Let's Encrypt 申请免费 SSL 证书

---

## 更新页面

### GitHub Pages
直接在 GitHub 仓库编辑或重新上传 index.html

### VPS 部署
```bash
# 编辑文件
nano ~/homepage/index.html

# 无需重启容器，Nginx 会自动读取最新文件