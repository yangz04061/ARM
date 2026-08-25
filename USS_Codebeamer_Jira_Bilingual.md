# VMC ARM Use Story - Codebeamer/Jira Bilingual Version

## 1. Basic Information / 基本信息

- Title / 标题: VMC ARM Active Roll Mitigation Function Development / VMC ARM 防侧翻辅助控制功能开发
- Type / 类型: Use Story
- Domain / 领域: VMC / Chassis Coordination / ARM
- Priority / 优先级: High / 高
- Status / 状态: Draft / 草稿
- Source / 来源: SGMW_VMC ARM_防侧翻功能规范_master

## 2. User Story / 用户故事

As VMC ARM software,
I need to determine ARM activation, deactivation, and coordinated actuator control based on vehicle dynamics, rollover risk indicators, and actuator availability when rollover risk exists and no higher-priority function is active,
so that the vehicle roll stability margin can be improved and rollover risk can be reduced before or during deep brake ARM intervention.

作为 VMC ARM 软件，
我需要在车辆存在侧翻风险、且未被更高优先级功能抢占时，基于车辆动态、风险指标和执行器可用性，完成 ARM 的激活、退出和执行器协调控制，
以便在制动 ARM 深度介入前或介入过程中提升车辆侧倾稳定裕度、降低侧翻风险。

## 3. Business Value / 业务价值

- Enable earlier coordinated intervention before deep brake ARM intervention to improve vehicle roll stability margin.
- Support rollover risk mitigation by coordinating suspension-related actuators and rear wheel steering.
- Reduce brake ARM workload and improve overall vehicle body attitude stability.

- 在制动 ARM 深度介入前提前进行协同控制，提升车辆侧倾稳定裕度。
- 通过协调悬架和后轮转向等执行器，降低车辆进入极限侧翻工况的风险。
- 减轻制动 ARM 的工作压力，改善整车姿态稳定性。

## 4. Scope / 范围

### In Scope / 范围内

- ARM state management: Inactive / Active
- ARM activation and deactivation judgment
- Key input signal validity monitoring
- Higher-priority function arbitration
- Actuator availability judgment
- ARM control request output to available actuators
- Actuator handshake and feedback monitoring
- Fault degradation and request release logic
- Variant-based actuator configuration tailoring

- ARM 状态机管理：Inactive / Active
- ARM 激活和退出判定
- 关键输入信号有效性监控
- 高优先级功能仲裁
- 执行器可用性判断
- 对可用执行器输出 ARM 控制请求
- 执行器握手和反馈监控
- 故障降级和退出释放逻辑
- Variant 执行器配置裁剪

### Out of Scope / 范围外

- Brake ARM internal algorithm design
- Brake force or drive torque distribution logic
- Internal closed-loop control design of RWS, CDC, ECAS, and active suspension
- HMI display strategy
- Detailed legal and regulatory requirements

- 制动 ARM 内部算法设计
- 制动力或驱动力分配逻辑
- RWS、CDC、ECAS、主动悬架内部闭环控制设计
- HMI 显示策略
- 法规需求展开

## 5. Preconditions / 前置条件

- VMC ARM function is configured and enabled.
- ARM-related key input signals are available and valid.
- At least one ARM-related actuator is available.
- No ARM-related fault degradation is present.
- No higher-priority function is active, such as tank turn, compass turn, drift mode, or brake steering backup.

- VMC ARM 功能已配置使能。
- ARM 相关关键输入信号可接收且有效。
- 至少 1 个 ARM 相关执行器可用。
- 无 ARM 相关故障降级。
- 无高优先级功能激活，例如：坦克掉头、圆规掉头、漂移、制动转向备份功能。

## 6. Functional Description / 功能描述

### 6.1 State Management / 状态管理

- The software shall output `VMCArmSts`: `0 = Inactive`, `1 = Active`.
- The software shall enter Active when activation conditions are met.
- The software shall return to Inactive and release all ARM requests when any deactivation condition is met.

