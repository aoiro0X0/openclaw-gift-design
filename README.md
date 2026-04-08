# OpenClaw Gift Design

`Openclaw-gift-design skill：运营文档一键拆解并生成飞书设计侧文档底稿 + Nano Banana 图像生成与编辑`

这是一个面向直播礼物设计场景的 OpenClaw skill。它把运营文档里的礼物信息拆成结构化价效规则，并生成给设计师继续完善的飞书设计文档底稿；同时提供基于 Nano Banana 的礼物图像生成、连续编辑、抠图和换背景能力。

## 主要能力

### 1. 运营文档 -> 飞书设计文档底稿
- 解析礼物名称、价格、主题描述
- 生成设计工作表
- 自动补齐屏占比、时长、镜头数等规则字段
- 通过 `lark-cli` 创建飞书文档，输出文档链接

设计工作表当前包含这些固定行：
- 屏占比
- 时长
- 镜头数
- 关键帧设计
- 直播间背景展示
- ICON预览

### 2. Nano Banana 图像生成与编辑
- `txt2img`：从文本生成礼物方案图
- `img2img`：基于已有图片继续修改
- `inpaint`：按 mask 局部编辑
- `background-replace`：保留主体、替换背景

### 3. 连续编辑上下文承接
- 回复某张图时，优先编辑被回复的那张图
- 用户说“上一张 / 刚刚那张 / 这张继续改 / 去掉背景”时，默认承接当前会话最近一次成功出图
- 如果没有可继续编辑的图片，会返回追问，而不是误走重新生图

## 仓库结构

```text
.
├─ SKILL.md
├─ references/
│  ├─ http-api.md
│  ├─ params.md
│  ├─ workflows.md
│  └─ zenmux-nano-banana.md
├─ scripts/
│  ├─ banana-image.mjs
│  ├─ feishu-bridge.mjs
│  ├─ image-context.mjs
│  ├─ intent-analyzer.mjs
│  └─ model-router.mjs
└─ tests/
```

## 依赖

- Node.js 22+
- `ZENMUX_API_KEY` 或 `GEMINI_API_KEY`
- 本地可用的 `lark-cli`

可选环境变量：
- `ZENMUX_BASE_URL`
- `GOOGLE_GEMINI_BASE_URL`
- `OPENCLAW_BANANA_MODEL`
- `ZENMUX_IMAGE_MODEL`
- `OPENCLAW_MEDIA_DIR`
- `BANANA_THREAD_STATE_PATH`

## 安装

### 作为 OpenClaw skill 安装

在支持 skill 安装的环境里，从这个 GitHub 仓库安装即可：

```text
https://github.com/aoiro0X0/openclaw-gift-design
```

### 本地开发

```powershell
git clone https://github.com/aoiro0X0/openclaw-gift-design.git
cd openclaw-gift-design
```

## 常用命令

### 1. 生成礼物图

```powershell
node .\scripts\banana-image.mjs `
  --task "Generate a fluffy plush bouquet gift image for live streaming" `
  --model-mode auto
```

### 2. 基于上一张图继续编辑

```powershell
node .\scripts\banana-image.mjs `
  --task "去掉背景，换成纯绿底" `
  --thread-id "feishu-thread-123" `
  --continue-last-image
```

### 3. 编辑指定图片

```powershell
node .\scripts\banana-image.mjs `
  --task "保留主体，改成蓝色背景" `
  --input-image-path ".\output\banana\txt2img-1.png"
```

### 4. 用被回复图片继续编辑

```powershell
node .\scripts\banana-image.mjs `
  --task "去掉背景，换成纯绿底" `
  --reply-target-image-path "C:\path\to\reply-target.png" `
  --thread-id "feishu-thread-123"
```

### 5. 创建飞书设计文档

```powershell
node .\scripts\banana-image.mjs `
  --create-design-doc `
  --title "冬日一起毛绒绒" `
  --ops-doc-link "https://bytedance.larkoffice.com/docx/xxx" `
  --theme-summary "延续冬日主题，通过不同物象和风格验证季节主题的新鲜感。" `
  --gifts-json "@.\gifts.json"
```

`gifts.json` 示例：

```json
[
  {
    "name": "毛绒花束",
    "price_str": "99钻",
    "subject_description": "毛茸茸花束，温暖、陪伴、治愈"
  },
  {
    "name": "冬日暖阳",
    "price_str": "8999钻",
    "subject_description": "暖冬城市街景，热闹、温暖、节日感"
  }
]
```

## 规则说明

### 屏占比
设计工作表中的屏占比按价位区间映射，来源于项目内的价效规范表。

### 镜头数
镜头数在设计工作表里按“上限”展示，而不是写死分镜数：
- `<99钻`：`0`
- `>=99钻 且 <1000钻`：`1`
- `>=1000钻 且 <6000钻`：`≤2`
- `>=6000钻 且 <10000钻`：`≤3`
- `>=10000钻`：`≤4`

如果 `6000–10000钻` 档命中明显的高速/竞速/追逐/速度感主题词，可自动放宽到 `≤4`。

## 输出格式

图像生成成功时，脚本会返回：
- `output_files`
- `media.mediaUrls`
- `mediaUrls`
- `mediaUrl`

连续编辑但缺少目标图片时，会返回：

```json
{
  "status": "follow_up_required",
  "follow_up_question": "当前会话里没有可继续编辑的图片，请回复某张图或重新发送图片。"
}
```

创建飞书设计文档成功时，会返回：

```json
{
  "ok": true,
  "url": "https://...",
  "doc_id": "..."
}
```

## 测试

```powershell
node --test .\tests\banana-image.test.mjs
node --test .\tests\banana-image-design-doc.test.mjs
node --test .\tests\image-context.test.mjs
node --test .\tests\intent-analyzer-layout.test.mjs
node --test .\tests\feishu-bridge.test.mjs
```

## 相关文档

- [SKILL.md](./SKILL.md)
- [references/params.md](./references/params.md)
- [references/workflows.md](./references/workflows.md)
- [references/http-api.md](./references/http-api.md)
- [references/zenmux-nano-banana.md](./references/zenmux-nano-banana.md)
