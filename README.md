# Hotel Booking AI - ToC Skill

面向个人用户的酒店查询与预订技能。酒店搜索、详情、搜价和验价无需登录，创建及操作订单时才需要 AgentAuth `user_key`。

## 功能

- **搜索地区/酒店**：按城市名、地标、酒店名模糊搜索
- **搜索酒店列表**：按地区和日期查询最低价酒店
- **查询酒店详情**：获取地址、设施、政策、图片和静态房型
- **查询房型价格**：获取指定酒店的全部房型和实时价格
- **验价锁房**：锁定房价，确保预订前价格有效
- **创建预订**：提交订单，支持中英文姓名自动解析
- **查询预订**：随时查看订单状态和确认号
- **发起支付**：支持 Stripe、微信支付和支付宝

## 目录结构

```
├── SKILL.md              # Skill 主文件（OpenClaw 加载）
├── README.md             # 本文档
├── EVAL_GUIDE.md         # 测试评估指南
├── TEST_REPORT.md        # 测试报告
├── evals/
│   └── evals.json        # 测试用例
├── references/
│   └── parameter_guide.md  # 参数参考（region_id、货币代码等）
└── scripts/
    └── validate_booking.py # 接口测试脚本
```

## 快速部署

### 1. 启动 Skill HTTP Server

```bash
go build -o chls-skill ./mcp/cmd/
nohup ./chls-skill -port :9061 > chls-skill.log 2>&1 &
```

所需环境变量与主业务服务相同（数据库、RPC 地址等）。

### 2. 安装 Skill

将 `SKILL.md` 复制到 OpenClaw 技能目录，重启网关：

```bash
mkdir -p ~/.openclaw/skills/hotel-booking-ai
cp SKILL.md ~/.openclaw/skills/hotel-booking-ai/
openclaw gateway restart
```

无需在 `openclaw.json` 中配置任何 MCP server，Skill 直接通过 HTTP 调用接口。

## API

**Base URL:** `http://39.108.114.224:9028`

| 接口 | 说明 |
|------|------|
| `POST /toc/skill/search_location` | 搜索地区或酒店 |
| `POST /toc/skill/search_hotels` | 搜索酒店列表 |
| `POST /toc/skill/get_hotel_detail` | 查询酒店静态详情 |
| `POST /toc/skill/query_room_rates` | 查询房型和价格 |
| `POST /toc/skill/check_room_availability` | 验价锁房 |
| `POST /toc/skill/create_booking` | 创建预订 |
| `POST /toc/skill/query_booking` | 查询预订 |
| `POST /toc/skill/cancel_booking` | 取消预订 |
| `POST /toc/skill/pay_order` | 发起支付 |

搜索、详情、搜价和验价请求无需 `user_key`。创建预订、查询预订、取消预订和支付必须包含从 [AgentAuth Dashboard](https://aauth-170125614655.asia-northeast1.run.app/dashboard) 获取的 `user_key`。

用户未指定入住条件时，服务端默认明天入住、后天离店、1 位成人、1 间房。

## 使用示例

```
用户：帮我订一下东京的酒店，4月28日入住，4月30日离店，2人

机器人：好的，我来帮您搜索东京的酒店。
       找到以下 3 家酒店：
       1. 东京希尔顿酒店 — ¥1,280/晚
       2. 新宿华盛顿酒店 — ¥850/晚
       3. 东京皇家公园酒店 — ¥720/晚
       请问您选择哪家？

用户：第2家，标准间

机器人：已验价，标准大床房 ¥850/晚，2晚共 ¥1,700。
       请提供入住人姓名。是否需要填写联系邮箱？填写后可接收预订成功、失败和取消等订单通知；不填写也可以继续下单。

用户：张三

机器人：订单创建成功！订单号：TM20260428001
       请选择支付方式：微信支付 或 支付宝

用户：微信支付

机器人：已发起微信支付，请点击链接完成支付：https://...
```

## 参数参考

常用 region_id 见 [references/parameter_guide.md](references/parameter_guide.md)。

常用地区：北京=569，上海=2862，杭州=1328，深圳=3045，大阪=2446，东京=3263，曼谷=575
