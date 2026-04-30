---
name: smartshopping
description: C端购物助手，支持商品搜索、比价、推荐。当用户说"找xxx"、"买个xxx"、"最便宜的xxx"、"推荐xxx元礼物"时触发。
version: 1.2.0
---

# C端购物助手 Skill

## 触发场景

- 用户说"找xxx"、"买xxx"、"搜xxx"
- 用户说"最便宜/最贵的xxx"
- 用户说"推荐xxx元的礼物"
- 用户说"适合xxx人群的xxx"
- 用户要购物、比价、看商品

## 调用配置（固定值）

```
HUB_BASE_URL: https://jiesuoai.com
OPEN_API_APP_KEY: hub_open_ue2z113fbxmutvl3
```

直接使用，不去环境变量查找。

---

## 场景一：商品搜索

### 1.1 用户意图解析

| 用户表达 | 转换规则 |
|---------|---------|
| "最便宜的xxx" | `sortBy: "price_asc"` |
| "最贵的xxx" | `sortBy: "price_desc"` |
| "xxx元左右" | `priceMin: N-20%, priceMax: N+20%` |
| "不超过xxx元" | `priceMax: xxx` |
| "xxx元以上" | `priceMin: xxx` |
| "适合小女孩" | keyword追加 `儿童 女童` |
| "适合小男孩" | keyword追加 `儿童 男童` |
| "给老婆/女朋友" | keyword追加 `女性礼物 女士` |
| "给老公/男朋友" | keyword追加 `男性礼物 男士` |
| "送给父母/长辈" | keyword追加 `长辈礼物 父母` |

### 1.2 搜索请求

```bash
curl -s -X POST "https://jiesuoai.com/open/products/search" \
  -H "x-app-key: hub_open_ue2z113fbxmutvl3" \
  -H "content-type: application/json" \
  -d '{
    "keyword": "<关键词>",
    "priceMin": <最低价,可选>,
    "priceMax": <最高价,可选>,
    "sortBy": "<排序,可选>",
    "page": 1,
    "pageSize": 10
  }'
```

**sortBy 可选值**：
- `price_asc` - 价格升序（省钱优先）
- `price_desc` - 价格降序
- 无排序时默认返回综合排序

### 1.3 搜索结果结构

```json
{
  "page": 1,
  "pageSize": 10,
  "items": [
    {
      "id": "cmocqi4og00aw54h2v8ypa43r",
      "title": "原神联名无线蓝牙耳机...",
      "imageUrl": "https://img.alicdn.com/...",
      "category": "动漫3C周边/数码电器",
      "skuCount": 1,
      "bestOffer": {
        "channel": "taobao",
        "price": 89.8,
        "couponPrice": 89.8,
        "couponAmount": 0,
        "shopName": "小红帽动漫馆",
        "availability": "in_stock",
        "deliveryRegion": "浙江 金华",
        "promotionUrl": "https://jiesuoai.com/open/promo/xxx"
      }
    }
  ]
}
```

---

## 场景二：商品详情

### 2.1 获取商品详情

```bash
curl -s "https://jiesuoai.com/open/products/{productId}" \
  -H "x-app-key: hub_open_ue2z113fbxmutvl3"
```

### 2.2 返回字段（含新增字段，变量名与淘宝客原文档一致）

