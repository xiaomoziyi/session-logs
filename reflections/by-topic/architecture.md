# Architecture 反思

## 2026-01-19 - 滑点配置应该存储精确汇率而非百分比

**场景**: Payment Kit 订阅的滑点配置功能设计

**问题**:
- 原设计只存储 slippage_percent（百分比）
- 每次展示都需要基于当前汇率动态计算最低可接受汇率
- 但用户设置百分比时是基于当时的汇率，汇率变化后百分比的含义也变了

**解决方案**:
1. 维护 min_acceptable_rate（精确的最低可接受汇率）
2. 用户设置百分比时，立即计算并保存精确汇率
3. 后端 API 支持保存精确汇率设置
4. 前端统一展示精确汇率，而非百分比

**教训**:
- 用户设置百分比，本质是想要一个精确的汇率下限
- 存储计算结果而非计算参数，避免重复计算和语义漂移
- UI 展示应该简化，订阅详情页只需展示最低可接受汇率

**标签**: architecture, payment, slippage, data-modeling

---

## 2026-01-19 - 组件复用：滑点配置组件

**场景**: 支付页和订阅详情页都需要滑点配置功能

**问题**:
- 原来支付页和配置页各自实现了滑点配置组件
- 两个组件逻辑相似但不完全相同，维护成本高

**解决方案**:
1. 抽取统一的滑点配置组件
2. 组件支持传入当前汇率作为参数
3. 两个页面复用同一组件

**教训**:
- 相似功能应该抽取复用，而非复制代码
- 组件设计要考虑不同场景的差异点，通过参数化处理

**标签**: architecture, component-design, code-reuse

---

## 2026-01-19 - 资金安全：自动充值的汇率检查

**场景**: Credit 自动充值场景需要支持动态计价

**问题**:
- 自动充值流程如果汇率波动，可能超出用户的滑点下限
- 多次自动充值可能导致超额扣款

**解决方案**:
1. 创建账单前检查汇率和滑点
2. 如果汇率低于滑点下限，通知用户说明原因
3. 约束单次充值必须能覆盖欠费额度
4. 不允许多次自动充值来覆盖余额

**教训**:
- 资金相关的自动化操作必须有严格的安全检查
- 放行自动扣款是危险操作，必须谨慎
- 无法保证的情况下，宁可不自动执行

**标签**: architecture, security, payment, auto-topup

---

## 2026-01-23 - Final Freeze 架构下的 Quote 管理

**场景**: payment-kit 动态定价 quote 功能

**要点**:
1. Quote 仅在 Submit 时创建，不在 preview 阶段
2. adjust-quantity 不应保留 quote 数据，价格由前端实时计算
3. Quote 状态流转: pending → used → paid

**相关代码**:
- `checkout-sessions.ts`: adjust-quantity 清除 quote 字段
- `shared.ts`: ensureInvoiceForCheckout 传递 quote_id
- `invoice.ts`: invoice.paid 事件标记 quotes

**标签**: payment-kit, quote, final-freeze

---
