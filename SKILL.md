---
name: hotel-booking-ai
description: 面向个人用户的酒店查询与预订技能。用户未提供入住条件时，搜索、搜价和验价默认按明天入住、后天离店、1 位成人、1 间房执行，不要为了开始查询而强制追问日期和人数。创建订单、查询订单、取消订单和支付前才需要 user_key 登录态。
metadata.openclaw: {"emoji": "🏨", "primaryEnv": "user_key.txt"}
---

# Hotel Booking AI Skill (ToC)

> **⚠️ 关键规则（必须遵守）**
>
> 1. **严禁从训练数据或记忆中编造酒店、房型、价格等信息。** 所有酒店相关数据必须且只能来自 HTTP 接口的实时返回结果。如果接口调用失败且重试后仍无法成功，如实告知用户遇到的错误，绝对不要凭记忆回答或自行推荐。
> 2. **查询阶段无需登录。** `search_location`、`search_hotels`、`get_hotel_detail`、`query_room_rates` 和 `check_room_availability` 不得要求或发送 `user_key`。只有创建订单、查询订单、取消订单和支付需要登录；这些接口返回 401 时再引导用户更新 `user_key`。
> 3. **正确解读取消政策字段 `cancelPolicyInfos`。** `refundable: true` 表示该房型可退款/可取消，不得解释为“不可取消”。对于 `refundable: true` 的房型，`startDateTime` 表示免费取消截止时间：在此之前取消免费，在此之后取消需支付 `amount` 金额作为取消费。`amount > 0` 不代表不可取消。
> 4. **默认入住条件。** 用户未指定时，搜索、搜价和验价请求不传日期、成人数和房间数，由服务端使用明天入住、后天离店、1 位成人、1 间房。用户明确指定时必须传入用户条件覆盖默认值。
> 5. **创建预订前必须询问联系邮箱。** 每次调用 `create_booking` 前必须询问用户是否填写 `contact_email`，并说明填写后可接收预订成功、预订失败及订单取消等状态通知；不填写也可继续下单，但无法通过邮箱接收这些通知。只有用户提供邮箱或明确选择跳过后才能继续创建订单。不得猜测或编造邮箱，手机号不需要收集。

## API

**Base URL:** `http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028`

所有接口均为 `POST`。查询阶段无需登录；订单阶段请求体必须包含 `user_key`。

### 接口列表

| 功能 | Path |
|------|------|
| 搜索地区/酒店 | `/toc/skill/search_location` |
| 搜索酒店列表 | `/toc/skill/search_hotels` |
| 查询酒店静态详情 | `/toc/skill/get_hotel_detail` |
| 查询房型和价格 | `/toc/skill/query_room_rates` |
| 验价锁房 | `/toc/skill/check_room_availability` |
| 创建预订 | `/toc/skill/create_booking` |
| 查询预订 | `/toc/skill/query_booking` |
| 取消预订 | `/toc/skill/cancel_booking` |
| 发起支付 | `/toc/skill/pay_order` |

### 响应格式

成功：`{"ok": true, "data": {...}}`  
失败：`{"ok": false, "error": "错误描述"}`

### 调用方式（curl 示例）

```bash
# 搜索地区
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/search_location" \
  -d '{"keyword": "东京"}'

# 搜索酒店
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/search_hotels" \
  -d '{"region_id": "3263"}'

# 查询酒店详情
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/get_hotel_detail" \
  -d '{"hotel_id": "12345"}'

# 查询房型
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/query_room_rates" \
  -d '{"hotel_id": "12345"}'

# 验价
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/check_room_availability" \
  -d '{"hotel_id": "12345", "rate_code": "xxx"}'

# 创建预订
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/create_booking" \
  -d '{"user_key": "<user_key>", "hotel_id": "12345", "rate_code": "xxx", "check_in_date": "2026-05-01", "check_out_date": "2026-05-02", "guest_name": "张三", "contact_email": "guest@example.com", "adults": 1, "room_count": 1, "total_price": 1260.00}'

# 查询预订
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/query_booking" \
  -d '{"user_key": "<user_key>", "agent_ref_id": "TM20260501001"}'

# 取消预订
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/cancel_booking" \
  -d '{"user_key": "<user_key>", "agent_ref_id": "TM20260501001"}'

# 支付
curl -s -X POST -H "Content-Type: application/json" \
  "http://nlb-3psfnp4wzcgnlw0fe0.cn-shenzhen.nlb.aliyuncsslb.com:19028/toc/skill/pay_order" \
  -d '{"user_key": "<user_key>", "agent_ref_id": "TM20260501001", "payment_method": "微信支付"}'
```

---

## Setup

仅在用户准备创建订单或操作已有订单时完成身份验证。搜索、酒店详情、搜价和验价阶段跳过本节。

### Step 1 — User Key

1. 读取 `{baseDir}/user_key.txt`
2. 如果文件**不存在或为空** — 暂停订单操作并告知用户：
   > "在开始之前，需要先验证你的身份。请前往 https://aauth-170125614655.asia-northeast1.run.app/dashboard 用 Google 账号登录，复制你的 `user_key`（格式：`uk_xxxxxxxx`），然后告诉我。"
   用户提供后保存到 `{baseDir}/user_key.txt`，然后继续。
3. 如果文件**存在且有内容** — 直接使用，不再询问用户。
4. 如果接口返回 401 或 error 包含 `unauthorized` — 删除 `{baseDir}/user_key.txt`，重新执行第 2 步。

---

## 接口参数说明

### /toc/skill/search_location

| 参数 | 类型 | 说明 |
|------|------|------|
| keyword | string | 搜索关键词（城市名、地标、酒店名等） |