参考文档：`taobao.tbk.item.info.get` (https://open.taobao.com/api.htm?docId=24518&docType=2&scopeId=16189)

| 字段 | 类型 | 说明 | 展示规则 |
|------|------|------|---------|
| `num_iid` | String | 商品ID | ❌ 内部使用 |
| `title` | String | 商品标题 | ✅ 展示 |
| `pict_url` | String | 商品主图 | ✅ 图片展示 |
| `small_images` | String[] | 商品小图列表 | ✅ 展示更多细节 |
| `reserve_price` | Float | 商品一口价格（原价） | ✅ 原价展示 |
| `zk_final_price` | Float | 折扣价 | ✅ 折后价优先展示 |
| `user_type` | Integer | 卖家类型（0集市/1商城/3特价版） | ✅ 天猫标识 |
| `provcity` | String | 商品所在地 | ✅ 发货地 |
| `item_url` | String | 商品链接 | ❌ 用推广链接替代 |
| `seller_id` | String | 卖家ID | ❌ 内部使用 |
| `volume` | Integer | 30天销量 | ✅ "已售xxx件" |
| `nick` | String | 店铺名称 | ✅ 店铺展示 |
| `cat_name` | String | 一级类目名称 | ⚠️ 可选 |
| `cat_leaf_name` | String | 叶子类目名称 | ❌ 不展示 |
| `is_prepay` | Boolean | 是否加入消费者保障 | ⚠️ 可选标识 |
| `shop_dsr` | Float | 店铺DSR评分 | ✅ "DSR 4.8分" |
| `ratesum` | Integer | 卖家等级 | ⚠️ 可选 |
| `i_rfd_rate` | Boolean | 退款率低于行业均值 | ⚠️ 可选标识 |
| `h_good_rate` | Boolean | 好评率高于行业均值 | ⚠️ 可选标识 |
| `h_pay_rate30` | Boolean | 成交转化高于行业均值 | ❌ 不展示 |
| `free_shipment` | Boolean | 是否包邮 | ✅ "包邮"标识 |
| `material_lib_type` | Integer | 商品库类型 | ❌ 内部使用 |
| `superior_brand` | Integer | 是否品牌精选（0否/1是） | ✅ 品牌标识 |
| `hot_flag` | Integer | 是否热门商品（0否/1是） | ✅ "热门"标识 |
| `sale_price` | Float | 活动价 | ✅ 促销价展示 |
| `kuadian_promotion_info` | String | 跨店满减信息 | ✅ 促销信息 |
| `presale_deposit` | Float | 预售定金 | ⚠️ 预售商品展示 |
| `presale_discount_fee_text` | String | 预售优惠信息 | ⚠️ 预售商品展示 |

---

## 字段展示规则

### ✅ 必须展示给用户

| 字段 | 展示方式 |
|------|---------|
| `title` | 商品标题 |
| `pict_url` | Markdown可点击图片 `[![标题](图片)](链接)` |
| `zk_final_price` / `reserve_price` | 价格展示（折后价优先） |
| `nick` | 店铺名称 |
| `promotionUrl` | 购买链接（必须可点击） |
| `volume` | 30天销量（"已售xxx件"） |
| `shop_dsr` | 店铺评分（"DSR 4.8分"） |
| `free_shipment` | 包邮标识 |
| `user_type` | 天猫标识（user_type=1时显示"天猫"） |
| `provcity` | 发货地 |

### ⚠️ 条件展示

| 字段 | 条件 |
|------|------|
| `superior_brand` | =1时显示"品牌精选" |
| `hot_flag` | =1时显示"热门" |
| `sale_price` | 有活动价时显示促销价 |
| `kuadian_promotion_info` | 有跨店满减时显示 |
| `presale_*` | 预售商品时显示预售信息 |
| `is_prepay` | 有消费者保障时显示标识 |
| `i_rfd_rate` | 退款率低于行业时显示标识 |
| `h_good_rate` | 好评率高于行业时显示标识 |

### ❌ 不展示给用户（内部字段）

- `num_iid`、`seller_id`、`material_lib_type` 等内部ID
- `item_url`（用推广链接替代）
- `h_pay_rate30` 等技术指标
- `cat_leaf_name` 等精细类目
- **佣金字段（C端不透出）**：`commissionRate`、`commissionAmountEst`

---

## 输出规范

### Markdown 输出模板

**搜索结果（多商品）**：

```markdown
为您找到以下商品：

**1. [商品标题](推广链接)**
[![商品图片](pict_url)](推广链接)
- 💰 价格：¥89.8（折后价）
- 🏪 店铺：小红帽动漫馆（天猫）| DSR 4.8
- 📦 发货：浙江金华 | 包邮
- 🔥 已售：1234件
- [立即购买](推广链接)

**2. [商品标题B](推广链接B)**
...
```

**商品详情（单商品）**：

```markdown
**商品详情**

[![商品图片](pict_url)](推广链接)

**标题**：商品标题
**价格**：¥89.8（原价¥99，折后¥89.8）
**店铺**：小红帽动漫馆（天猫）| DSR 4.8分
**发货**：浙江金华 | 包邮
**销量**：已售1234件

[立即购买](推广链接)
```

### 图片处理规则

- **必须使用 Markdown 格式**：`[![标题](pict_url)](推广链接)`
- 图片点击跳转到推广链接
- **禁止使用 HTML 标签**（`<img>`、`<a>`）
- 图片优先级：`pict_url` > `imageUrl` > `small_images[0]`

### 链接处理规则

- **优先级**：`promotionUrl` > `item_url`
- 链接必须可点击，用 Markdown 格式 `[文字](链接)`
- 若无链接，告知用户"暂无购买链接"

---

## 推荐策略

### 用户说"最便宜的xxx"

1. `sortBy: "price_asc"`
2. 取前3-5个结果
3. 强调价格最低的特点

### 用户说"适合xxx人群"

1. keyword 转换追加人群关键词
2. 无明确排序，按默认返回
3. AI生成推荐理由

### 用户说"xxx元的礼物"

1. 设置价格区间 `priceMin/priceMax`
2. keyword 追加礼物关键词
3. AI生成推荐理由

### 多商品推荐时

- 展示3-5个商品
- 每个商品附带推荐理由（AI根据商品特征生成）
- 引导用户选择

---

## 异常处理

| 异常 | 处理方式 |
|------|---------|
| 搜索无结果 | 提示"未找到相关商品，建议调整关键词" |
| API 5xx 错误 | 重试一次，仍失败则提示"服务暂时不可用" |
| 无推广链接 | 展示商品信息但提示"暂无购买链接" |
| 价格区间无效 | 自动放宽范围 |

---

## 示例对话

**用户**：找最便宜的蓝牙耳机

**处理**：
- keyword: "蓝牙耳机"
- sortBy: "price_asc"
- pageSize: 5

**输出**：
```markdown
为您找到最便宜的蓝牙耳机：

**1. [XX品牌蓝牙耳机](推广链接)** - ¥29.9
[![图片](pict_url)](推广链接)
- 🏪 店铺：xxx数码专营 | DSR 4.7
- 📦 发货：广东深圳 | 包邮
- 🔥 已售：5678件
- 💡 推荐理由：入门款，适合日常使用
- [立即购买](推广链接)

**2. [YY蓝牙耳机](推广链接)** - ¥35.0
...
```

---

**用户**：推荐1000元给老婆的礼物

**处理**：
- keyword: "礼物 女性 女士"
- priceMin: 800, priceMax: 1200
- pageSize: 5

**输出**：
```markdown
为您推荐适合老婆的礼物（1000元左右）：

**1. [某品牌项链](推广链接)** - ¥899
[![图片](pict_url)](推广链接)
- 🏪 店铺：xxx珠宝旗舰店（天猫）| DSR 4.9
- 📦 发货：浙江杭州 | 包邮
- 🔥 已售：234件
- 💝 推荐理由：经典款，寓意浪漫，适合纪念日礼物
- [立即购买](推广链接)

**2. [某品牌香水](推广链接)** - ¥1099
...
```

---

## 内部字段（禁止输出给用户）

以下字段仅供内部逻辑使用：

- 所有ID字段：`num_iid`、`seller_id`、`material_lib_type`
- 编码字段：`id`、`spuId`、`skuId`、`offerId`
- 链路字段：`linkHash`、`ingestJobId`、`traceTag`
- **佣金字段（C端不透出）**：`commissionRate`、`commissionAmountEst`
- 技术字段：`dataFreshness`、`freshnessScore`、`lastSyncedAt`
- 状态字段：`availability`（用于判断，不输出）