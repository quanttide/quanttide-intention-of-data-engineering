# 量潮数据云的商业模式

> 来自 2026-08-02 关于 qtcloud-data 商业化的讨论。

## 一句话

> 托管执行 + 按次计量：跑一次收一次钱。其他所有功能都是挂在这个主干上的 resource。

## 1. 核心模式：托管执行 + 按次计量

服务端（Provider）提供托管执行，按次计量收费。这是唯一能收钱的核心，因为：

- 远程统计执行、AI 生成、报告生成、模板实例化，本质全是"一次 run 的不同 resource"——一个计量器，全部功能共用
- 离现状最近：Provider 已有 pipeline runner、job store、沙箱路径防护雏形，缺的只是多租户隔离、配额/超时、每次 run 的计量
- 收入与使用强挂钩，客户"跑了多少付多少"，账单自然随价值增长

## 2. 分层：CLI 获客，服务端收费

- CLI（clarify/design/implement/transfer/process）是获客入口，免费
- 服务端执行才是收入所在：客户用 CLI 白嫖，用服务端 API 就得付钱
- 链路终点从"发个链接"变成"交一份报告"，交付报告（`report`）是客户直接付费的最终产物

## 3. AI 功能：不是第二个计价器，是第一种 resource

AI（clarify/design/implement）不进单独的 token 计费体系：

```
states:
  - name: "clarify"
    type: ai
  - name: "design"
    type: ai
  - name: "implement"
    type: ai
```

- run 成本 = compute 时间 + AI 调用（token in/out × 模型单价），对外只暴露一个价格
- token 价格持续下跌，单独按 token 定价会被价格战拖死；把 AI 当成本项，AI 降价反而扩大毛利空间
- AI 是转化引擎不是收入主干：AI 生成规格 → 跑 pipeline → 为执行付费
- 架构收益：AI 生成变成 run 后，审计链自动覆盖（job 记录含 model、token 用量）

## 4. 参考定价：Modal + Posit Cloud

- **Modal**：按秒计费，只算实际执行的 compute 时间，CPU/GPU 分开计价，preemptible 打折。账单透明到客户心算得过来，这是按次计费成立的前提
- **Posit Cloud**：证明统计软件用户（R 用户，与 Stata 用户同类）愿意为"别人帮我跑环境"付钱，验证客户画像和付费意愿；但它是席位+时长混合，偏订阅，不照抄
- **不参考 Databricks**：DBU 体系计量复杂到客户算不出自己这次花了多少钱，账单复杂度是负资产

## 5. 第一里程碑

把 `POST /blueprints/{name}/runs` 升级为「多租户 + 沙箱 + 计量」的执行 API：

```
单价 = 执行时长 × 资源档位系数 × 是否可抢占
job 记录已有 started_at/finished_at → 计量字段现成
```

先做单档单价 + 按秒计量 + job 记录当账单依据，等有客户量再加档位和折扣，不做 DBU 式多档体系。
