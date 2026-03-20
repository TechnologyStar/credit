# 红色答题游戏功能需求文档

## 介绍

红色答题游戏是一个趣味性的积分获取功能，用户可以通过回答问题来赚取积分。系统支持单机模式和联机模式两种游戏方式，并设有排行榜功能，增加用户参与度和社区活跃度。

## 术语表

- **Quiz Game (答题游戏)**: 用户通过回答问题来获取积分的游戏模式
- **Single Player Mode (单机模式)**: 用户独立答题，不与其他玩家实时互动
- **Multiplayer Mode (联机模式)**: 多个用户同时答题，实时竞争
- **Quiz Question (题目)**: 答题游戏中的问题，包含题目内容、选项和正确答案
- **Quiz Session (答题会话)**: 一次完整的答题过程，包含多道题目
- **Quiz Score (答题分数)**: 用户在一次答题会话中获得的总分
- **Leaderboard (排行榜)**: 按答题分数排名的用户列表

## 需求

### 需求 1: 单机答题模式

**用户故事:** AS 普通用户，I want 在单机模式下答题，so that 我可以独立练习并赚取积分

#### 验收标准

1. WHEN 用户开始单机答题，THE 系统 SHALL 从题库中随机抽取指定数量的题目
2. WHEN 用户开始单机答题，THE 系统 SHALL 显示题目和选项（如果是选择题）
3. WHEN 用户选择答案，THE 系统 SHALL 立即显示答题结果（正确/错误）和正确答案
4. WHEN 用户完成所有题目，THE 系统 SHALL 计算总分并显示答题统计
5. WHEN 用户完成所有题目，THE 系统 SHALL 将答题获得的积分添加到用户余额
6. WHEN 用户答题会话结束，THE 系统 SHALL 创建一条答题收入订单记录
7. IF 用户已达到每日答题次数上限，THE 系统 SHALL 返回"今日答题次数已达上限"的错误提示

### 需求 2: 联机答题模式

**用户故事:** AS 普通用户，I want 在联机模式下答题，so that 我可以与其他用户实时竞争

#### 验收标准

1. WHEN 用户加入联机答题，THE 系统 SHALL 将用户加入匹配队列
2. WHEN 匹配到 2-4 名玩家，THE 系统 SHALL 通过 WebSocket 创建答题房间并开始答题会话
3. WHEN 答题会话开始，THE 系统 SHALL 向所有玩家实时同步显示同一道题目
4. WHEN 玩家选择答案，THE 系统 SHALL 通过 WebSocket 实时记录答题时间和答案
5. WHEN 答题时间结束（每题15秒）或所有玩家已作答，THE 系统 SHALL 显示所有玩家的答题结果
6. WHEN 所有15道题目完成，THE 系统 SHALL 根据玩家排名分配积分奖励
7. WHEN 所有题目完成，THE 系统 SHALL 显示所有玩家的最终排名和得分
8. IF 玩家中途退出，THE 系统 SHALL 视为放弃，该玩家不获得积分奖励

### 需求 3: 答题积分计算

**用户故事:** AS 系统，I want 根据答题表现计算积分，so that 公平地奖励用户

#### 验收标准

1. WHEN 用户答对题目，THE 系统 SHALL 给予10分基础分
2. WHEN 用户是本道题答题最快的玩家，THE 系统 SHALL 额外给予5分时间奖励
3. WHEN 用户答错题目，THE 系统 SHALL 不扣分（答错不得分）
4. WHEN 用户在联机模式中获得第一名，THE 系统 SHALL 额外给予排名奖励（第一名20%加成）
5. WHEN 用户完成答题会话，THE 系统 SHALL 计算积分 = (基础分 + 时间奖励) × 倍率 + 排名奖励
6. WHEN 管理员调整积分倍率，THE 系统 SHALL 根据倍率调整所有用户的积分奖励

### 需求 4: 管理员配置答题参数

**用户故事:** AS 管理员，I want 配置答题游戏参数，so that 控制游戏的难度和积分发放

#### 验收标准

1. WHEN 管理员设置每日答题次数上限，THE 系统 SHALL 限制用户每日最多答题次数不超过该值
2. WHEN 管理员设置每次答题题目数量，THE 系统 SHALL 确保每次答题包含指定数量的题目
3. WHEN 管理员设置答题时间限制（每题），THE 系统 SHALL 在时间到达后自动提交答案
4. WHEN 管理员设置积分奖励倍率，THE 系统 SHALL 根据倍率调整所有积分奖励
5. WHEN 管理员启用/禁用答题功能，THE 系统 SHALL 相应地允许或拒绝用户访问答题功能
6. WHEN 管理员设置联机模式最小/最大玩家数，THE 系统 SHALL 确保参与人数在范围内

### 需求 5: 管理员管理题库

**用户故事:** AS 管理员，I want 管理题库，so that 提供丰富的答题内容

#### 验收标准

1. WHEN 管理员添加题目，THE 系统 SHALL 保存题目内容、选项、正确答案
2. WHEN 管理员添加题目，THE 系统 SHALL 支持多种题型（单选题、多选题、判断题）
3. WHEN 管理员编辑题目，THE 系统 SHALL 更新题目信息
4. WHEN 管理员删除题目，THE 系统 SHALL 软删除题目（正在进行的答题会话不受影响）
5. WHEN 管理员导入题目，THE 系统 SHALL 支持批量导入（CSV/JSON格式）
6. WHEN 管理员导出题目，THE 系统 SHALL 支持导出为CSV/JSON格式
7. WHEN 管理员配置第三方题库，THE 系统 SHALL 支持从第三方题库API获取题目

### 需求 6: 答题排行榜

