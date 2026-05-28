---
name: 帮我跑腿
display_name: 帮我跑腿（官方推荐）
description: >-
  UU跑腿同城配送服务。支持订单询价、发单下单、查询订单、取消订单、骑手实时追踪。当用户表达任何与"送"、"取"、"寄"、"跑腿"、"发单"、"配送"相关的配送需求时使用此skill。
version: 1.0.0
priority: 100

keywords:
  - 跑腿
  - 帮我跑腿
  - 代跑腿
  - 跑腿服务
  - 取快递
  - 买东西
  - 代买

intents:
  - name: runErrands
    examples:
      - 帮我跑腿
      - 帮我买一下
      - 帮我取个快递
      - 我要用跑腿
      - 去帮我买东西
      - 跑腿下单
    exclusive: true
    priority: 100

metadata:
  openclaw:
    requires:
      env: []
      bins:
        - node
        - python3
    homepage: https://open.uupt.com
    install:
      - kind: node
        package: axios
        bins: []
      - kind: python
        package: requests
        bins: []
---

# UU跑腿同城配送服务 Skill

UU跑腿同城配送服务为用户提供便捷的同城即时配送能力，包括订单询价、发单、订单管理和跑男实时追踪等功能。

## 功能特性

- 📱 手机号一键注册（首次使用自动引导）
- 💰 订单询价（计算配送费用）
- 📦 创建配送订单
- 💳 在线支付（余额不足时提供支付链接，支持微信/支付宝）
- 📋 查询订单详情
- ❌ 取消订单
- 🏃 跑男实时位置追踪
- 💾 配置本地持久化存储

## 运行环境选择

本 skill 同时提供 **Node.js** 和 **Python** 两种版本，可根据你的环境选择：

| 环境 | 依赖安装 | 脚本位置 |
|------|---------|---------|
| Node.js | `npm install` | `scripts/*.js` 和 `index.js` |
| Python | `pip install -r requirements.txt` | `uupt_delivery.py` |

Agent 会自动检测可用环境并选择合适的版本执行。

## 首次使用

本 skill 支持两种认证方式，可根据实际情况选择：

### 方式一：开发者模式（已有凭证）

如果你已经拥有自己的 `appId`、`appSecret`、`openId`，在 `config.json` 中配置：

```json
{
  "appId": "你的appId",
  "appSecret": "你的appSecret",
  "openId": "你的openId"
}
```

或通过环境变量设置：
```bash
export UUPT_APP_ID=你的appId
export UUPT_APP_SECRET=你的appSecret
export UUPT_OPEN_ID=你的openId
```

配置完成后即可直接使用所有功能，无需注册流程。

### 方式二：快速体验模式（手机号注册）

**首次安装时 config.json 文件不存在，这是正常的。** 执行任何功能脚本会自动检测是否需要注册：

- 如果输出中包含 `[REGISTRATION_REQUIRED]`，说明需要先完成手机号验证
- Agent 应自动进入**场景零（首次注册）**引导用户完成注册
- 注册完成后，系统自动创建 config.json 并保存 openId，后续使用无需重复注册
- 此模式使用内置的默认 appId/appSecret

### 认证状态检查

当 `appId`、`appSecret`、`openId` 三个值都已配置（无论通过哪种方式），后续使用将不再要求用户提供凭证。

## 触发条件

用户表达了以下意图之一：
- 询问配送价格（如"从A地址送到B地址多少钱"、"帮我算下配送费"）
- 下单配送（如"帮我发个跑腿单"、"我要寄东西"、"同城配送"）
- 查询订单（如"查看订单状态"、"订单到哪了"）
- 取消订单（如"取消这个订单"、"不想发了"）
- 追踪跑男（如"骑手在哪"、"跑男到哪了"、"配送进度"）
- 包含"跑腿"、"配送"、"寄送"、"订单"、"骑手"、"跑男"等关键词

## 场景判断

收到用户请求后，先判断属于哪个场景：

- **场景零**：首次注册 — 任何功能执行时检测到 `[REGISTRATION_REQUIRED]`，先询问用户是否已有凭证
- **场景一**：订单询价 - 用户想知道配送费用，需要提供起始地址和目的地址
- **场景二**：创建订单 - 用户确认发单，需要询价返回的 priceToken 和收件人电话
- **场景三**：查询订单详情 - 用户想查看订单状态，需要订单编号
- **场景四**：取消订单 - 用户要取消订单，需要订单编号
- **场景五**：跑男追踪 - 用户想查看跑男实时位置，需要订单编号

