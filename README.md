# CMD

统一汇总 DR16、VT13 与上位机控制输入，发布底盘、云台、发射和机械臂命令。

## 接口与行为

本模块使用 XRobot 1.0 的普通 C++ 模块接口，适配 LibXR 6.0 生成器。
不再接收 `HardwareContainer` 或 `ApplicationManager`，由 XRobot 按配置顺序构造。

```cpp
CMD(Mode mode = CMD::Mode::CMD_OP_CTRL,
    const char* chassis_cmd_topic_name = "chassis_cmd",
    const char* gimbal_cmd_topic_name = "gimbal_cmd",
    const char* launcher_cmd_topic_name = "launcher_cmd",
    const char* arm_cmd_topic_name = "arm_cmd");
```

- `FeedRC(source, data)` 接收指定遥控链路的数据；`FeedRC(data)` 默认使用 DR16。
- `FeedAI(data)` 接收上位机数据；`SetCtrlMode()` 或 `GetEvent()` 切换控制模式。
- 遥控活动源切换和离线回退保持原有规则，开始与丢失控制时发出事件。
- 操作员模式发布遥控命令；自动模式按在线状态选择 AI 或遥控的底盘、云台命令，
  开火需要 AI 和遥控共同允许，机械臂始终使用遥控输入。
- `ArmCMD` 包含 `x`、`y`、`z`、`rotation_x`、`rotation_y` 五个归一化增量，
  通过 `arm_cmd` Topic 发布，供机械臂模块换算成末端位姿增量。

## XRobot 配置

在模块源中加入个人仓库后，使用 `xrobot module add laba-111/CMD@same-or-dev`
添加模块请求，然后在产品配置中添加实例：

```yaml
modules:
  - module: laba-111/CMD
    id: cmd
    args:
      - mode: CMD::Mode::CMD_OP_CTRL
      - chassis_cmd_topic_name: "chassis_cmd"
      - gimbal_cmd_topic_name: "gimbal_cmd"
      - launcher_cmd_topic_name: "launcher_cmd"
      - arm_cmd_topic_name: "arm_cmd"
settings:
  monitor_sleep_ms: 1000
```

依赖 CMD 的实例应排在它之后。配置更改后运行 `xrobot format` 和 `xrobot gen`。
新增构造参数时，已有配置可先运行 `xrobot sync` 补齐默认值。
本模块无硬件依赖。
