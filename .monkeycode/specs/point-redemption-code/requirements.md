# 积分换取码功能需求文档

## 介绍

积分换取码功能允许管理员生成可兑换积分的兑换码，用户可以使用这些兑换码来获取积分。该功能类似于红包系统，但更加灵活，支持批量创建和管理。

## 术语表

- **Redemption Code (换取码)**: 由管理员生成的一串唯一编码，用户可以通过输入该编码来兑换指定金额的积分
- **Code Status (码状态)**: 换取码的当前状态，包括未使用、已使用、已过期、已禁用
- **Batch (批次)**: 管理员可以一次性创建多个换取码，这些码属于同一个批次
- **Redemption Record (兑换记录)**: 用户使用换取码兑换积分的历史记录

## 需求

### 需求 1: 管理员创建换取码

**用户故事:** AS 管理员，I want 创建积分换取码，so that 我可以向用户发放积分奖励

#### 验收标准

1. WHEN 管理员创建换取码，THE 系统 SHALL 生成唯一的兑换码字符串（长度16位，包含大小写字母和数字）
2. WHEN 管理员指定换取码金额，THE 系统 SHALL 验证金额大于0且不超过系统配置的最大值
3. WHEN 管理员设置过期时间，THE 系统 SHALL 确保过期时间不早于当前时间
4. WHEN 管理员批量创建换取码，THE 系统 SHALL 支持两种模式：
   - 统一模式：同一批次的所有码金额和过期时间相同
   - 灵活模式：可以指定每个码的金额和过期时间（通过导入文件）
5. WHEN 管理员批量创建换取码，THE 系统 SHALL 为每个码生成唯一的兑换码字符串
6. IF 创建过程中发生错误，THE 系统 SHALL 回滚所有已创建的换取码并返回错误信息

### 需求 2: 用户兑换积分

**用户故事:** AS 普通用户，I want 使用换取码兑换积分，so that 我可以获得积分奖励

#### 验收标准

1. WHEN 用户输入有效的换取码，THE 系统 SHALL 验证码的状态为未使用且未过期
2. WHEN 换取码验证通过，THE 系统 SHALL 将指定金额的积分添加到用户的可用余额
3. WHEN 换取码验证通过，THE 系统 SHALL 将换取码状态更新为已使用并记录使用者和使用时间
4. WHEN 换取码验证通过，THE 系统 SHALL 创建一条积分收入订单记录
5. WHEN 同一用户兑换频率超过限制（每分钟10次），THE 系统 SHALL 返回"兑换过于频繁，请稍后再试"的错误提示
6. IF 换取码已被使用，THE 系统 SHALL 返回"换取码已被使用"的错误提示
7. IF 换取码已过期，THE 系统 SHALL 返回"换取码已过期"的错误提示
8. IF 换取码已被禁用，THE 系统 SHALL 返回"换取码无效"的错误提示
9. IF 换取码不存在，THE 系统 SHALL 返回"换取码不存在"的错误提示

### 需求 3: 管理员查看换取码列表

**用户故事:** AS 管理员，I want 查看换取码列表，so that 我可以了解所有换取码的使用情况

#### 验收标准

1. WHEN 管理员请求换取码列表，THE 系统 SHALL 支持按批次、状态、创建时间范围进行筛选
2. WHEN 管理员请求换取码列表，THE 系统 SHALL 支持分页查询，每页最多100条记录
3. WHEN 管理员请求换取码列表，THE 系统 SHALL 返回每个码的详细信息（码值、金额、状态、使用者、使用时间、创建时间、过期时间）
4. WHEN 换取码已被使用，THE 系统 SHALL 在列表中显示使用者用户名和头像

### 需求 4: 管理员管理换取码

**用户故事:** AS 管理员，I want 管理换取码，so that 我可以控制换取码的有效性

#### 验收标准

1. WHEN 管理员禁用未使用的换取码，THE 系统 SHALL 将码状态更新为已禁用
2. WHEN 管理员启用已禁用的换取码，THE 系统 SHALL 将码状态更新为未使用（前提是未过期）
3. IF 管理员尝试禁用已使用的换取码，THE 系统 SHALL 返回"无法禁用已使用的换取码"的错误提示
4. WHEN 管理员删除换取码，THE 系统 SHALL 软删除该记录（保留历史数据）
5. IF 管理员尝试删除已使用的换取码，THE 系统 SHALL 返回"无法删除已使用的换取码"的错误提示