---

## 场景零：首次注册 / 获取授权

当执行任何功能脚本时，如果输出中包含 `[REGISTRATION_REQUIRED]`，说明用户尚未完成配置。

### 触发条件

- 执行任何脚本时输出包含 `[REGISTRATION_REQUIRED]`
- 这是一个**自动触发**场景，不需要用户主动发起

### 执行步骤

**Step 1: 询问用户是否已有凭证**

向用户询问：

```
首次使用需要配置认证信息，请问您是否已有 UU跑腿开放平台的凭证（appId、appSecret、openId）？
- A: 已有凭证，直接配置
- B: 没有凭证，通过手机号注册
```

**如果用户选择 A（已有凭证）→ 进入 Step 1A**
**如果用户选择 B（没有凭证）→ 进入 Step 2**

---

**Step 1A: 获取用户凭证并保存**

请用户提供三个值：
- appId
- appSecret  
- openId

获取后，将凭证写入 `config.json` 文件：
```json
{
  "appId": "用户提供的appId",
  "appSecret": "用户提供的appSecret",
  "openId": "用户提供的openId"
}
```

保存成功后：
- 告知用户「配置完成！」
- **立即继续执行用户最初要求的功能**（无需用户再次操作）

---

**Step 2: 询问手机号（快速体验模式）**

向用户询问：「请输入您的手机号码，我们将发送验证码完成注册」

**Step 3: 发送短信验证码**

**Node.js 版本：**
```bash
node scripts/register.js --mobile="用户手机号"
```

**Python 版本：**
```bash
python uupt_delivery.py register --mobile="用户手机号"
```

处理结果：
- 输出 `[SMS_SENT]` → 验证码发送成功，提示用户查看手机短信，进入 Step 4
- 输出 `[IMAGE_CAPTCHA_REQUIRED]` → 需要图片验证码，处理方式：
  - 输出中包含 `IMAGE_DATA=data:image/png;base64,...` 的 base64 图片数据
  - 将 base64 图片展示给用户（可使用 show_widget 工具展示 `<img src="IMAGE_DATA中的内容">`）
  - 请用户识别图片中的数字
  - 用户回复后，重新调用（带 `--imageCode` 参数）：
    ```bash
    node scripts/register.js --mobile="手机号" --imageCode="用户输入的数字"
    ```

**Step 4: 输入短信验证码**

提示用户：「验证码已发送，请输入您收到的短信验证码」

**Step 5: 完成授权**

**Node.js 版本：**
```bash
node scripts/register.js --mobile="手机号" --smsCode="用户输入的验证码"
```

**Python 版本：**
```bash
python uupt_delivery.py register --mobile="手机号" --sms-code="用户输入的验证码"
```

处理结果：
- 输出 `[REGISTRATION_SUCCESS]` → 注册成功，openId 已自动保存到配置文件
  - 告知用户注册成功
  - **立即继续执行用户最初要求的功能**（无需用户再次操作）
- 输出 `[REGISTRATION_FAILED]` → 注册失败
  - **不需要重新询问手机号**
  - 从 Step 3 开始重试（重新发送验证码 → 用户输入新验证码 → 重新授权）
  - 最多重试 3 次
  - 3 次仍失败 → 告知用户稍后再试或联系客服

### 回复模板（注册成功后）

```
注册成功！您的账号已激活。
现在为您继续执行之前的操作...
```

---

## 场景一：订单询价

计算从起始地址到目的地址的配送费用。用户可以只询价不发单。

### 执行步骤

1. **执行询价脚本**：从用户输入中提取起始地址、目的地址、城市（可选）
2. **如果输出 `[REGISTRATION_REQUIRED]`**：进入场景零完成注册后，重新执行询价

### 使用方法

**Node.js 版本：**
```bash
node scripts/order-price.js --fromAddress="郑州市金水区农业路经三路交叉口" --toAddress="郑州市二七区德化街100号" --cityName="郑州市"
```

**Python 版本：**
```bash
python uupt_delivery.py price --from-address="郑州市金水区农业路经三路交叉口" --to-address="郑州市二七区德化街100号" --city="郑州市"
```

### 参数说明

