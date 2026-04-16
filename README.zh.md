# unitree-sdk2-mcp

[English](./README.md) | **中文**

ROSClaw MCP Server for **Unitree 机器人** via DDS 协议。

支持: G1, Go2, Go2w, H1, H2, B2, B2w, A2, R1

Part of the [ROSClaw](https://github.com/ros-claw) Embodied Intelligence Operating System.

## 概述

该 MCP server 使 LLM agents (Claude, GPT-4 等) 能够通过 Model Context Protocol 控制任意 Unitree 机器人。它使用 DDS (Data Distribution Service) 协议与机器人通信 —— 与 Unitree 官方 SDK 使用的协议相同。

```
LLM Agent  ──MCP──►  unitree-sdk2-mcp  ──DDS──►  Unitree Robot
```

## 支持的机器人

| 型号 | 类型 | 自由度 | 功能特性 | 重量 | 续航 |
|-------|------|-----|----------|--------|---------|
| **G1** | 人形 | 23 | 行走, 手臂控制 | ~35kg | 2.5h |
| **Go2** | 四足 | 12 | 小跑, 疾驰 | ~15kg | 2.0h |
| **Go2w** | 四足+轮 | 12 | 混合运动 | ~16kg | 2.0h |
| **H1** | 人形 | 20 | 行走, 手臂控制 | ~47kg | 2.0h |
| **H2** | 人形 | 20 | 行走, 手臂控制 | ~45kg | 2.5h |
| **B2** | 四足 | 12 | 工业级, 重载 | ~60kg | 4.0h |
| **B2w** | 四足+轮 | 12 | 工业+轮式 | ~62kg | 4.0h |
| **A2** | 四足 | 12 | 敏捷, 教育 | ~15kg | 1.5h |
| **R1** | 轮式 | 4 | 导航, 配送 | ~25kg | 3.0h |

## SDK 信息

| 属性 | 值 |
|----------|-------|
| **SDK 名称** | unitree_sdk2 |
| **SDK 版本** | 2.1.0+ |
| **协议** | DDS (Data Distribution Service) |
| **源码仓库** | [github.com/unitreerobotics/unitree_sdk2](https://github.com/unitreerobotics/unitree_sdk2) |
| **文档** | [support.unitree.com](https://support.unitree.com/home/zh/developer) |
| **许可证** | BSD-3-Clause |
| **生成日期** | 2026-04-15 |

## 功能特性

- **多机器人支持**: 使用统一接口控制任意 Unitree 机器人
- **动态配置**: 自动检测机器人类型并加载配置
- **高级动作**: 站立、坐下、行走、挥手 (因机器人而异)
- **安全保护**: 每个命令执行前强制执行关节限制
- **实时状态**: 电池、模式、关节位置、IMU 100Hz 更新
- **DDS 协议**: 兼容 CycloneDDS / FastDDS
- **异步设计**: 非阻塞 MCP 工具，后台状态线程

## 安装

```bash
# Clone
git clone https://github.com/ros-claw/unitree-sdk2-mcp.git
cd unitree-sdk2-mcp

# Install with uv (recommended)
uv venv --python python3.10
source .venv/bin/activate
uv pip install -e .

# Or with pip
pip install -e .
```

### 依赖

```bash
# Required: Unitree SDK2
# Download from: https://github.com/unitreerobotics/unitree_sdk2

# DDS Middleware (choose one)
pip install cyclonedds  # or fastdds
```

## 快速开始

### 作为 MCP Server 运行

```bash
# stdio transport (for Claude Desktop / MCP clients)
python src/unitree_sdk2_mcp_server.py

# Or using the installed entry point
unitree-sdk2-mcp

# Legacy alias (backward compatible)
rosclaw-g1-mcp
```

### 连接机器人

```python
# First, list available robots
list_robots()

# Get robot info
get_robot_info(robot_id="g1")

# Connect
connect_robot(robot_id="g1", domain_id=0)

# Or connect to Go2
connect_robot(robot_id="go2", domain_id=0)
```

### Claude Desktop 配置

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "unitree-sdk2": {
      "command": "python",
      "args": ["/path/to/unitree-sdk2-mcp/src/unitree_sdk2_mcp_server.py"],
      "transportType": "stdio",
      "description": "Unitree Multi-Robot via DDS",
      "sdk_version": "2.1.0",
      "sdk_source": "https://github.com/unitreerobotics/unitree_sdk2"
    }
  }
}
```

### MCP Inspector (测试)

```bash
mcp dev src/unitree_sdk2_mcp_server.py
```

## 可用工具

### 多机器人工具

| 工具 | 描述 |
|------|-------------|
| `list_robots` | 列出所有支持的 Unitree 机器人 |
| `get_robot_info` | 获取指定机器人的配置信息 |
| `connect_robot` | 连接机器人 (指定 robot_id) |
| `disconnect_robot` | 断开机器人连接 |

### 控制工具

| 工具 | 描述 | 支持的机器人 |
|------|-------------|------------------|
| `stand_up` | 命令机器人站立 | G1, H1, H2, Go2, B2, A2 |
| `sit_down` | 命令机器人坐下/停止 | 全部 |
| `walk_with_velocity` | 以线速度/角速度行走 | G1, Go2, H1, H2, B2, A2 |
| `move_joint` | 移动单个关节到目标角度 | 所有带关节的机器人 |
| `move_joints` | 同时移动多个关节 | 所有带关节的机器人 |
| `stop_movement` | 停止并保持当前位置 | 全部 |
| `emergency_stop` | 紧急停止 (谨慎使用!) | 全部 |
| `wave_hand` | 执行挥手动作 | G1, H1, H2 |

## 可用资源

| 资源 | 描述 |
|----------|-------------|
| `unitree://{robot_id}/status` | 电池、模式、关节位置 |
| `unitree://{robot_id}/joints` | 所有关节的关节限制 |
| `unitree://connections` | 所有活动的 DDS 连接 |

## 机器人配置

### 人形机器人 (G1, H1, H2)

**关节结构:**
```
左腿:   left_hip_yaw, left_hip_roll, left_hip_pitch, left_knee, left_ankle
右腿:  right_hip_yaw, right_hip_roll, right_hip_pitch, right_knee, right_ankle
腰部:      waist_yaw, waist_roll, waist_pitch (G1: 3 DOF, H1/H2: 1 DOF)
左臂:   left_shoulder_pitch, left_shoulder_roll, left_shoulder_yaw, left_elbow
右臂:  right_shoulder_pitch, right_shoulder_roll, right_shoulder_yaw, right_elbow
```

**G1 关节限制:**

| 关节 | 范围 (rad) |
|-------|-------------|
| left_hip_yaw | [-2.35, 2.35] |
| left_hip_roll | [-0.78, 0.78] |
| left_hip_pitch | [-2.5, 2.5] |
| left_knee | [-0.5, 2.5] |
| left_ankle | [-1.0, 1.0] |
| right_hip_yaw | [-2.35, 2.35] |
| right_hip_roll | [-0.78, 0.78] |
| right_hip_pitch | [-2.5, 2.5] |
| right_knee | [-0.5, 2.5] |
| right_ankle | [-1.0, 1.0] |
| waist_yaw | [-2.5, 2.5] |
| waist_roll | [-0.5, 0.5] |
| waist_pitch | [-1.0, 1.0] |
| left_shoulder_pitch | [-3.14, 3.14] |
| left_shoulder_roll | [-0.5, 3.5] |
| left_shoulder_yaw | [-2.0, 2.0] |
| left_elbow | [-1.5, 2.0] |
| right_shoulder_pitch | [-3.14, 3.14] |
| right_shoulder_roll | [-3.5, 0.5] |
| right_shoulder_yaw | [-2.0, 2.0] |
| right_elbow | [-1.5, 2.0] |

### 四足机器人 (Go2, Go2w, B2, B2w, A2)

**关节结构:**
```
前左:  front_left_hip, front_left_thigh, front_left_calf
前右: front_right_hip, front_right_thigh, front_right_calf
后左:   rear_left_hip, rear_left_thigh, rear_left_calf
后右:  rear_right_hip, rear_right_thigh, rear_right_calf
```

**Go2 关节限制:**

| 关节 | 范围 (rad) |
|-------|-------------|
| front_left_hip | [-0.87, 0.87] |
| front_left_thigh | [-0.52, 2.97] |
| front_left_calf | [-2.76, -0.52] |
| ... (其他腿类似) |

## 安全信息

**警告:** 该 MCP server 控制物理机器人。不当使用可能导致:
- 设备损坏
- 人身伤害
- 财产损失

### 安全特性

| 特性 | 描述 |
|---------|-------------|
| **关节限制** | 所有命令都根据机器人类型的硬件限制进行验证 |
| **速度限制** | 根据机器人配置强制执行 |
| **紧急停止** | `emergency_stop()` 立即禁用电机 |

### 安全级别

| 级别 | 颜色 | 描述 |
|-------|-------|-------------|
| **CRITICAL** | 🔴 | 立即危险 (摔倒、碰撞) |
| **HIGH** | 🟠 | 潜在硬件损坏 |
| **MEDIUM** | 🟡 | 操作问题 |
| **LOW** | 🟢 | 信息提示 |

### 紧急程序

1. **立即停止**: 使用 `emergency_stop(robot_id)` 或按下物理紧急停止按钮
2. **断电**: 如果安全，断开电池
3. **检查状态**: 使用 `unitree://{robot_id}/status` 资源

## 错误处理

### 错误代码

| 代码 | 名称 | 严重级别 | 描述 |
|------|------|----------|-------------|
| -1 | CONNECTION_FAILED | 🟠 error | 无法连接到 DDS domain |
| -2 | TIMEOUT | 🟠 error | 操作超时 |
| -3 | INVALID_PARAMETER | 🟠 error | 无效的关节名称或值 |
| -4 | SAFETY_VIOLATION | 🔴 critical | 命令超出关节限制 |
| -5 | NOT_INITIALIZED | 🟠 error | 未连接到机器人 |
| -6 | UNKNOWN_ROBOT | 🟠 error | 不支持的 robot ID |

### 故障排除

| 问题 | 可能原因 | 解决方案 |
|-------|---------------|----------|
| 连接失败 | 机器人断电 | 检查电池和电源开关 |
| 连接失败 | 错误的 DDS domain | 验证 domain_id 参数 |
| 命令被拒绝 | 关节限制超出 | 检查机器人特定的关节限制 |
| 响应缓慢 | 网络延迟 | 检查 WiFi/Ethernet 连接 |
| 未知 robot ID | robot_id 拼写错误 | 使用 `list_robots()` 查看有效 ID |

## 架构

```
unitree_sdk2_mcp_server.py
├── SDK_METADATA        — SDK 版本和源码信息
├── RobotState          — 机器人状态数据类 (通用)
├── StateBuffer         — DDS 数据的线程安全环形缓冲区
├── UnitreeDDSBridge    — DDS 通信桥接
│   ├── connect()       — 初始化 DDS participant
│   ├── _dds_listener() — 100Hz 后台线程
│   └── publish_command() — 发送 /lowcmd
├── robot_configs.py    — 所有 9 个机器人的配置
│   ├── G1_CONFIG       — G1 人形 (23 DOF)
│   ├── GO2_CONFIG      — Go2 四足 (12 DOF)
│   ├── H1_CONFIG       — H1 人形 (20 DOF)
│   ├── B2_CONFIG       — B2 工业四足
│   ├── A2_CONFIG       — A2 敏捷四足
│   └── R1_CONFIG       — R1 轮式机器人
└── MCP Tools           — FastMCP 工具定义
```

## DDS Topics

| Topic | 方向 | 描述 |
|-------|-----------|-------------|
| `/lowstate` | Subscribe | 机器人状态 (关节、IMU、电池) |
| `/lowcmd` | Publish | 机器人命令 (关节目标、模式) |
| `/sportmodestate` | Subscribe | 运动模式状态 |

## 依赖

- Python 3.10+
- `mcp[fastmcp]` — MCP 框架
- `unitree_sdk2` — Unitree DDS SDK (从硬件包单独安装)
- `cyclonedds` or `fastdds` — DDS 中间件

## 参考

- [Unitree SDK2 GitHub](https://github.com/unitreerobotics/unitree_sdk2)
- [Unitree Developer Docs](https://support.unitree.com/home/zh/developer)
- [Unitree G1 Product Page](https://www.unitree.com/products/g1)
- [Unitree Go2 Product Page](https://www.unitree.com/products/go2)
- [DDS Specification](https://www.dds-foundation.org/)
- [MCP Protocol](https://modelcontextprotocol.io/)

## 许可证

MIT License — See [LICENSE](LICENSE)

## 属于 ROSClaw 生态

- [rosclaw](https://github.com/ros-claw/rosclaw) — 核心框架（[arXiv 论文](https://arxiv.org/pdf/2604.04664)）
- [unitree-sdk2-mcp](https://github.com/ros-claw/unitree-sdk2-mcp) — Unitree 多机器人 (DDS)
- [rosclaw-ur-ros2-mcp](https://github.com/ros-claw/rosclaw-ur-ros2-mcp) — UR5 机械臂 (ROS2)
- [rosclaw-gimbal-mcp](https://github.com/ros-claw/rosclaw-gimbal-mcp) — GCU 云台 (串口)
- [rosclaw-ur-rtde-mcp](https://github.com/ros-claw/rosclaw-ur-rtde-mcp) — UR5 通过 RTDE

---

**由 ROSClaw SDK-to-MCP 转换器生成**

*SDK 版本: unitree_sdk2 2.1.0+ | 协议: DDS | 机器人: G1, Go2, Go2w, H1, H2, B2, B2w, A2, R1*