- 软件应输出 `VMCArmSts`：`0 = Inactive`，`1 = Active`。
- 满足激活条件时进入 Active。
- 满足任一退出条件时退出到 Inactive，并释放所有 ARM 请求。

### 6.2 Activation Logic / 激活逻辑

The software shall enter Active only when all of the following conditions are met:

- No ARM safety monitoring degradation is present.
- No higher-priority function is active.
- At least one ARM-related actuator is available.
- Vehicle state satisfies the activation criteria:
  - `Ax > Cal` or `VehSpd > Cal`
  - `Ay > Cal` or `StrWhAng > Cal` or `StrWhAngRate > Cal`
  - `YawRate > Cal`
  - `ROI > Cal`

仅在以下条件同时满足时允许进入 Active：

- ARM 安全监控无故障降级。
- 无高优先级功能激活。
- 至少 1 个 ARM 相关执行器可用。
- 车辆状态满足激活判定：
  - `Ax > Cal` 或 `VehSpd > Cal`
  - `Ay > Cal` 或 `StrWhAng > Cal` 或 `StrWhAngRate > Cal`
  - `YawRate > Cal`
  - `ROI > Cal`

### 6.3 Deactivation Logic / 退出逻辑

The software shall exit Active when any of the following conditions occurs:

- ARM safety monitoring degradation or key signal invalidity occurs.
- A higher-priority function becomes active.
- All ARM-related actuators become unavailable.
- Vehicle state satisfies the deactivation criteria:
  - `Ax < Cal`
  - `VehSpd < Cal`
  - `Ay < Cal`
  - `StrWhAng < Cal`
  - `YawRate < Cal`
  - `ROI < Cal`

出现以下任一情况时应退出 Active：

- ARM 安全监控故障降级或关键信号失效。
- 高优先级功能激活。
- 所有 ARM 相关执行器均不可用。
- 车辆状态满足退出判定：
  - `Ax < Cal`
  - `VehSpd < Cal`
  - `Ay < Cal`
  - `StrWhAng < Cal`
  - `YawRate < Cal`
  - `ROI < Cal`

### 6.4 Input Monitoring / 输入监控

The software shall cyclically monitor the following information:

- Vehicle dynamics: `Ax`, `Ay`, `VehSpd`, `StrWhAng`, `StrWhAngRate`, `YawRate`, `ROI`
- Arbitration status: higher-priority function status
- Safety status: key signal validity and ARM-related fault status
- Actuator status: availability, controllability, and handshake feedback
- Brake-side information: availability of yaw torque magnitude and direction related information

软件应周期监控以下信息：

- 车辆动态：`Ax`、`Ay`、`VehSpd`、`StrWhAng`、`StrWhAngRate`、`YawRate`、`ROI`
- 功能仲裁状态：高优先级功能状态
- 安全状态：关键信号有效性、ARM 相关故障状态
- 执行器状态：可用性、可控性、握手反馈
- 制动侧信息：横摆力矩大小和方向相关可用性信息

### 6.5 Actuator Control Boundaries / 执行器控制边界

- RWS: Allowed to participate in ARM only when the brake side can provide accurate yaw torque magnitude and direction information; output rear target steering angle, request flag, and validity flag.
- CDC: Output damping current upper and lower limit requests to constrain the allowed CDC control range without replacing CDC internal control.
- ECAS: Output four-corner height and stiffness requests; after ECAS accepts the VMC request, VMC exclusively controls target height and stiffness.
- Active suspension: Output offset or override mode and four-corner target damping force; in override mode VMC has higher control priority.

- RWS：仅在制动侧可提供准确横摆力矩大小和方向信息时允许参与 ARM 控制；输出目标后轮转角、请求标志和有效标志。
- CDC：输出阻尼电流上下限请求，用于限制 CDC 允许控制范围，不替代 CDC 自身控制。
- ECAS：输出四轮高度请求和刚度请求；当 ECAS 接受 VMC 请求后，由 VMC 独占目标高度和刚度控制。
- 主动悬架：输出 offset 或 override 模式及四轮目标阻尼力；override 模式下 VMC 拥有更高控制优先级。

