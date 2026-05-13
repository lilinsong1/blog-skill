---
name: blog-skill
description: |
  个人博客文章管理技能。支持创建、更新、发布文章。
  当用户说"写文章"、"发博客"、"往博客写篇 XXX" 时触发此 skill。
  通过博客 API Key 鉴权，无需 SSH 到服务器。
homepage: https://example.com
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
      - your-blog-domain.com
---

# 博客技能 · blog-skill

用于管理个人博客文章的 OpenClaw 技能。

## 环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `BLOG_API_URL` | 博客 API 地址 | `https://your-blog.com/api` |
| `BLOG_API_KEY` | API Key（后台管理创建） | `bk_xxx...` |

## API 端点

### 创建文章

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

### 更新文章

```
PUT {BLOG_API_URL}/articles/:id
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

请求体字段同创建，所有字段可选。需要 `update` 权限。

### 删除文章

```
DELETE {BLOG_API_URL}/articles/:id
Authorization: Bearer {BLOG_API_KEY}
```

需要 `delete` 权限。

### 发布/取消发布

```
PATCH {BLOG_API_URL}/articles/:id/publish
Authorization: Bearer {BLOG_API_KEY}
Content-Type: application/json
```

**请求体：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `is_published` | boolean | ✅ | `true` 发布，`false` 取消发布 |

需要 `publish` 权限。

## 使用示例

### 写一篇新文章

用户说："往博客写一篇关于 Docker 部署的文章"

小爱应：
1. 生成 Markdown 内容
2. 调用 `POST /api/articles` 创建文章
3. 告知用户文章已创建（含链接）

### 更新已有文章

用户说："帮我更新那篇 Docker 文章，加一段关于网络的说明"

小爱应：
1. 先获取文章列表找到对应文章
2. 调用 `PUT /api/articles/:id` 更新内容
3. 告知用户更新完成

### 发布文章

用户说："把那篇草稿发布了"

小爱应：
1. 找到草稿文章
2. 调用 `PATCH /api/articles/:id/publish` 发布
3. 告知用户已发布

## 注意事项

- API Key 在博客后台管理「API Key」tab 中创建
- 创建 Key 后立即复制保存，关闭后无法恢复
- 默认使用 `write` + `publish` 权限的 Key
- 文章 slug 由后端自动生成（标题 + 时间戳后4位）