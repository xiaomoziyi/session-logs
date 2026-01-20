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

- [ ] 授权金额计算为 100TBA 的 bug
- [ ] delegation 流程在切换支付方式时未触发
- [ ] Credit 自动充值的滑点支持

---
