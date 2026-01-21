# Debugging 反思

## 2026-01-19 - 正面回答问题，不要盲目修改

**场景**: 在 debug 授权金额计算时，授权金额显示为 100TBA，但用户询问是怎么计算出来的

**问题**: 没有正面回答问题，而是直接尝试修改代码

**解决方案**:
1. 先追踪计算逻辑，找出 100TBA 是如何得出的
2. 明确说明计算过程和每一步的数值
3. 确认真正的 bug 位置后再修改

**教训**:
- 不要盲目修改代码，先 debug 找到真正原因
- 正面回答用户的问题，用户问"怎么计算的"就要给出计算过程
- 如果有重复逻辑可以抽取，但前提是计算逻辑本身是正确的

**标签**: debugging, communication, root-cause-analysis

---

## 2026-01-19 - 改动原有逻辑前必须说明原因

**场景**: 在修改滑点计算逻辑时，涉及到 quote-service 的 decimals 修改

**问题**: 未经确认就准备修改原有逻辑

**解决方案**:
1. 在改动所有原有逻辑时先向用户说明原因
2. 在用户确认后再进行调整
3. 这些方法论可以沉淀到 xiaofang skill 里

**教训**:
- 改动原有逻辑是危险操作，需要先获得确认
- 说明修改的原因、影响范围、预期结果
- 形成规范化的工作流程

**标签**: debugging, code-change, communication

---

## 2026-01-20 - 防御性代码不是解决方案，要找根本原因

**场景**: Debug `Cannot read properties of undefined (reading 'upsell')` 错误，`line_items` 中的 `price` 对象为 undefined

**问题**:
- 我添加了防御性代码（可选链 `?.`）来避免报错
- 但这只是治标不治本，没有找到为什么 `price` 会是 undefined

**根本原因** (用户发现):
- `doc.update()` 之后，Sequelize 会刷新模型实例
- `doc.line_items` 被重置回数据库原始值（只有 `price_id`，没有展开的 `price` 对象）
- 需要在 `doc.update()` 之后重新调用 `Price.expand()` 展开数据

**我的错误**:
1. 过于关注"添加防御性代码"，而没有追踪数据流找出真正的根本原因
2. 没有理解 Sequelize ORM 的行为（`update()` 会刷新实例，`toJSON()` 返回数据库值）
3. 用户说"Enriching checkout session with quotes 的 lineitems 是对的"时，我应该检查**之后**哪一步导致数据丢失
4. 我的"修复"只是表面功夫

**教训**:
- 防御性代码（`?.`）可以防止崩溃，但不能解决根本问题
- 当用户说"某个点的数据是对的"，要重点检查**之后的流程**
- 理解 ORM 的行为：
  - Sequelize `update()` 会刷新实例属性
  - 运行时赋值的属性在 `update()` 后会丢失
  - 需要在数据库操作后重新展开关联数据
- 追踪数据流：数据从哪里来 → 在哪里变化 → 在哪里丢失

**标签**: debugging, sequelize, orm, root-cause-analysis, price-expand

---

## 2026-01-22 - 变量语义混淆导致的连锁 Bug

**场景**: 修复 upsell 价格计算错误，经过多轮调试才找到真正的根因

**问题**:
- 第一轮：修复了 quote 创建使用原价的问题，但漏了 `findIndex` 查找逻辑
- 第二轮：添加了 `Price.expand({ upsell: true })`，但 quote_id 仍然没更新
- 第三轮：终于发现 `findIndex` 用错了变量

**根本原因**:
```typescript
const priceId = item.upsell_price_id || item.price_id;  // priceId = upsell price
// ...创建 quote 用 priceId (正确)...
const itemIndex = updatedLineItems.findIndex((li) => li.price_id === priceId);  // 错误！
// lineItems 的 key 是原始 price_id，不是 upsell_price_id
```

变量 `priceId` 的语义从"要使用的价格"变成了"要查找的 key"，但 lineItems 的 key 始终是原始 `price_id`。

**我的错误**:
1. **变量命名不明确**：同一个 `priceId` 变量承担了两个不同的语义
2. **数据结构约定没有理解**：lineItems 始终以原始 price_id 为 key，这是隐式约定
3. **修复时只看"要改的地方"**：改了 quote 创建逻辑，但没检查下游的 `findIndex`
4. **测试场景不完整**：只测试了"直接选 upsell"，没测试"先选原价再改 upsell"

**教训**:
- **明确命名**：使用 `selectedPriceId` vs `lookupPriceId` vs `originalPriceId`
- **画数据流图**：修改前画出 price_id → quote → lineItem → amount 的完整链路
- **搜索变量所有使用点**：改变量赋值后，grep 所有使用该变量的地方
- **测试边界场景**：列出组合矩阵（有/无 upsell × 首次/重试 × 不同支付方式）
- **理解隐式约定**：数据结构的 key 是什么？是否在所有地方保持一致？

**标签**: debugging, variable-naming, semantic-confusion, upsell, price-id, impact-analysis

---
