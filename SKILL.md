---
name: b2b-booking
description: 酒店预订技能。**仅当**用户明确表达要预订酒店、查询房价、或确认已有订单时才触发（例如"帮我订酒店"、"查一下北京的酒店"、"我要预定房间"）。纯粹的旅游计划、问路、景点推荐等不触发此skill。进入工作流后，调用工具前必须先确认地点、入住日期、离店日期、人数这四项信息，缺少任何一项则先向用户询问补齐。在调用接口过程中如遇到任何错误，如实告知用户遇到了具体错误信息，不要自行推荐替代方案或编造信息。
metadata.openclaw: {"emoji": "🏨", "primaryEnv": "user_key.txt"}
---

# B2B Booking Skill

> **⚠️ 关键规则（必须遵守）**
>
> 1. **严禁从训练数据或记忆中编造酒店、房型、价格等信息。** 所有酒店相关数据必须且只能来自 HTTP 接口的实时返回结果。如果接口调用失败且重试后仍无法成功，如实告知用户遇到的错误，绝对不要凭记忆回答或自行推荐。
> 2. **接口返回 HTTP 401 或 `{"ok": false, "error": "unauthorized: ..."}` 时，说明 user_key 无效或已过期，必须停止流程：删除 `{baseDir}/user_key.txt`，提示用户前往 [AgentAuth Dashboard](https://aauth-170125614655.asia-northeast1.run.app/dashboard) 重新获取 user_key 后再继续。**
> 3. **正确解读取消政策字段 `cancelPolicyInfos`。** `refundable: true` 表示该房型可退款/可取消，不得解释为“不可取消”。对于 `refundable: true` 的房型，`startDateTime` 表示免费取消截止时间：在此之前取消免费，在此之后取消需支付 `amount` 金额作为取消费。`amount > 0` 不代表不可取消。

## API

**Base URL:** `http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028`

所有接口均为 `POST`，请求体中需包含 `user_key` 字段。

### 接口列表

| 功能 | Path |
|------|------|
| 搜索地区/酒店 | `/skill/search_location` |
| 搜索酒店列表 | `/skill/search_hotels` |
| 查询房型和价格 | `/skill/query_room_rates` |
| 验价锁房 | `/skill/check_room_availability` |
| 创建预订 | `/skill/create_booking` |
| 查询预订 | `/skill/query_booking` |
| 取消预订 | `/skill/cancel_booking` |
| 发起支付 | `/skill/pay_order` |

### 响应格式

成功：`{"ok": true, "data": {...}}`  
失败：`{"ok": false, "error": "错误描述"}`

### 调用方式（curl 示例）

```bash
# 搜索地区
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/search_location" \
  -d '{"user_key": "<user_key>", "keyword": "东京"}'

# 搜索酒店
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/search_hotels" \
  -d '{"user_key": "<user_key>", "region_id": "3263", "check_in_date": "2026-05-01", "check_out_date": "2026-05-03", "adults": 2}'

# 查询房型
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/query_room_rates" \
  -d '{"user_key": "<user_key>", "hotel_id": "12345", "check_in_date": "2026-05-01", "check_out_date": "2026-05-03", "adults": 2, "room_count": 1}'

# 验价
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/check_room_availability" \
  -d '{"user_key": "<user_key>", "hotel_id": "12345", "rate_code": "xxx", "check_in_date": "2026-05-01", "check_out_date": "2026-05-03", "adults": 2, "room_count": 1}'

# 创建预订
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/create_booking" \
  -d '{"user_key": "<user_key>", "hotel_id": "12345", "rate_code": "xxx", "check_in_date": "2026-05-01", "check_out_date": "2026-05-03", "guest_name": "张三", "adults": 2, "room_count": 1, "total_price": 1260.00}'

# 查询预订
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/query_booking" \
  -d '{"user_key": "<user_key>", "agent_ref_id": "TM20260501001"}'

# 取消预订
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/cancel_booking" \
  -d '{"user_key": "<user_key>", "agent_ref_id": "TM20260501001"}'

# 支付
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/skill/pay_order" \
  -d '{"user_key": "<user_key>", "agent_ref_id": "TM20260501001", "payment_type": 11}'
```

---

## Setup

调用任何接口前，必须先完成用户身份验证。

### Step 1 — User Key

1. 读取 `{baseDir}/user_key.txt`
2. 如果文件**不存在或为空** — 不要调用任何接口，告知用户：
   > "在开始之前，需要先验证你的身份。请前往 https://aauth-170125614655.asia-northeast1.run.app/dashboard 用 Google 账号登录，复制你的 `user_key`（格式：`uk_xxxxxxxx`），然后告诉我。"
   用户提供后保存到 `{baseDir}/user_key.txt`，然后继续。
3. 如果文件**存在且有内容** — 直接使用，不再询问用户。
4. 如果接口返回 401 或 error 包含 `unauthorized` — 删除 `{baseDir}/user_key.txt`，重新执行第 2 步。

---

## 接口参数说明

### /skill/search_location

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| keyword | string | 搜索关键词（城市名、地标、酒店名等） |

返回 `data.regions`（地区列表，含 `region_id`）和 `data.hotels`（酒店列表，含 `hotel_id`）。

### /skill/search_hotels

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| region_id | string | 地区 ID（必须传字符串，如 `"3263"`） |
| check_in_date | string | 入住日期 YYYY-MM-DD |
| check_out_date | string | 离店日期 YYYY-MM-DD |
| adults | int | 每间客房成人数 |
| lowest_price | int | 最低价格（CNY，可选） |
| highest_price | int | 最高价格（CNY，可选） |

返回 `data.hotels`，最多 3 家价格最低的酒店。

### /skill/query_room_rates

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| hotel_id | string | 酒店 ID |
| check_in_date | string | 入住日期 |
| check_out_date | string | 离店日期 |
| adults | int | 每间客房成人数 |
| room_count | int | 房间数（默认 1） |

返回 `data.room_types`，每个房型含 `rate_code`、`total_price`、`currency_code`、`meal_info`、`refundable`、`cancelPolicyInfos`。

### /skill/check_room_availability

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| hotel_id | string | 酒店 ID |
| rate_code | string | 来自 query_room_rates 的 rate_code |
| check_in_date | string | 入住日期 |
| check_out_date | string | 离店日期 |
| adults | int | 每间客房成人数 |
| room_count | int | 房间数（默认 1） |

返回 `data.room_types`，验价成功后的实时价格和 rate_code（可能与查询时不同），以及 `cancelPolicyInfos`。

### /skill/create_booking

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| hotel_id | string | 酒店 ID |
| rate_code | string | 来自 check_room_availability 的 rate_code |
| check_in_date | string | 入住日期 |
| check_out_date | string | 离店日期 |
| guest_name | string | 入住人姓名（系统自动解析中英文） |
| adults | int | 每间客房成人数 |
| room_count | int | 房间数（默认 1） |
| currency | string | 货币，默认 CNY |
| total_price | float | check_room_availability 返回的总价 |

返回 `data.agent_ref_id`（订单号）。

### /skill/query_booking

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| agent_ref_id | string | create_booking 返回的订单号 |

### /skill/cancel_booking

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| agent_ref_id | string | create_booking 返回的订单号 |

返回 `data.status`、`data.cancel_fee`、`data.refund_amount`（如有）、`data.currency`。取消前必须向用户确认要取消的订单号。

### /skill/pay_order

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| agent_ref_id | string | 订单号 |
| payment_type | int | `11` = 微信支付，`12` = 支付宝 |
| return_url | string | 支付完成跳转地址（可选） |

返回 `data.pay_url`，将链接分享给用户完成支付。

---

## 预订流程

```
0. 搜索地区（如需要）→ search_location 获取 region_id
1. 搜索酒店         → search_hotels
2. 查询房型价格     → query_room_rates
3. 验价锁房         → check_room_availability
4. 创建预订         → create_booking（无需手机号和邮箱）
5. 发起支付         → 询问支付方式后调用 pay_order
6. 查询订单         → query_booking（随时可查）
7. 取消订单         → 用户明确要求取消且确认订单号后调用 cancel_booking
```

---

## 注意事项

- **所有日期格式必须是 `YYYY-MM-DD`**
- **`region_id`、`hotel_id` 必须以字符串传入**（如 `"3263"`，不是 `3263`）
- **`total_price` 使用 `check_room_availability` 返回的价格**，不要使用 `query_room_rates` 的价格
- **不要主动收集手机号和邮箱** — 预订流程不需要
- **create_booking 后询问支付方式**，再调用 pay_order
- **取消订单前必须向用户确认订单号**，再调用 cancel_booking
- **解读取消政策时：`refundable: true` = 可退款/可取消；`startDateTime` = 免费取消截止时间；`amount` = 超过免费取消截止时间后的取消费，不代表不可取消**
- **展示多家酒店房型时，每家酒店的房型数据必须严格来自该酒店 `hotel_id` 对应的 `query_room_rates` 返回结果，禁止将其他酒店的数据混用；若某酒店返回 `room_types: None` 或 `total: 0`，只能显示"该酒店暂无可用房型"，不得用其他酒店数据填充**
- 接口调用出错时如实告知错误信息，不要编造数据或推荐替代方案

> **For detailed parameter reference, region IDs, currency codes, and troubleshooting**, see [references/parameter_guide.md](references/parameter_guide.md)