### 6.6 Arbitration and Degradation / 仲裁与降级

- If an actuator returns `Not controllable`, handshake failure, or actuator fault, the software shall stop sending effective ARM control requests to that actuator.
- If no available actuator remains, the software shall exit Active.
- Higher-priority functions such as ADAS crosswind compensation may preempt relevant suspension actuators; after preemption, the software shall re-arbitrate or exit.
- The software shall interface with Fault Management so that unified fault management can trigger ARM degradation and exit.

- 若执行器返回 `Not controllable`、握手失败或执行器故障，则不得继续对该执行器输出有效 ARM 控制请求。
- 若因此导致无可用执行器剩余，软件应退出 Active。
- ADAS 侧风抗扰等高优先级功能可抢占相关悬架执行器；被抢占后，软件应重新仲裁或退出。
- 软件应与 Fault Management 对接，由统一故障管理触发 ARM 降级和退出。

## 7. Acceptance Criteria / 验收标准

### AC1 State Activation / 状态机激活

- Given VMC ARM is enabled, fault-free, no higher-priority function is active, and at least one actuator is available
- When vehicle dynamics and ROI satisfy the ARM activation conditions
- Then `VMCArmSts` shall enter `Active`

- Given VMC ARM 已使能、无故障、无高优先级功能、至少 1 个执行器可用
- When 车辆动态和 ROI 满足 ARM 激活条件
- Then `VMCArmSts` 应进入 `Active`

### AC2 State Deactivation / 状态机退出

- Given VMC ARM is in `Active`
- When any deactivation condition is met
- Then `VMCArmSts` shall immediately return to `Inactive`

- Given VMC ARM 处于 `Active`
- When 任一退出条件满足
- Then `VMCArmSts` 应立即回到 `Inactive`

### AC3 Request Release / 请求释放

- Given VMC ARM exits from `Active`
- When deactivation occurs
- Then all ARM actuator requests shall be cleared, invalidated, or released

- Given VMC ARM 从 `Active` 退出
- When 退出动作发生
- Then 所有 ARM 执行器请求应被清零、失效或释放

### AC4 Actuator Fault Blocking / 执行器故障屏蔽

- Given an actuator fault, handshake failure, or `Not controllable` status occurs
- When the software performs ARM control arbitration
- Then that actuator shall not continue participating in ARM control

- Given 某个执行器故障、握手失败或返回 `Not controllable`
- When 软件进行 ARM 控制仲裁
- Then 该执行器不得继续参与 ARM 控制

### AC5 Exit on No Available Actuator / 无执行器退出

- Given VMC ARM is in `Active`
- When all ARM-related actuators become unavailable
- Then ARM shall exit to `Inactive`

- Given VMC ARM 处于 `Active`
- When 所有 ARM 相关执行器均不可用
- Then ARM 应退出到 `Inactive`

### AC6 Higher-Priority Preemption / 高优先级功能抢占

- Given VMC ARM is in `Active`
- When a higher-priority function is activated or a related actuator is preempted
- Then ARM shall re-arbitrate actuators and exit if control can no longer continue

- Given VMC ARM 处于 `Active`
- When 高优先级功能激活或相关执行器被抢占
- Then ARM 应重新仲裁执行器；若无法继续控制则退出

### AC7 Variant Configuration / Variant 配置

- Given different vehicle variants
- When the software loads the Variant configuration
- Then ARM shall use only the actuator set allowed for that Variant

- Given 不同车型 Variant
- When 软件加载 Variant 配置
- Then ARM 仅使用该 Variant 允许的执行器集合

### AC8 Calibration Support / 标定支持

- Given software integration is completed
- When calibration and acceptance activities are performed
- Then ARM activation/deactivation thresholds and actuator enable switches shall support configuration or calibration

- Given 软件集成完成
- When 进行标定与验收
- Then ARM 激活/退出阈值及执行器开关应支持配置或标定

## 8. Dependencies / Interfaces / 依赖与接口

