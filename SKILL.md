---
name: blog-skill
description: |
  个人博客管理技能。支持文章、文章分类、Linux命令、命令分类的增删改查及文章发布。
  当用户说"写文章"、"发博客"、"往博客写篇 XXX"、"添加命令"、"添加分类" 时触发此 skill。
  通过博客 API Key 鉴权，无需 SSH 到服务器。
homepage: https://www.lilinsong.cn
metadata:
  openclaw:
    emoji: 📝
    requires:
      env:
        - BLOG_API_URL
        - BLOG_API_KEY
    primaryEnv: BLOG_API_URL
  security:
    credentials_usage: |
      Requires user-provisioned blog API Key.
      Key is sent as Bearer token in Authorization header to lilinsong.cn only.
      No credentials are logged or stored elsewhere.
    allowed_domains:
      - www.lilinsong.cn
      - lilinsong.cn
---

# 博客技能 · blog-skill

用于管理个人博客的 OpenClaw 技能，覆盖文章、文章分类、Linux命令、命令分类的完整 CRUD。

## 环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `BLOG_API_URL` | 博客 API 地址 | `https://www.lilinsong.cn/api` |
| `BLOG_API_KEY` | API Key（后台管理创建） | `bk_xxx...` |

## 通用说明

- 所有写操作需要在 `Authorization` 头携带 API Key：`Bearer {BLOG_API_KEY}`
- API Key 权限按资源+操作细分，格式为 `resource:action`
- 权限列表：

| 权限 | 说明 |
|------|------|
| `article:read` | 查看文章详情和列表 |
| `article:write` | 创建文章 |
| `article:update` | 修改文章 |
| `article:delete` | 删除文章 |
| `article:publish` | 发布/取消发布文章 |
| `category:read` | 查看分类列表 |
| `category:write` | 创建分类 |
| `category:update` | 修改分类 |
| `category:delete` | 删除分类 |
| `command:read` | 查看命令详情和列表 |
| `command:write` | 创建命令 |
| `command:update` | 修改命令 |
| `command:delete` | 删除命令 |

- GET 请求一般无需认证（公开接口），但回收站等接口需要对应 `read` 权限
- 建议创建 Key 时选择 `article:write,article:update,article:delete,article:publish,category:write,category:update,category:delete,command:write,command:update,command:delete` 全权限
- 所有请求和响应均为 JSON 格式

---

## 一、文章管理

### 1.1 获取文章列表

```
GET {BLOG_API_URL}/articles?page=1&limit=10&q=关键词
```

**查询参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `page` | number | 页码，默认 1 |
| `limit` | number | 每页条数，默认 10，最大 100 |
| `q` | string | 搜索关键词（标题/摘要） |
| `all` | string | `true` 显示草稿（需认证） |

**返回：** `{ articles: [...], pagination: { total, page, limit, totalPages } }`

### 1.2 获取文章详情

```
GET {BLOG_API_URL}/articles/:slug
```

**返回：** 完整文章对象，含 `content_md`、`content_html`、`categories`、`tags` 等

### 1.3 创建文章

```
POST {BLOG_API_URL}/articles
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `title` | string | ✅ | 文章标题 |
| `content_md` | string | ✅ | Markdown 正文 |
| `summary` | string | ❌ | 摘要，默认取正文前200字 |
| `cover_image` | string | ❌ | 封面图 URL |
| `categories` | string[] | ❌ | 分类名数组，自动查找或创建 |
| `tags` | string[] | ❌ | 标签名数组，自动查找或创建 |
| `is_published` | boolean | ❌ | 是否发布，默认 `true` |

**返回：** 201 Created，包含完整文章对象（含 `id`、`slug` 等）

需要 `article:write` 权限。

### 1.4 更新文章

```
PUT {BLOG_API_URL}/articles/:id
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

请求体字段同创建，所有字段可选。

需要 `article:update` 权限。

### 1.5 删除文章（软删除，移入回收站）

```
DELETE {BLOG_API_URL}/articles/:id
Authorization: Bearer {BLOG_API_KEY}
```

需要 `article:delete` 权限。

### 1.6 发布/取消发布

```
PATCH {BLOG_API_URL}/articles/:id/publish
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `is_published` | boolean | ✅ | `true` 发布，`false` 取消发布 |

需要 `article:publish` 权限。

### 1.7 置顶/取消置顶

```
PATCH {BLOG_API_URL}/articles/:id/pin
Authorization: Bearer {BLOG_API_KEY}
```

需要 `article:update` 权限。切换置顶状态。

---

## 二、文章分类管理

### 2.1 获取所有分类

```
GET {BLOG_API_URL}/categories
```

**返回：** `[{ id, name, slug, _count: { articles } }, ...]`

### 2.2 创建分类

```
POST {BLOG_API_URL}/categories
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 分类名称 |

需要 `category:write` 权限。

### 2.3 更新分类