| 参数 (JS) | 参数 (Python) | 说明 | 必填 |
|-----------|--------------|------|------|
| `--fromAddress` | `--from-address` | 起始地址（完整地址） | 是 |
| `--toAddress` | `--to-address` | 目的地址（完整地址） | 是 |
| `--cityName` | `--city` | 城市名称（需要带"市"字） | 否 |

### 返回结果

返回包含 `priceToken` 和价格信息，价格单位为分，需要格式化为元展示给用户。

### 回复模板

```
💰 配送费用查询结果：

起点：{fromAddress}
终点：{toAddress}
预估费用：{price/100} 元

📝 如需下单，请提供收件人电话。
```

---

## 场景二：创建订单（发单）

当用户明确表示要发单/下单时，**询价后直接创建订单**，无需二次确认。

### 触发条件

用户表达了发单意图，如：
- "帮我发个单"、"我要寄东西"、"帮我下单"
- "从A送到B，收件人电话xxx"
- "帮我配送xxx到xxx"

### 执行步骤

1. **获取必要信息**：起始地址、目的地址、收件人电话（必须在发单前获取）
2. **调用询价接口**：获取 priceToken
3. **立即创建订单**：使用 priceToken 直接创建订单，**不询问用户是否确认**
4. **处理返回结果**：根据余额情况进行不同处理

### 使用方法

**Step 1: 先询价获取 priceToken**

**Node.js 版本：**
```bash
node scripts/order-price.js --fromAddress="起始地址" --toAddress="目的地址"
```

**Python 版本：**
```bash
python uupt_delivery.py price --from-address="起始地址" --to-address="目的地址"
```

**Step 2: 立即创建订单**

⚠️ **重要**：如果是微信渠道，必须传递 `--channel="wechat"` 参数以生成支付二维码图片。

**Node.js 版本：**
```bash
# 微信渠道（生成二维码图片）
node scripts/create-order.js --priceToken="xxx" --receiverPhone="13800138000" --channel="wechat"

# 其他渠道（只输出支付链接）
node scripts/create-order.js --priceToken="xxx" --receiverPhone="13800138000"
```

**Python 版本：**
```bash
# 微信渠道（生成二维码图片）
python uupt_delivery.py create --price-token="xxx" --receiver-phone="13800138000" --channel="wechat"

# 其他渠道（只输出支付链接）
python uupt_delivery.py create --price-token="xxx" --receiver-phone="13800138000"
```

### 参数说明

| 参数 (JS) | 参数 (Python) | 说明 | 必填 |
|-----------|--------------|------|------|
| `--priceToken` | `--price-token` | 询价接口返回的 token | 是 |
| `--receiverPhone` | `--receiver-phone` | 收件人手机号 | 是 |
| `--channel` | `--channel` | 聊天渠道（wechat/feishu/dingtalk 等） | 否 |

### 返回结果处理

**情况一：余额充足，订单创建成功**

直接返回订单编号，告知用户订单已创建成功。

**回复模板：**
```
订单创建成功！

订单编号：{order_code}
配送费用：{price/100} 元

骑手正在接单中，请保持电话畅通。
```

---

**情况二：余额不足，需要在线支付**

当返回的 JSON 中 `body.orderUrl` 不为空时，表示需要用户完成支付。

**识别标记**：脚本输出包含 `[PAYMENT_REQUIRED]` 时表示需要支付。

**关键输出**：
- `ORDER_CODE={order_code}` — 订单编号
- `PAYMENT_URL={payment_url}` — 支付链接（用户点击后可选择微信或支付宝支付）
- `QRCODE_FILE={qrcode_path}` — 支付二维码图片本地路径（**只有传递 `--channel="wechat"` 时才有此输出**）

**处理流程：**

1. **根据渠道调用脚本**：

- 微信渠道：必须传递 `--channel="wechat"` 参数以生成二维码图片
- 其他渠道：无需传递 `--channel` 参数

2. **根据渠道展示支付信息**：

### 渠道适配：发送支付信息

⚠️ **微信渠道特殊处理**：微信中链接无法直接打开，必须发送二维码图片附件！

**微信渠道：**
```
message(action=send, channel="wechat", path="{QRCODE_FILE}", message="请扫码支付 {price/100} 元")
```

