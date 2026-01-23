# Payment Kit 项目反思

## 2026-01-21 - 动态定价滑点配置实现

**场景**: 为自动充值和订阅功能添加滑点配置支持

**问题**: 动态定价场景下，用户需要能够设置可接受的价格波动范围（滑点），否则汇率波动可能导致支付失败

**解决方案**:
1. 在 `auto_recharge_config` 和 `subscription` 中添加 `slippage_config` 字段
2. slippage_config 包含 `percent` (百分比) 和 `min_acceptable_rate` (最低可接受汇率)
3. 在 quote 创建时验证当前汇率是否在滑点范围内
4. 更新前端组件支持滑点配置的展示和编辑

**教训**:
- 动态定价涉及多个模块，需要统一的滑点计算逻辑
- 用户设置的滑点百分比需要转换为具体的 min_acceptable_rate 存储
- 滑点配置应该在 quote 创建时就验证，而不是等到支付时才发现问题

**标签**: dynamic-pricing, slippage, auto-recharge, subscription

---

## 2026-01-23 - 支付金额 vs 授权金额的区分

**场景**: 免费试用订阅在快捷支付时显示续费金额而非 $0

**问题**: `getFastCheckoutAmount` 返回 `total + renew * minimumCycle`，用于授权检查。但前端错误地将此金额作为"当前支付金额"展示。

**解决方案**:
1. 后端新增 `getCheckoutAmount` 调用获取纯 `total`（当前支付金额）
2. 当 `currentPaymentTotal === '0'` 时，设置 `noPaymentRequired: true`
3. 前端检测到 `noPaymentRequired` 时自动确认，跳过对话框

**教训**:
- `currentPaymentTotal`: 当前实际需支付金额，用于判断是否需要支付
- `fastCheckoutAmount`: 包含未来续费的授权金额，用于检查委托额度
- 两者用途不同，不可混用

**标签**: payment, subscription, trial, fast-checkout

---

## 2026-01-23 - 事件监听器的错误隔离

**场景**: 多个逻辑共用 `invoice.paid` 事件监听器

**问题**: 如果一个逻辑出错（如 quote 标记失败），可能影响其他逻辑执行

**解决方案**:
创建独立的事件监听器，每个监听器有自己的 try-catch：
```typescript
// 监听器 1: 原有的账单校验逻辑
events.on('invoice.paid', async ({ id }) => { /* ... */ });

// 监听器 2: 独立的 quote 标记逻辑
events.on('invoice.paid', async ({ id }) => { /* ... */ });
```

**教训**:
- 事件驱动架构中，相互独立的逻辑应使用独立的监听器
- 即使在同一监听器内，也应对每个操作单独 catch

**标签**: architecture, events, error-handling

---
