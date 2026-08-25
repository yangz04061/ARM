# VMC ARM Use Story 拆分建议 - Jira 子任务

基于主需求：VMC ARM 防侧翻辅助控制功能开发

## 子任务 1：ARM 状态机与状态输出开发

- Title: ARM 状态机与 `VMCArmSts` 输出实现
- Objective: 实现 ARM 的 `Inactive` / `Active` 两态状态机及状态输出。
- Scope:
  - 定义状态机框架
  - 实现 `VMCArmSts` 输出
  - 支持进入 Active 和退出 Inactive 的状态切换
  - 预留故障降级和执行器仲裁接入点
- Deliverable:
  - 状态机代码实现
  - 状态切换接口
  - 基础单元测试或模块测试
- Done Criteria:
  - `VMCArmSts` 可按条件稳定输出 `0/1`
  - 状态切换无死锁、无非法跳转

## 子任务 2：ARM 激活/退出判定逻辑开发

- Title: ARM 激活与退出条件判定实现
- Objective: 根据车辆动态和风险指标实现 ARM 激活/退出逻辑。
- Scope:
  - 接入 `Ax`、`Ay`、`VehSpd`、`StrWhAng`、`StrWhAngRate`、`YawRate`、`ROI`
  - 实现激活判定条件
  - 实现退出判定条件
  - 所有阈值支持标定
- Deliverable:
  - 激活/退出判定模块
  - 标定参数接口
- Done Criteria:
  - 满足激活条件时可触发 Active
  - 任一退出条件满足时可立即退出
  - 阈值参数可配置

## 子任务 3：关键信号有效性与故障降级处理开发

- Title: ARM 关键信号监控与故障降级实现
- Objective: 实现 ARM 所需关键输入的有效性检查，以及故障降级后的禁用/退出逻辑。
- Scope:
  - 监控关键输入信号有效性
  - 对接 Fault Management 故障状态
  - 失效时禁止激活
  - Active 中发生失效时立即退出并释放请求
- Deliverable:
  - 信号监控逻辑
  - 故障降级处理逻辑
- Done Criteria:
  - 关键信号失效时无法进入 Active
  - Active 中故障触发后可立即退出

## 子任务 4：高优先级功能仲裁与执行器可用性管理开发

- Title: 高优先级功能仲裁与执行器可用性管理实现
- Objective: 实现高优先级功能抢占处理，以及 ARM 执行器可用性判断与屏蔽。
- Scope:
  - 接入高优先级功能状态
  - 实现功能抢占仲裁
  - 实现执行器可用/不可用判断
  - 执行器故障、握手失败、`Not controllable` 时执行屏蔽
  - 无执行器可用时触发 ARM 退出
- Deliverable:
  - 仲裁逻辑模块
  - 执行器状态管理模块
- Done Criteria:
  - 高优先级功能激活时 ARM 不得激活或需退出
  - 不可用执行器不得继续参与控制
  - 无可用执行器时 ARM 退出

## 子任务 5：RWS 接口与控制请求开发

- Title: RWS ARM 控制请求与反馈处理实现
- Objective: 实现 RWS 参与 ARM 时的请求输出和反馈处理。
- Scope:
  - 接入 RWS 使能条件
  - 接入制动侧横摆力矩大小/方向可用性判定
  - 输出后轮目标转角、请求标志、有效标志
  - 监控 RWS 反馈状态
- Deliverable:
  - RWS ARM 控制接口实现
  - RWS 状态反馈处理逻辑
- Done Criteria:
  - 仅在前提满足时允许 RWS 参与 ARM
  - RWS 请求和反馈处理链路完整

## 子任务 6：CDC 接口与控制请求开发

- Title: CDC ARM 控制请求实现
- Objective: 实现 ARM 对 CDC 的阻尼电流上下限请求。
- Scope:
  - 输出 CDC 电流上下限请求
  - 保持 CDC 原有控制架构，仅限制允许范围
  - 处理 CDC 控制状态反馈
- Deliverable:
  - CDC 请求发送逻辑
  - CDC 状态反馈处理逻辑
- Done Criteria:
  - CDC 请求值可正确输出
  - CDC 不可用或反馈异常时可被屏蔽

## 子任务 7：ECAS 与主动悬架接口开发

- Title: ECAS 与主动悬架 ARM 控制请求实现
- Objective: 实现 ARM 对 ECAS 和主动悬架的控制请求与反馈处理。
- Scope:
  - 输出 ECAS 四轮高度请求和刚度请求
  - 输出主动悬架 offset / override 模式及四轮目标阻尼力
  - 处理 ECAS、主动悬架握手反馈
  - 处理被 ADAS 等高优先级功能抢占的情况
- Deliverable:
  - ECAS 控制接口实现
  - 主动悬架控制接口实现
  - 握手反馈与抢占处理逻辑
- Done Criteria:
  - ECAS 和主动悬架请求可按状态机正确输出
  - 被抢占、故障或握手失败时可正确退出或屏蔽

## 子任务 8：Variant 配置、联调与验收支持开发

- Title: ARM Variant 配置与联调验收支持实现
- Objective: 实现不同车型 Variant 的执行器配置裁剪，并完成联调支持。
- Scope:
  - 实现 Variant 执行器配置矩阵接入
  - 按 Variant 决定可用执行器集合
  - 实现退出后请求释放统一处理
  - 支持联调、标定和验收场景
- Deliverable:
  - Variant 配置逻辑
  - 请求释放统一处理逻辑
  - 联调支持说明
- Done Criteria:
  - 不同 Variant 仅启用允许的执行器
  - 退出后所有 ARM 请求统一释放
  - 支持标定和验收使用

## 推荐执行顺序

1. 子任务 1：ARM 状态机与状态输出开发
2. 子任务 2：ARM 激活/退出判定逻辑开发
3. 子任务 3：关键信号有效性与故障降级处理开发
4. 子任务 4：高优先级功能仲裁与执行器可用性管理开发
5. 子任务 5：RWS 接口与控制请求开发
6. 子任务 6：CDC 接口与控制请求开发
7. 子任务 7：ECAS 与主动悬架接口开发
8. 子任务 8：Variant 配置、联调与验收支持开发

## 可直接复制到 Jira 的简版子任务标题

- ARM 状态机与 `VMCArmSts` 输出实现
- ARM 激活与退出条件判定实现
- ARM 关键信号监控与故障降级实现
- 高优先级功能仲裁与执行器可用性管理实现
- RWS ARM 控制请求与反馈处理实现
- CDC ARM 控制请求实现
- ECAS 与主动悬架 ARM 控制请求实现
- ARM Variant 配置与联调验收支持实现