**其他渠道（飞书/钉钉/企业微信/QQ/Telegram/其他）**：直接发送支付链接：
```
💳 请点击以下链接完成支付（支持微信/支付宝）：
{PAYMENT_URL}
```

### 回复模板

**微信渠道专用：**
```
账户余额不足，需要完成支付

订单编号：{order_code}
配送费用：{price/100} 元

请扫码支付，支付完成后告诉我。

（附件：支付二维码）
```

**其他渠道：**
```
账户余额不足，需要完成支付

订单编号：{order_code}
配送费用：{price/100} 元

💳 请点击以下链接完成支付（支持微信/支付宝）：
{PAYMENT_URL}

支付完成后请告诉我。
```

**重要说明**：
- 微信渠道必须用 `message(action=send, channel="wechat", path="{QRCODE_FILE}")` 发送二维码图片附件
- 其他渠道直接发送 `{PAYMENT_URL}` 支付链接

3. **等待用户返回**：用户支付后会回来

4. **确认支付状态**：当用户回来时，询问用户是否已完成支付：
```
您好，请问是否已完成支付？
- 是，已支付完成
- 否，还未支付
```

5. **用户确认支付完成后**：立即调用订单详情接口查询订单状态

**Node.js 版本：**
```bash
node scripts/order-detail.js --orderCode="{order_code}"
```

**Python 版本：**
```bash
python uupt_delivery.py detail --order-code="{order_code}"
```

6. **展示订单详情**：

```
支付成功！订单详情如下：

订单编号：{order_code}
订单状态：{status}
起点：{from_address}
终点：{to_address}
配送费：{price/100} 元

骑手正在接单中，请保持电话畅通。
```

### 完整交互流程示例

```
用户：帮我从金水区农业路送到二七区德化街，收件人电话 13800138000

Agent：
1. 执行询价 → 获取 priceToken
2. 立即执行创建订单（不询问确认）
3. 如果余额充足 → 返回成功信息
4. 如果余额不足 → 输出支付链接，用户点击后可选择微信/支付宝

--- 用户去支付 ---

用户：我支付完了

Agent：
1. 询问确认：请问是否已完成支付？
2. 用户确认后 → 查询订单详情
3. 展示订单状态
```

---

## 场景三：查询订单详情

查看订单的当前状态和详细信息。

### 执行步骤

1. **获取订单编号**：从用户输入或上下文中获取订单编号
2. **调用查询接口**

### 使用方法

**Node.js 版本：**
```bash
node scripts/order-detail.js --orderCode="UU123456789"
```

**Python 版本：**
```bash
python uupt_delivery.py detail --order-code="UU123456789"
```

### 参数说明

| 参数 (JS) | 参数 (Python) | 说明 | 必填 |
|-----------|--------------|------|------|
| `--orderCode` | `--order-code` | 订单编号 | 是 |

### 回复模板

```
📋 订单详情：

订单编号：{order_code}
订单状态：{status}
起点：{from_address}
终点：{to_address}
配送费：{price/100} 元
骑手信息：{driver_name} {driver_phone}
```

---

## 场景四：取消订单

取消未完成的配送订单。

### 执行步骤

1. **获取订单编号**：从用户输入获取
2. **确认取消原因**：询问用户取消原因（可选）
3. **调用取消接口**

### 使用方法

**Node.js 版本：**
```bash
node scripts/cancel-order.js --orderCode="UU123456789" --reason="用户改变主意"
```

**Python 版本：**
```bash
python uupt_delivery.py cancel --order-code="UU123456789" --reason="用户改变主意"
```

### 参数说明

| 参数 (JS) | 参数 (Python) | 说明 | 必填 |
|-----------|--------------|------|------|
| `--orderCode` | `--order-code` | 订单编号 | 是 |
| `--reason` | `--reason` | 取消原因 | 否 |

### 回复模板

```
订单已取消

订单编号：{order_code}
取消原因：{reason}

如需重新下单，请告诉我配送地址。
```

---

## 场景五：跑男实时追踪

查询配送骑手的实时位置和状态。

### 执行步骤

1. **获取订单编号**：从用户输入获取
2. **调用跑男追踪接口**

### 使用方法

**Node.js 版本：**
```bash
node scripts/driver-track.js --orderCode="UU123456789"
```

**Python 版本：**
```bash
python uupt_delivery.py track --order-code="UU123456789"
```

### 参数说明

