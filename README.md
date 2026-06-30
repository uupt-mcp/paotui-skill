# UU跑腿同城配送服务 Skill

[![Version](https://img.shields.io/badge/version-1.0.6-blue.svg)](./SKILL.md)

Skill 让 AI 助手学会新技能。通过安装本 Skill，AI 助手可以获得同城即时配送和现场帮忙服务的专业能力，在对话中自动识别用户的需求并调用对应的接口来完成任务。

## 安装方式

### OpenClaw 用户：从 ClawHub 安装（推荐）

在 ClawHub 应用市场一键安装：

```bash
clawhub install uupt-delivery
```

> 提示：如果未安装 ClawHub CLI，可用 `npx clawhub@latest install uupt-delivery` 直接运行。

也可以访问 [ClawHub](https://clawhub.ai/) 网站搜索 `uupt-delivery`，下载 ZIP 包后解压到 Skills 目录。

### 从 GitHub 安装

从 [GitHub Releases](https://github.com/uupt-mcp/uupt-delivery-skill/releases) 下载最新版本的 `uupt-delivery.zip`，解压到 Agent 的 Skill 目录后安装依赖。

以 **Qoder** 为例（Skill 目录为 `~/.qoder/skills/`）：

```bash
# 下载最新版本
wget https://github.com/uupt-mcp/uupt-delivery-skill/releases/latest/download/uupt-delivery.zip

# 解压到 Skill 目录
unzip uupt-delivery.zip -d ~/.qoder/skills/uupt-delivery

# 进入目录安装依赖
cd ~/.qoder/skills/uupt-delivery
npm install

# 或安装 Python 依赖
pip install -r requirements.txt
```

> 不同 Agent 的 Skill 目录位置不同，请根据实际使用的 Agent 调整路径。例如 **Claude Code** 的 Skill 目录为 `~/.claude/skills/`。

安装完成后，首次使用会自动引导注册，详见 [首次使用](#首次使用)。

## 核心能力

| 能力 | 说明 | 适用场景 |
|------|------|---------|
| 手机号注册 | 首次使用时通过短信验证自动获取授权 | 首次使用，自动触发 |
| 订单询价 | 计算配送费用或帮忙服务费用 | 用户想知道价格 |
| 创建配送订单 | 发起跑腿配送订单（从A地到B地） | 用户需要同城配送物品 |
| 创建帮忙订单 | 发起帮忙服务订单（在指定地点获得现场协助） | 用户需要现场帮忙（搬东西、取号、排队等） |
| 查询订单 | 获取订单的当前状态和详细信息 | 用户想了解订单进度 |
| 取消订单 | 取消未完成的订单 | 用户不想继续服务 |
| 跑男追踪 | 实时查询跑男的位置和状态 | 用户想知道跑男在哪 |

## 运行环境

本 Skill 同时提供 **Node.js** 和 **Python** 两种版本，可根据你的环境选择：

| 环境 | 依赖安装 | 脚本位置 |
|------|---------|---------|
| Node.js | `npm install` | `scripts/*.js` 和 `index.js` |
| Python | `pip install -r requirements.txt` | `uupt_delivery.py` |

## 首次使用

本 Skill 支持两种认证方式：

### 方式一：快速体验模式（推荐）

**无需手动配置任何凭证。** 首次运行任何功能时会自动检测是否需要注册，如果未注册会输出 `[REGISTRATION_REQUIRED]` 提示，AI 助手会自动引导你通过手机号短信验证完成注册。

注册完成后 openId 自动保存到本地配置文件，后续使用无需重复操作。

**注册流程：**
1. 执行任意功能脚本，检测到未注册时输出 `[REGISTRATION_REQUIRED]`
2. 输入手机号，系统发送短信验证码
3. 如遇 `[IMAGE_CAPTCHA_REQUIRED]`，需识别图片验证码后重试
4. 输入短信验证码完成授权
5. 注册成功后自动继续执行原功能

### 方式二：开发者模式

如果你已拥有 UU跑腿开放平台的凭证（appId、appSecret、openId），可通过以下方式配置：

**环境变量方式：**
```bash
export UUPT_APP_ID=你的appId
export UUPT_APP_SECRET=你的appSecret
export UUPT_OPEN_ID=你的openId
```

**配置文件方式：**
创建 `config.json` 文件：
```json
{
  "appId": "你的appId",
  "appSecret": "你的appSecret",
  "openId": "你的openId"
}
```

## 快速开始

### 安装依赖

```bash
# Node.js
npm install

# Python
pip install -r requirements.txt
```

### 注册（首次使用）

```bash
# Step 1: 发送验证码
node scripts/register.js --mobile="13800138000"
# 或 Python: python uupt_delivery.py register --mobile="13800138000"

# Step 2: 输入验证码完成授权
node scripts/register.js --mobile="13800138000" --smsCode="123456"
# 或 Python: python uupt_delivery.py register --mobile="13800138000" --sms-code="123456"
```

### 使用示例

```bash
# === 跑腿配送 ===
# 询价
node scripts/order-price.js --fromAddress="郑州市金水区农业路经三路交叉口" --toAddress="郑州市二七区德化街100号"

# 创建订单
node scripts/create-order.js --priceToken="xxx" --receiverPhone="13800138000"

# === 帮忙服务 ===
# 询价
node scripts/order-price.js --fromAddress="郑州人民医院" --orderType="help"

# 创建帮忙订单（必须带 --note 参数描述帮忙内容）
node scripts/create-order.js --priceToken="xxx" --receiverPhone="13800138000" --note="帮我在郑州人民医院挂个号"

# === 通用操作 ===
# 查询订单
node scripts/order-detail.js --orderCode="UU123456789"

# 取消订单
node scripts/cancel-order.js --orderCode="UU123456789" --reason="用户改变主意"

# 跑男追踪
node scripts/driver-track.js --orderCode="UU123456789"
```

## 配置管理

配置分为两层，无需手动编辑：

| 配置文件 | 内容 | 说明 |
|---------|------|------|
| `defaults.json` | appId、appSecret、apiUrl | 内置应用凭证，随 Skill 分发，**请勿修改** |
| `config.json` | openId（或完整凭证） | 用户级配置，注册成功后自动生成 |

配置优先级（从高到低）：环境变量 > config.json > defaults.json

### 环境变量

| 变量名 | 说明 | 必填 |
|--------|------|------|
| `UUPT_APP_ID` | 应用 ID | 开发者模式必填 |
| `UUPT_APP_SECRET` | 应用密钥 | 开发者模式必填 |
| `UUPT_OPEN_ID` | 用户唯一标识 | 是 |
| `UUPT_API_URL` | API 地址 | 否，默认生产环境 |

### API 环境

| 环境 | URL |
|------|-----|
| 生产环境 | `https://api-open.uupt.com/openapi/v3/` |
| 测试环境 | `http://api-open.test.uupt.com/openapi/v3/` |

## 注意事项

- **首次使用**：需通过手机号验证获取授权，之后无需重复操作
- **图片验证码**：如发送短信时返回 `[IMAGE_CAPTCHA_REQUIRED]`，需展示 base64 图片让用户识别后重试
- **注册重试**：授权失败时自动重试，最多 3 次（无需重新输入手机号）
- **询价有效期**：priceToken 有时效性，建议获取后尽快创建订单
- **地址完整性**：地址信息越完整，配送越准确
- **城市默认值**：如未指定城市，默认使用"郑州市"
- **价格单位**：API 返回的价格单位是分，展示时需除以 100 转换为元
- **余额不足**：订单返回 `[PAYMENT_REQUIRED]` 时，需以附件形式发送微信支付二维码图片，同时提供在线链接作为兜底方案
- **配置文件**：`defaults.json` 为内置凭证，请勿修改或删除
- **帮忙订单**：帮忙订单的 fromAddress 和 toAddress 相同，必须传递 `--note` 参数描述具体帮忙内容

## 相关链接

- [ClawHub 应用市场](https://clawhub.ai/uupt-mcp/skills/uupt-delivery)
- [GitHub 仓库](https://github.com/uupt-mcp/uupt-delivery-skill)
- [UU跑腿开放平台](https://open.uupt.com/#/development/agentSkill/quickStart)