返回 `data.regions`（地区列表，含 `region_id`）和 `data.hotels`（酒店列表，含 `hotel_id`）。

### /toc/skill/search_hotels

| 参数 | 类型 | 说明 |
|------|------|------|
| region_id | string | 地区 ID（必须传字符串，如 `"3263"`） |
| check_in_date | string | 入住日期（可选，默认明天） |
| check_out_date | string | 离店日期（可选，默认入住次日） |
| adults | int | 每间客房成人数（可选，默认 1） |
| room_count | int | 房间数（可选，默认 1） |
| lowest_price | int | 最低价格（CNY，可选） |
| highest_price | int | 最高价格（CNY，可选） |

返回 `data.hotels`，最多 20 家价格最低的酒店。

### /toc/skill/get_hotel_detail

| 参数 | 类型 | 说明 |
|------|------|------|
| hotel_id | string | 酒店 ID |

返回酒店地址、星级、设施、政策、图片和静态房型信息。本接口无需登录。

### /toc/skill/query_room_rates

| 参数 | 类型 | 说明 |
|------|------|------|
| hotel_id | string | 酒店 ID |
| check_in_date | string | 入住日期（可选，默认明天） |
| check_out_date | string | 离店日期（可选，默认入住次日） |
| adults | int | 每间客房成人数（可选，默认 1） |
| room_count | int | 房间数（可选，默认 1） |

返回 `data.room_types`。每个房型包含 `room_type_code`、`name`、`name_cn`、`bed_type_desc`、`basic_room_image` 和 `products`。每个 product 按「房型 + 最大入住人数 + 餐食 + 取消政策」聚合，只保留该产品维度最低价 RP；使用 `product.rate.rate_code` 进行验价。

### /toc/skill/check_room_availability

| 参数 | 类型 | 说明 |
|------|------|------|
| hotel_id | string | 酒店 ID |
| rate_code | string | 来自 query_room_rates 的 rate_code |
| check_in_date | string | 入住日期（可选，默认明天） |
| check_out_date | string | 离店日期（可选，默认入住次日） |
| adults | int | 每间客房成人数（可选，默认 1） |
| room_count | int | 房间数（可选，默认 1） |

返回 `data.room_types`，验价成功后的实时价格和 rate_code（可能与查询时不同），以及 `cancelPolicyInfos`。

### /toc/skill/create_booking

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| hotel_id | string | 酒店 ID |
| rate_code | string | 来自 check_room_availability 的 rate_code |
| check_in_date | string | 入住日期 |
| check_out_date | string | 离店日期 |
| guest_name | string | 入住人姓名（系统自动解析中英文） |
| contact_email | string | 联系邮箱（可选）；填写后用于接收预订成功、预订失败及取消等订单通知，不填写则无法通过邮箱接收通知 |
| adults | int | 每间客房成人数 |
| room_count | int | 房间数（默认 1） |
| currency | string | 货币，默认 CNY |
| total_price | float | check_room_availability 返回的总价 |

返回 `data.agent_ref_id`（订单号）。

### /toc/skill/query_booking

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| agent_ref_id | string | create_booking 返回的订单号 |

### /toc/skill/cancel_booking

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| agent_ref_id | string | create_booking 返回的订单号 |

返回 `data.status`、`data.cancel_fee`、`data.refund_amount`（如有）、`data.currency`。取消前必须向用户确认要取消的订单号。

### /toc/skill/pay_order

| 参数 | 类型 | 说明 |
|------|------|------|
| user_key | string | 从 `{baseDir}/user_key.txt` 读取 |
| agent_ref_id | string | 订单号 |
| payment_method | string | 支付方式：`Stripe`、`微信支付`、`支付宝` |

返回 `data.pay_url`，将链接分享给用户完成支付。

---

## 预订流程

```
0. 搜索地区（匿名）  → search_location 获取 region_id
1. 搜索酒店（匿名）  → search_hotels
2. 查询详情（匿名）  → 用户需要时调用 get_hotel_detail
3. 查询房价（匿名）  → query_room_rates
4. 验价锁房（匿名）  → check_room_availability
5. 获取登录态       → 创建或操作订单前读取/获取 user_key
6. 收集预订信息     → 入住人姓名；必须询问联系邮箱，获得邮箱或用户明确跳过后继续
7. 创建预订         → create_booking（无需手机号）
8. 发起支付         → 询问支付方式后调用 pay_order
9. 查询/取消订单    → 用户明确要求后调用对应接口
```

---

## 注意事项

- **所有日期格式必须是 `YYYY-MM-DD`**
- **`region_id`、`hotel_id` 必须以字符串传入**（如 `"3263"`，不是 `3263`）
- **`total_price` 使用 `check_room_availability` 返回的价格**，不要使用 `query_room_rates` 的价格
- **查询到验价阶段不得要求 user_key**，也不要在请求中发送 user_key
- **用户未指定日期和人数时不要追问**，使用服务端默认的明天入住、住一晚、1 位成人、1 间房
- **每次调用 create_booking 前必须询问用户是否填写联系邮箱**，说明填写后可接收预订成功、失败和取消等通知；只有用户提供邮箱或明确选择跳过后才能继续，手机号不需要收集
- **create_booking 后询问支付方式**，只展示 Stripe、微信支付和支付宝，再调用 pay_order
- **取消订单前必须向用户确认订单号**，再调用 cancel_booking
- **解读取消政策时：`refundable: true` = 可退款/可取消；`startDateTime` = 免费取消截止时间；`amount` = 超过免费取消截止时间后的取消费，不代表不可取消**
- 接口调用出错时如实告知错误信息，不要编造数据或推荐替代方案

> **For detailed parameter reference, region IDs, currency codes, and troubleshooting**, see [references/parameter_guide.md](references/parameter_guide.md)