### 需求 5: 用户查看兑换历史

**用户故事:** AS 普通用户，I want 查看我的兑换历史，so that 我可以了解我兑换过的积分

#### 验收标准

1. WHEN 用户请求兑换历史，THE 系统 SHALL 返回该用户的所有兑换记录
2. WHEN 用户请求兑换历史，THE 系统 SHALL 支持分页查询
3. WHEN 用户请求兑换历史，THE 系统 SHALL 按兑换时间倒序排列
4. WHEN 用户请求兑换历史，THE 系统 SHALL 返回每条记录的兑换金额、兑换时间

### 需求 6: 系统自动过期处理

**用户故事:** AS 系统，I want 自动过期未使用的换取码，so that 避免长期有效的换取码造成潜在风险

#### 验收标准

1. WHEN 换取码到达过期时间，THE 系统 SHALL 自动将状态更新为已过期
2. WHILE 用户尝试使用已过期但状态未更新的换取码，THE 系统 SHALL 返回"换取码已过期"的错误提示
3. WHEN 系统执行过期检查任务，THE 系统 SHALL 批量更新所有已过期但状态为未使用的换取码

### 需求 7: 管理员导出换取码

**用户故事:** AS 管理员，I want 导出换取码，so that 我可以方便地分发给用户

#### 验收标准

1. WHEN 管理员请求导出批次换取码，THE 系统 SHALL 生成 CSV 格式的文件
2. WHEN 管理员请求导出批次换取码，THE 系统 SHALL 包含每个码的详细信息（码值、金额、状态、过期时间、创建时间）
3. WHEN 管理员请求导出批次换取码，THE 系统 SHALL 只导出未删除的换取码
4. IF 批次中没有可导出的换取码，THE 系统 SHALL 返回"该批次没有可导出的换取码"的错误提示

## 非功能性需求

### 安全性

- 换取码字符串应具有足够的随机性，避免被猜测
- 管理员操作需要验证管理员权限
- 所有数据库操作应在事务中执行，确保数据一致性

### 性能

- 换取码验证和兑换操作应在500ms内完成
- 列表查询应支持索引优化，避免全表扫描
- 批量创建换取码时，单次最多创建1000个

### 可用性

- 系统应提供清晰的错误提示，帮助用户理解失败原因
- 管理员界面应提供搜索和筛选功能，方便管理大量换取码

## 数据模型

### RedemptionCode (换取码表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uint64 | 主键 |
| code | string(32) | 兑换码（唯一索引） |
| batch_id | uint64 | 批次ID |
| amount | decimal(20,2) | 积分金额 |
| status | enum | 状态：unused, used, expired, disabled |
| used_by | uint64 | 使用者用户ID（外键） |
| used_at | timestamp | 使用时间 |
| created_by | uint64 | 创建者管理员ID |
| expires_at | timestamp | 过期时间 |
| created_at | timestamp | 创建时间 |
| updated_at | timestamp | 更新时间 |
| deleted_at | timestamp | 软删除时间 |

### RedemptionBatch (批次表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uint64 | 主键 |
| name | string(100) | 批次名称 |
| description | text | 批次描述 |
| total_count | int | 总码数 |
| used_count | int | 已使用数量 |
| total_amount | decimal(20,2) | 总金额 |
| created_by | uint64 | 创建者管理员ID |
| created_at | timestamp | 创建时间 |

## API 接口设计

### 用户端接口

1. **POST /api/v1/redemption-code/redeem** - 兑换积分
2. **GET /api/v1/redemption-code/history** - 获取兑换历史

### 管理员接口

1. **POST /api/v1/admin/redemption-code/create** - 创建换取码（支持批量）
   - 统一模式：`{"batch_name": "...", "count": 10, "amount": "10.00", "expires_at": "2026-12-31T23:59:59Z"}`
   - 灵活模式：上传CSV文件，包含 `amount` 和 `expires_at` 列
2. **GET /api/v1/admin/redemption-code/list** - 获取换取码列表
3. **PUT /api/v1/admin/redemption-code/:id/status** - 更新换取码状态
4. **DELETE /api/v1/admin/redemption-code/:id** - 删除换取码
5. **GET /api/v1/admin/redemption-code/export/:batch_id** - 导出批次换取码为CSV
6. **GET /api/v1/admin/redemption-code/batch/list** - 获取批次列表
7. **GET /api/v1/admin/redemption-code/batch/:id** - 获取批次详情