```
PUT {BLOG_API_URL}/categories/:id
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 新分类名称 |

需要 `category:update` 权限。

### 2.4 删除分类

```
DELETE {BLOG_API_URL}/categories/:id
Authorization: Bearer {BLOG_API_KEY}
```

注意：分类下有文章时无法删除，需先移除文章关联。

需要 `category:delete` 权限。

---

## 三、Linux 命令管理

### 3.1 获取命令列表

```
GET {BLOG_API_URL}/commands?page=1&limit=20&q=关键词&categoryId=1
```

**查询参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `page` | number | 页码，默认 1 |
| `limit` | number | 每页条数，默认 20，最大 100 |
| `q` | string | 搜索关键词（命令名/描述） |
| `categoryId` | number | 按分类ID筛选 |

**返回：** `{ commands: [...], pagination: { total, page, limit, totalPages } }`

每个命令对象含 `categoryRel: { id, name, slug }` 分类信息。

### 3.2 获取命令详情

```
GET {BLOG_API_URL}/commands/:slug
```

**参数说明：**

- `:slug` 优先用命令的 `slug` 字段（从列表接口 3.1 获取）
- 也可传命令 `name`，但**含特殊字符的命令名必须 URL 编码**（如 `g++` → `g%2B%2B`，否则 `+` 会被解析为空格）
- **推荐做法**：先调用 3.1 列表接口拿到 `slug`，再用 `slug` 请求详情，避免特殊字符问题

**返回：** 完整命令对象，含 `content_html`、`categoryRel`、`relatedCommands`（相关命令列表）

`relatedCommands` 格式：`[{ id, name, slug, description }, ...]`，最多5个，优先手动关联，不足用同分类补充。

### 3.3 创建命令

```
POST {BLOG_API_URL}/commands
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 命令名称（如 `ls`） |
| `description` | string | ✅ | 一句话简介 |
| `categoryId` | number | ❌ | 所属分类ID |
| `content` | string | ❌ | Markdown 详细说明 |

需要 `command:write` 权限。

### 3.4 更新命令

```
PUT {BLOG_API_URL}/commands/:id
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

请求体字段同创建，所有字段可选。

需要 `command:update` 权限。

### 3.5 设置命令关联

```
PUT {BLOG_API_URL}/commands/:id/relations
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `relatedIds` | number[] | ✅ | 关联命令ID数组（全量替换） |

需要 `command:update` 权限。设置后会覆盖之前的关联。

### 3.6 删除命令（软删除）

```
DELETE {BLOG_API_URL}/commands/:id
Authorization: Bearer {BLOG_API_KEY}
```

需要 `command:delete` 权限。

---

## 四、命令分类管理

### 4.1 获取所有命令分类

```
GET {BLOG_API_URL}/command-categories
```

**返回：** `[{ id, name, slug, sort, count }, ...]`，`count` 为该分类下命令数量。

### 4.2 创建命令分类

```
POST {BLOG_API_URL}/command-categories
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 分类名称 |
| `sort` | number | ❌ | 排序权重，默认 0 |

需要 `command:write` 权限。

### 4.3 更新命令分类

```
PUT {BLOG_API_URL}/command-categories/:id
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ❌ | 新分类名称 |
| `sort` | number | ❌ | 排序权重 |

需要 `command:update` 权限。

### 4.4 删除命令分类

```
DELETE {BLOG_API_URL}/command-categories/:id
Authorization: Bearer {BLOG_API_KEY}
```

删除后该分类下的命令的 `categoryId` 会被设为 null。

需要 `command:delete` 权限。

---

## 使用示例

### 写一篇新文章

用户说："往博客写一篇关于 Docker 部署的文章"

1. 生成 Markdown 内容
2. 调用 `POST /api/articles` 创建文章
3. 告知用户文章已创建（含链接）

### 添加一个 Linux 命令

用户说："帮我添加一个 grep 命令的说明"

1. 先获取分类列表 `GET /api/command-categories` 找到合适分类
2. 调用 `POST /api/commands` 创建命令
3. 告知用户命令已添加

### 获取命令详情（含特殊字符的命令）

用户说："帮我查下 g++ 命令的说明"

1. 先调用 `GET /api/commands?q=g++` 查列表，找到命令的 `slug` 字段（`g++` 的 slug 是 `g--`）
2. 用 slug 调用 `GET /api/commands/g--` 获取详情
3. 告知用户命令详情

> **重要**：不要直接用命令名拼 URL（如 `/api/commands/g++`），因为 `+` 在 URL 中会被解析为空格导致查不到。必须用列表返回的 `slug`，或对命令名做 URL 编码（`g++` → `g%2B%2B`）。

### 创建命令分类

用户说："加一个网络工具分类"

1. 调用 `POST /api/command-categories` 创建分类
2. 告知用户分类已创建

### 更新已有文章

用户说："帮我更新那篇 Docker 文章，加一段关于网络的说明"

1. 先获取文章列表找到对应文章
2. 调用 `PUT /api/articles/:id` 更新内容
3. 告知用户更新完成

### 发布文章

用户说："把那篇草稿发布了"

1. 找到草稿文章
2. 调用 `PATCH /api/articles/:id/publish` 发布
3. 告知用户已发布

## 注意事项

- API Key 在博客后台管理「API Key」tab 中创建
- 创建 Key 后立即复制保存，关闭后无法恢复
- 权限按 `资源:操作` 格式细分，可按需勾选
- 文章 slug 由后端自动生成（标题 + 时间戳后4位）
- 命令 slug 由后端自动生成（命令名小写 + 特殊字符替换，如 `g++` 的 slug 是 `g--`）
- **获取命令详情时**：`:slug` 参数推荐用列表接口返回的 `slug` 字段；若直接用命令名且含特殊字符（如 `+`、`/`、空格），必须做 URL 编码（`encodeURIComponent`），否则会解析错误（例：`g++` 会被解析成空格）
- 删除操作均为软删除（移入回收站），可恢复