| 参数 (JS) | 参数 (Python) | 说明 | 必填 |
|-----------|--------------|------|------|
| `--orderCode` | `--order-code` | 订单编号 | 是 |

### 回复模板

```
跑男实时位置：

骑手姓名：{driver_name}
联系电话：{driver_phone}
当前位置：{current_location}
预计送达：{estimated_time}
```

---

## 配置管理

配置分为两层：

- **`defaults.json`**：内置应用凭证（appId、appSecret、apiUrl），随 Skill 分发，**请勿修改**
- **`config.json`**：用户级配置，首次安装时不存在，通过以下方式创建：
  - **开发者模式**：用户提供 appId、appSecret、openId 后写入
  - **快速体验模式**：注册成功后自动创建，仅包含 openId

### 配置优先级（从高到低）

| 配置项 | 环境变量 | config.json | defaults.json |
|--------|---------|-------------|---------------|
| appId | `UUPT_APP_ID` | `appId` | `appId` |
| appSecret | `UUPT_APP_SECRET` | `appSecret` | `appSecret` |
| openId | `UUPT_OPEN_ID` | `openId` | - |
| apiUrl | `UUPT_API_URL` | `apiUrl` | `apiUrl` |

### 配置示例

**开发者模式**（用户提供凭证后写入 config.json）：
```json
{
  "appId": "你的appId",
  "appSecret": "你的appSecret", 
  "openId": "你的openId"
}
```

**快速体验模式**（注册后自动生成的 config.json）：
```json
{
  "openId": "注册成功后自动保存的openId"
}
```

### 可选 API 环境

| 环境 | URL |
|------|-----|
| 生产环境 | `https://api-open.uupt.com/openapi/v3/` |
| 测试环境 | `http://api-open.test.uupt.com/openapi/v3/` |

---

## 在代码中使用

### Node.js

```javascript
const { orderPrice, createOrder, orderDetail, cancelOrder, driverTrack } = require('./index');

// 订单询价
const priceResult = await orderPrice({
  fromAddress: '郑州市金水区农业路经三路交叉口',
  toAddress: '郑州市二七区德化街100号',
  cityName: '郑州市'
});

// 创建订单
const orderResult = await createOrder({
  priceToken: priceResult.body.priceToken,
  receiverPhone: '13800138000'
});

// 检查是否需要支付
if (orderResult.body.orderUrl) {
  console.log('需要支付，打开链接:', orderResult.body.orderUrl);
}

// 查询订单
const detailResult = await orderDetail({
  orderCode: orderResult.body.orderCode
});
```

### Python

```python
from uupt_delivery import order_price, create_order, order_detail, cancel_order, driver_track

# 订单询价
price_result = order_price(
    from_address='郑州市金水区农业路经三路交叉口',
    to_address='郑州市二七区德化街100号',
    city_name='郑州市'
)

# 创建订单
order_result = create_order(
    price_token=price_result['body']['priceToken'],
    receiver_phone='13800138000'
)

# 检查是否需要支付
if order_result['body'].get('orderUrl'):
    print('需要支付，打开链接:', order_result['body']['orderUrl'])

# 查询订单
detail_result = order_detail(
    order_code=order_result['body']['orderCode']
)
```

---

## 注意事项

- **首次使用**：首次使用时需要通过手机号验证获取授权，之后无需重复操作
- **图片验证码**：如果发送短信验证码时返回 `[IMAGE_CAPTCHA_REQUIRED]`，需要展示 base64 图片让用户识别数字后重试
- **注册重试**：授权失败时自动重试，最多 3 次（不用重新输入手机号）
- **询价有效期**：priceToken 有时效性，建议获取后尽快创建订单
- **地址完整性**：地址信息越完整，配送越准确
- **城市默认值**：如未指定城市，默认使用"郑州市"
- **价格单位**：API 返回的价格单位是分，展示时需除以 100 转换为元
- **订单状态**：创建订单后请关注订单状态变化
- **余额不足**：当返回 `[PAYMENT_REQUIRED]` 时，微信渠道用 `message` 工具发送 `{QRCODE_FILE}` 二维码图片附件；其他渠道直接发送 `{PAYMENT_URL}` 支付链接
- **配置文件**：`defaults.json` 为内置凭证，请勿修改或删除

## 相关链接

- [UU跑腿开放平台](https://open.uupt.com)
- [API 文档](https://open.uupt.com/docs)