- Brake system: provides ARM-related status and yaw torque magnitude/direction availability information
- RWS: rear steering control requests and feedback
- CDC / SUCU: damping current requests and feedback
- ECAS / SUCU: height and stiffness requests and feedback
- Active suspension / SUCU: damping force mode, target requests, and handshake feedback
- Fault Management: fault degradation and exit trigger
- Variant configuration: actuator enable scope definition

- 制动系统：提供 ARM 相关状态和横摆力矩大小/方向可用性信息
- RWS：后轮转向控制请求与反馈
- CDC / SUCU：阻尼电流请求与反馈
- ECAS / SUCU：高度、刚度请求与反馈
- 主动悬架 / SUCU：阻尼力模式与目标值请求、握手反馈
- Fault Management：故障降级和退出触发
- Variant 配置：执行器使能范围定义

## 9. Implementation Notes / 实现说明

- Threshold values `Cal` are not defined in the customer document and shall be completed by calibration documents or system requirements.
- The definition, source, and ownership of `ROI` calculation shall be clarified.
- Whether RWS participates in ARM depends on the accuracy of brake-side yaw torque information.
- The request release strategy shall remain consistent with the interface definition, including explicit clear values and invalid values.

- 所有阈值 `Cal` 当前未在客户文档中定值，应通过标定文档或系统需求补齐。
- `ROI` 的定义、来源和计算责任需明确。
- RWS 是否参与 ARM 受制动侧横摆力矩信息准确性约束。
- 执行器请求释放策略需与接口定义保持一致，明确清零值和无效值。

## 10. Open Points / 待定项

- Specific values of activation/deactivation thresholds
- Definition and source of `ROI`
- Actuator configuration matrix for each Variant
- Signals and criteria used to determine whether brake-side yaw torque magnitude/direction is accurate and available
- Acceptance scenarios and quantitative performance targets

- 激活/退出阈值具体数值
- `ROI` 的定义和来源
- 各 Variant 对应的执行器配置矩阵
- 制动侧“横摆力矩大小/方向准确可用”的判定信号和判据
- 验收工况和量化性能指标

## 11. Short Jira Description / Jira 描述区简版

As VMC ARM software, I need to determine ARM activation, deactivation, and coordinated actuator control based on vehicle dynamics, rollover risk indicators, and actuator availability when rollover risk exists and no higher-priority function is active, so that the vehicle roll stability margin can be improved and rollover risk can be reduced before or during deep brake ARM intervention.

Scope includes ARM state machine, activation/deactivation judgment, input monitoring, actuator arbitration, RWS/CDC/ECAS/active suspension request output, handshake feedback monitoring, fault degradation, and request release. Brake ARM internal algorithms, actuator internal closed-loop control, HMI, and legal/regulatory details are out of scope.

Acceptance criteria: enter Active when activation conditions are met; return to Inactive immediately when any deactivation condition occurs; release all ARM requests after exit; faulty, failed-handshake, or preempted actuators shall not continue participating in control; support Variant-based actuator configuration tailoring; support configuration or calibration of activation/deactivation thresholds and actuator enable switches.

作为 VMC ARM 软件，我需要在车辆存在侧翻风险、且未被更高优先级功能抢占时，基于车辆动态、风险指标和执行器可用性，完成 ARM 的激活、退出和执行器协调控制，以便在制动 ARM 深度介入前或介入过程中提升车辆侧倾稳定裕度、降低侧翻风险。

范围包括 ARM 状态机、激活/退出判定、输入监控、执行器仲裁、RWS/CDC/ECAS/主动悬架请求输出、握手反馈监控、故障降级与请求释放。制动 ARM 内部算法、执行器内部闭环控制、HMI 和法规需求不在本条范围内。

验收标准：满足激活条件时进入 Active；任一退出条件出现时立即退出到 Inactive；退出后所有 ARM 请求释放；故障、握手失败或被抢占的执行器不得继续参与控制；支持 Variant 执行器配置裁剪；激活/退出阈值和执行器开关支持配置或标定。