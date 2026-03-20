# 用户指令记忆

本文件记录了用户的指令、偏好和教导,用于在未来的交互中提供参考。

## 格式

### 用户指令条目
用户指令条目应遵循以下格式：

[用户指令摘要]
- Date: [YYYY-MM-DD]
- Context: [提及的场景或时间]
- Instructions:
  - [用户教导或指示的内容,逐行描述]

### 项目知识条目
Agent 在任务执行过程中发现的条目应遵循以下格式：

[项目知识摘要]
- Date: [YYYY-MM-DD]
- Context: Agent 在执行 [具体任务描述] 时发现
- Category: [代码结构|代码模式|代码生成|构建方法|测试方法|依赖关系|环境配置]
- Instructions:
  - [具体的知识点,逐行描述]

## 去重策略
- 添加新条目前,检查是否存在相似或相同的指令
- 若发现重复,跳过新条目或与已有条目合并
- 合并时,更新上下文或日期信息
- 这有助于避免冗余条目,保持记忆文件整洁

## 条目

### Linux Do Credit 项目架构
- Date: 2026-03-20
- Context: Agent 在执行积分换取码功能设计时发现
- Category: 代码结构
- Instructions:
  - 项目是一个 Linux Do 社区的积分服务平台
  - 后端使用 Go + Gin + GORM + PostgreSQL + Redis
  - 前端使用 Next.js 16 + React 19 + TypeScript + Tailwind CSS + Shadcn UI
  - 后端代码结构：internal/apps/ 下按功能模块划分，每个模块包含 routers.go、logics.go、types.go、errs.go、constants.go 等文件
  - 管理员功能放在 internal/apps/admin/ 下的子模块中
  - 数据模型在 internal/model/ 目录
  - 使用 GORM 进行数据库操作，支持事务和软删除
  - 使用 service.UpdateBalance 方法更新用户余额，确保一致性

### 红包功能参考实现
- Date: 2026-03-20
- Context: Agent 在执行积分换取码功能设计时发现
- Category: 代码模式
- Instructions:
  - 红包功能在 internal/apps/redenvelope/ 目录
  - 使用 decimal.Decimal 处理金额，避免精度问题
  - 使用 FOR UPDATE NOWAIT 锁防止并发领取
  - 兑换流程：验证 -> 更新状态 -> 更新余额 -> 创建订单
  - 所有操作在数据库事务中完成
  - 订单类型使用枚举区分不同业务场景

### 用户积分系统
- Date: 2026-03-20
- Context: Agent 在执行积分换取码功能设计时发现
- Category: 代码模式
- Instructions:
  - 用户模型包含多个余额字段：AvailableBalance（可用余额）、TotalReceive（总收入）、TotalPayment（总支出）等
  - 使用 service.UpdateBalance 方法统一更新余额，支持增加和扣减操作
  - 每次余额变更都需要创建对应的订单记录（Order 模型）
  - 订单记录包含：订单名称、付款方ID、收款方ID、金额、状态、类型、备注等字段
