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

### 2. 配置Cloudflare

1. **创建KV命名空间**：
   - 在Cloudflare控制台创建KV命名空间并命名为`IMAGE_KV`
   - 记录生成的命名空间ID

2. **创建wrangler.jsonc**：
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
     },
     "vars": {
       "API_KEY": "您的API密钥"
     },
     "routes": [
       {
         "pattern": "您的域名",
         "custom_domain": true
       }
     ]
   }
   ```

3. **设置DNS**：
   - 在Cloudflare DNS设置中添加CNAME记录
   - 名称：您的子域名（如img）
   - 目标：<您的用户名>.workers.dev
   - 代理状态：开启（橙色云朵）

### 3. 部署Worker

```bash
# 安装wrangler
npm install -g wrangler

# 登录Cloudflare
npx wrangler login

# 部署
npx wrangler deploy
```

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

- SF_TOKEN为硅基流动平台的API Token，需要提前申请，不使用可不填写
- 部分模型可能有使用限制，请注意查看Cloudflare Workers AI配额

![CloudFlare配置](https://raw.githubusercontent.com/justlovemaki/CloudFlare-AI-Image/refs/heads/main/example-1.png)
