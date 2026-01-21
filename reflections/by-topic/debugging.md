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
