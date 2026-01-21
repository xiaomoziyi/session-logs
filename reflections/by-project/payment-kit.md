# Payment Kit 项目反思

## 2026-01-19 - 滑点配置改造

### 关键经验

1. **min_acceptable_rate 优先于 slippage_percent**
   - 存储精确汇率，而非百分比
   - 用户设置百分比时立即转换为精确值

2. **授权金额计算要考虑滑点**
   - delegation 金额 = 订阅价格 / 最低可接受汇率
   - 不是用当前汇率计算，而是用滑点下限计算

3. **切换支付方式的 delegation 流程**
   - connect/change-payment 本身是一次 connect
   - 需要检查授权金额是否低于阈值
   - 如果低于阈值需要重新发起 delegation

4. **汇率精度处理**
   - 参考汇率使用两位精度（如 0.26$）
   - 简化滑点计算，小数位数多不会有太大影响

### 相关文件

- `api/src/routes/checkout-session/slippage.ts` - 滑点 API
- `api/src/services/quote-service.ts` - 汇率服务
- `api/src/routes/connect/delegation.ts` - 委托授权
- `web/src/components/SlippageConfig.tsx` - 滑点配置组件

### 待解决问题

- [x] 授权金额计算为 100TBA 的 bug (已修复 2026-01-20)
- [x] delegation 流程在切换支付方式时未触发 (已修复 2026-01-20)
- [ ] Credit 自动充值的滑点支持

---

## 2026-01-20 - 汇率缓存与 Re-authorization 修复

### 关键经验

1. **汇率缓存设计**
   - Provider 调用频率过高会触发 rate limit
   - 通过环境变量控制缓存时长 (默认10分钟)
   - 缓存在 quote-service 层实现

2. **Re-authorization 触发条件**
   - 切换支付方式时必须检查当前授权金额
   - 授权金额低于阈值时重新发起 delegation
   - change-payment 本身也是一次 connect 操作

3. **授权金额计算已修复**
   - 使用 min_acceptable_rate 计算
   - 100TBA bug 根因是未使用正确的汇率

### 相关文件

- `api/src/services/quote-service.ts` - 添加缓存逻辑
- `api/src/routes/connect/delegation.ts` - 修复 re-authorization

### 待跟进

- [ ] 监控生产环境 rate limit 情况
- [ ] Credit 自动充值的滑点支持

---