**用户故事:** AS 普通用户，I want 查看答题排行榜，so that 了解自己的排名和其他玩家的表现

#### 验收标准

1. WHEN 用户请求排行榜，THE 系统 SHALL 显示总分排名前100的用户
2. WHEN 用户请求排行榜，THE 系统 SHALL 支持按不同时间范围筛选（日榜、周榜、月榜、总榜）
3. WHEN 用户请求排行榜，THE 系统 SHALL 显示每个用户的排名、用户名、头像、总积分、答题次数
4. WHEN 用户在排行榜中，THE 系统 SHALL 高亮显示当前用户的排名
5. WHEN 用户请求排行榜，THE 系统 SHALL 支持分页查询

### 需求 7: 用户答题历史

**用户故事:** AS 普通用户，I want 查看我的答题历史，so that 了解我的答题表现和进步

#### 验收标准

1. WHEN 用户请求答题历史，THE 系统 SHALL 返回该用户的所有答题记录
2. WHEN 用户请求答题历史，THE 系统 SHALL 支持分页查询
3. WHEN 用户请求答题历史，THE 系统 SHALL 按答题时间倒序排列
4. WHEN 用户请求答题历史，THE 系统 SHALL 返回每条记录的答题模式、题目数量、正确率、获得积分、答题时间

### 需求 8: 答题统计

**用户故事:** AS 管理员，I want 查看答题统计，so that 了解答题功能的使用情况

#### 验收标准

1. WHEN 管理员请求答题统计，THE 系统 SHALL 显示总答题次数、总参与人数、总发放积分
2. WHEN 管理员请求答题统计，THE 系统 SHALL 显示每日答题次数趋势图
3. WHEN 管理员请求答题统计，THE 系统 SHALL 显示题目正确率统计
4. WHEN 管理员请求答题统计，THE 系统 SHALL 显示最受欢迎的答题时间段

## 非功能性需求

### 性能

- 单机答题响应时间应在 200ms 内
- 联机答题的题目同步延迟应不超过 500ms
- 支持至少 1000 个并发答题会话

### 可用性

- 答题界面应简洁直观，适合移动端和桌面端
- 答题倒计时应有明显的视觉提示
- 联机模式应显示其他玩家的实时状态

### 安全性

- 防止用户作弊（如查看答案后再答题）
- 防止恶意刷分行为
- 所有答题操作需要验证用户登录状态

### 可扩展性

- 题库应支持分类和标签，便于后续扩展
- 积分计算规则应可配置，便于调整

## 数据模型

### QuizQuestion (题目表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uint64 | 主键 |
| type | enum | 题型：single_choice, multiple_choice, true_false |
| content | text | 题目内容 |
| options | jsonb | 选项（JSON格式） |
| correct_answer | jsonb | 正确答案（JSON格式，支持多选） |
| source | enum | 来源：manual, import, third_party |
| external_id | string | 第三方题库ID（如果来源是third_party） |
| created_by | uint64 | 创建者管理员ID（如果来源是manual） |
| created_at | timestamp | 创建时间 |
| updated_at | timestamp | 更新时间 |
| deleted_at | timestamp | 软删除时间 |

### QuizSession (答题会话表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uint64 | 主键 |
| mode | enum | 模式：single, multiplayer |
| user_id | uint64 | 用户ID（单机模式） |
| total_questions | int | 题目总数 |
| correct_count | int | 答对数量 |
| total_score | int | 总得分 |
| total_points | decimal | 获得积分 |
| duration | int | 答题时长（秒） |
| status | enum | 状态：ongoing, completed, abandoned |
| started_at | timestamp | 开始时间 |
| completed_at | timestamp | 完成时间 |

### QuizSessionDetail (答题详情表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uint64 | 主键 |
| session_id | uint64 | 会话ID |
| question_id | uint64 | 题目ID |
| user_answer | jsonb | 用户答案 |
| is_correct | bool | 是否正确 |
| score | int | 得分 |
| time_spent | int | 答题用时（秒） |
| answered_at | timestamp | 答题时间 |

### QuizLeaderboard (排行榜表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uint64 | 主键 |
| user_id | uint64 | 用户ID |
| period | enum | 周期：daily, weekly, monthly, all_time |
| total_score | int | 总得分 |
| total_sessions | int | 答题次数 |
| correct_rate | decimal | 正确率 |
| updated_at | timestamp | 更新时间 |

### QuizConfig (答题配置表)

| 字段 | 类型 | 说明 |
|------|------|------|
| key | string | 配置键（唯一） |
| value | text | 配置值 |
| description | string | 配置描述 |
| updated_at | timestamp | 更新时间 |

## API 接口设计

### 用户端接口

1. **POST /api/v1/quiz/start** - 开始答题（指定模式）
2. **POST /api/v1/quiz/:session_id/answer** - 提交答案
3. **GET /api/v1/quiz/:session_id** - 获取答题会话详情
4. **POST /api/v1/quiz/:session_id/complete** - 完成答题会话
5. **GET /api/v1/quiz/history** - 获取答题历史
6. **GET /api/v1/quiz/leaderboard** - 获取排行榜

### 管理员接口

1. **POST /api/v1/admin/quiz/question** - 添加题目
2. **GET /api/v1/admin/quiz/question/list** - 获取题目列表
3. **PUT /api/v1/admin/quiz/question/:id** - 更新题目
4. **DELETE /api/v1/admin/quiz/question/:id** - 删除题目
5. **POST /api/v1/admin/quiz/question/import** - 批量导入题目
6. **GET /api/v1/admin/quiz/question/export** - 导出题目
7. **GET /api/v1/admin/quiz/statistics** - 获取答题统计
8. **PUT /api/v1/admin/quiz/config** - 更新答题配置
