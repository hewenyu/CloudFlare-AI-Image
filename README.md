# CloudFlare-AI-image

基于Cloudflare Worker的AI图片生成脚本，支持多种模型，兼容OpenAI API格式。

## 支持的模型

-  "DS-8-CF": "@cf/lykon/dreamshaper-8-lcm"
-  "SD-XL-Bash-CF": "@cf/stabilityai/stable-diffusion-xl-base-1.0"
-  "SD-XL-Lightning-CF": "@cf/bytedance/stable-diffusion-xl-lightning"
-  "FLUX.1-Schnell-CF": "@cf/black-forest-labs/flux-1-schnell"
-  "SF-Kolors": "Kwai-Kolors/Kolors"
 
五种可选文生图模型，默认SD-XL-Bash-CF，推荐FLUX.1-Schnell-CF效果最好，但有每日使用限制。

## 部署步骤

### 1. 准备工作

- Cloudflare账户
- 已接入Cloudflare的域名
- Node.js环境（用于wrangler部署）

### 2. 配置wrangler.jsonc

创建一个基本的wrangler.jsonc文件：

```json
{
  "name": "ai-img",
  "compatibility_date": "2025-04-10",
  "main": "text2img.js",
  "kv_namespaces": [
    {
      "binding": "IMAGE_KV",
      "id": "您的KV命名空间ID"
    }
  ],
  "ai": {
    "binding": "AI"
  }
}
```

### 3. 部署Worker

```bash
# 安装wrangler
npm install -g wrangler

# 登录Cloudflare
npx wrangler login

# 部署
npx wrangler deploy
```

### 4. 手动配置(推荐)

部署后，在Cloudflare控制台中完成以下配置：

1. **KV命名空间**:
   - 创建KV命名空间并命名为`IMAGE_KV`
   - 在Worker设置中绑定此KV命名空间

2. **环境变量**:
   - 添加环境变量`API_KEY`，设置为您的API密钥

3. **自定义域名**:
   - 在Worker的"触发器"选项卡中添加自定义域名
   - 例如：`img.yourdomain.com`

4. **DNS配置**:
   - 在DNS设置中添加CNAME记录，指向您的Worker地址
   - 名称：子域名（如img）
   - 目标：`<您的用户名>.workers.dev`
   - 代理状态：开启（橙色云朵）

## 使用说明

该API与OpenAI API格式兼容，可以在任何支持OpenAI的客户端中使用。

### API端点

- 获取模型列表：`GET https://您的域名/v1/models`
- 生成图像：`POST https://您的域名/v1/chat/completions`

### 生成图像示例

```json
{
  "model": "FLUX.1-Schnell-CF",
  "messages": [
    {
      "role": "user",
      "content": "生成一张风景画"
    }
  ]
}
```

请求头：
```
Authorization: Bearer 您的API密钥
Content-Type: application/json
```

### 提示词优化

- 默认启用中英文提示词翻译及优化
- 添加 `---ntl` 后缀可以关闭翻译（例如：`生成一只猫 ---ntl`）

### 图像到文本功能

支持图像作为输入，使用LLaVA模型进行图像描述生成，然后与文本提示组合生成图像。

## 高级配置

可在`text2img.js`文件中调整以下配置：

- `CF_IS_TRANSLATE`: 是否启用提示词翻译
- `CF_TRANSLATE_MODEL`: 使用的翻译模型
- `CF_IMG2TEXT_MODEL`: 图像描述模型
- `FLUX_NUM_STEPS`: Flux模型参数
- `IMAGE_EXPIRATION`: 图像在KV中的过期时间

## 注意事项

- 如需使用"SF-Kolors"模型，请在代码中配置SF_TOKEN（硅基流动平台的API Token）
- 部分模型可能有使用限制，请注意查看Cloudflare Workers AI配额
- 推荐使用手动配置方式，避免自动DNS配置可能带来的冲突