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
