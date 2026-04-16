# unitree-sdk2-mcp

🌐 **English** | [中文](./README.zh.md)

ROSClaw MCP Server for **Unitree Robots** via DDS protocol.

Supports: G1, Go2, Go2w, H1, H2, B2, B2w, A2, R1

Part of the [ROSClaw](https://github.com/ros-claw) Embodied Intelligence Operating System.

## Overview

This MCP server enables LLM agents (Claude, GPT-4, etc.) to control any Unitree robot through the Model Context Protocol. It communicates with the robot using DDS (Data Distribution Service) — the same protocol used by Unitree's official SDK.

```
LLM Agent  ──MCP──►  unitree-sdk2-mcp  ──DDS──►  Unitree Robot
```

## Supported Robots

| Model | Type | DOF | Features | Weight | Battery |
|-------|------|-----|----------|--------|---------|
| **G1** | Humanoid | 23 | Walking, Arm Control | ~35kg | 2.5h |
| **Go2** | Quadruped | 12 | Trotting, Galloping | ~15kg | 2.0h |
| **Go2w** | Quadruped+Wheels | 12 | Hybrid Locomotion | ~16kg | 2.0h |
| **H1** | Humanoid | 20 | Walking, Arm Control | ~47kg | 2.0h |
| **H2** | Humanoid | 20 | Walking, Arm Control | ~45kg | 2.5h |
| **B2** | Quadruped | 12 | Industrial, Heavy Payload | ~60kg | 4.0h |
| **B2w** | Quadruped+Wheels | 12 | Industrial+Wheels | ~62kg | 4.0h |
| **A2** | Quadruped | 12 | Agile, Education | ~15kg | 1.5h |
| **R1** | Wheeled | 4 | Navigation, Delivery | ~25kg | 3.0h |

## SDK Information

| Property | Value |
|----------|-------|
| **SDK Name** | unitree_sdk2 |
| **SDK Version** | 2.1.0+ |
| **Protocol** | DDS (Data Distribution Service) |
| **Source Repository** | [github.com/unitreerobotics/unitree_sdk2](https://github.com/unitreerobotics/unitree_sdk2) |
| **Documentation** | [support.unitree.com](https://support.unitree.com/home/zh/developer) |
| **License** | BSD-3-Clause |
| **Generated** | 2026-04-15 |

## Features

- **Multi-Robot Support**: Control any Unitree robot with unified interface
- **Dynamic Configuration**: Automatic robot type detection and configuration
- **High-Level Actions**: Stand, sit, walk, wave (robot-dependent)
- **Safety Guards**: Joint limits enforced before every command
- **Real-time State**: Battery, mode, joint positions, IMU at 100Hz
- **DDS Protocol**: CycloneDDS / FastDDS compatible
- **Async Design**: Non-blocking MCP tools with background state thread

## Installation

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

### Dependencies

```bash
# Required: Unitree SDK2
# Download from: https://github.com/unitreerobotics/unitree_sdk2

# DDS Middleware (choose one)
pip install cyclonedds  # or fastdds
```

## Quick Start

### Run as MCP Server

```bash
# stdio transport (for Claude Desktop / MCP clients)
python src/unitree_sdk2_mcp_server.py

# Or using the installed entry point
unitree-sdk2-mcp

# Legacy alias (backward compatible)
rosclaw-g1-mcp
```

### Connect to a Robot

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

### Claude Desktop Configuration

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

### MCP Inspector (Testing)

```bash
mcp dev src/unitree_sdk2_mcp_server.py
```

## Available Tools

### Multi-Robot Tools

| Tool | Description |
|------|-------------|
| `list_robots` | List all supported Unitree robots |
| `get_robot_info` | Get configuration for specific robot |
| `connect_robot` | Connect to a robot (specify robot_id) |
| `disconnect_robot` | Disconnect from a robot |

### Control Tools

| Tool | Description | Supported Robots |
|------|-------------|------------------|
| `stand_up` | Command robot to stand up | G1, H1, H2, Go2, B2, A2 |
| `sit_down` | Command robot to sit down/stop | All |
| `walk_with_velocity` | Walk with linear/angular velocity | G1, Go2, H1, H2, B2, A2 |
| `move_joint` | Move a single joint to target angle | All with joints |
| `move_joints` | Move multiple joints simultaneously | All with joints |
| `stop_movement` | Stop and hold current position | All |
| `emergency_stop` | Emergency halt (use with caution!) | All |
| `wave_hand` | Perform waving gesture | G1, H1, H2 |

## Available Resources

| Resource | Description |
|----------|-------------|
| `unitree://{robot_id}/status` | Battery, mode, joint positions |
| `unitree://{robot_id}/joints` | Joint limits for all joints |
| `unitree://connections` | All active DDS connections |

## Robot Configuration

### Humanoid Robots (G1, H1, H2)

**Joint Structure:**
```
Left Leg:   left_hip_yaw, left_hip_roll, left_hip_pitch, left_knee, left_ankle
Right Leg:  right_hip_yaw, right_hip_roll, right_hip_pitch, right_knee, right_ankle
Waist:      waist_yaw, waist_roll, waist_pitch (G1: 3 DOF, H1/H2: 1 DOF)
Left Arm:   left_shoulder_pitch, left_shoulder_roll, left_shoulder_yaw, left_elbow
Right Arm:  right_shoulder_pitch, right_shoulder_roll, right_shoulder_yaw, right_elbow
```

**G1 Joint Limits:**

| Joint | Range (rad) |
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

### Quadruped Robots (Go2, Go2w, B2, B2w, A2)

**Joint Structure:**
```
Front Left:  front_left_hip, front_left_thigh, front_left_calf
Front Right: front_right_hip, front_right_thigh, front_right_calf
Rear Left:   rear_left_hip, rear_left_thigh, rear_left_calf
Rear Right:  rear_right_hip, rear_right_thigh, rear_right_calf
```

**Go2 Joint Limits:**

| Joint | Range (rad) |
|-------|-------------|
| front_left_hip | [-0.87, 0.87] |
| front_left_thigh | [-0.52, 2.97] |
| front_left_calf | [-2.76, -0.52] |
| ... (similar for other legs) |

## Safety Information

**WARNING:** This MCP server controls physical robots. Improper use can cause:
- Equipment damage
- Personal injury
- Property damage

### Safety Features

| Feature | Description |
|---------|-------------|
| **Joint Limits** | All commands validated against hardware limits per robot type |
| **Velocity Limits** | Enforced per robot configuration |
| **Emergency Stop** | `emergency_stop()` disables motors immediately |

### Safety Levels

| Level | Color | Description |
|-------|-------|-------------|
| **CRITICAL** | 🔴 | Immediate danger (falling, collision) |
| **HIGH** | 🟠 | Potential hardware damage |
| **MEDIUM** | 🟡 | Operational issue |
| **LOW** | 🟢 | Informational |

### Emergency Procedures

1. **Immediate Stop**: Use `emergency_stop(robot_id)` or press physical E-stop
2. **Power Off**: Disconnect battery if safe
3. **Check Status**: Use `unitree://{robot_id}/status` resource

## Error Handling

### Error Codes

| Code | Name | Severity | Description |
|------|------|----------|-------------|
| -1 | CONNECTION_FAILED | 🟠 error | Failed to connect to DDS domain |
| -2 | TIMEOUT | 🟠 error | Operation timed out |
| -3 | INVALID_PARAMETER | 🟠 error | Invalid joint name or value |
| -4 | SAFETY_VIOLATION | 🔴 critical | Command exceeds joint limits |
| -5 | NOT_INITIALIZED | 🟠 error | Not connected to robot |
| -6 | UNKNOWN_ROBOT | 🟠 error | Unsupported robot ID |

### Troubleshooting

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| Connection failed | Robot powered off | Check battery and power switch |
| Connection failed | Wrong DDS domain | Verify domain_id parameter |
| Command rejected | Joint limit exceeded | Check robot-specific joint limits |
| Slow response | Network latency | Check WiFi/Ethernet connection |
| Unknown robot ID | Typo in robot_id | Use `list_robots()` to see valid IDs |

## Architecture

```
unitree_sdk2_mcp_server.py
├── SDK_METADATA        — SDK version and source info
├── RobotState          — Robot state dataclass (generic)
├── StateBuffer         — Thread-safe ring buffer for DDS data
├── UnitreeDDSBridge    — DDS communication bridge
│   ├── connect()       — Initialize DDS participant
│   ├── _dds_listener() — Background thread at 100Hz
│   └── publish_command() — Send /lowcmd
├── robot_configs.py    — Configuration for all 9 robots
│   ├── G1_CONFIG       — G1 humanoid (23 DOF)
│   ├── GO2_CONFIG      — Go2 quadruped (12 DOF)
│   ├── H1_CONFIG       — H1 humanoid (20 DOF)
│   ├── B2_CONFIG       — B2 industrial quadruped
│   ├── A2_CONFIG       — A2 agile quadruped
│   └── R1_CONFIG       — R1 wheeled robot
└── MCP Tools           — FastMCP tool definitions
```

## DDS Topics

| Topic | Direction | Description |
|-------|-----------|-------------|
| `/lowstate` | Subscribe | Robot state (joints, IMU, battery) |
| `/lowcmd` | Publish | Robot commands (joint targets, mode) |
| `/sportmodestate` | Subscribe | Sport mode state |

## Dependencies

- Python 3.10+
- `mcp[fastmcp]` — MCP framework
- `unitree_sdk2` — Unitree DDS SDK (install separately from hardware package)
- `cyclonedds` or `fastdds` — DDS middleware

## References

- [Unitree SDK2 GitHub](https://github.com/unitreerobotics/unitree_sdk2)
- [Unitree Developer Docs](https://support.unitree.com/home/zh/developer)
- [Unitree G1 Product Page](https://www.unitree.com/products/g1)
- [Unitree Go2 Product Page](https://www.unitree.com/products/go2)
- [DDS Specification](https://www.dds-foundation.org/)
- [MCP Protocol](https://modelcontextprotocol.io/)

## License

MIT License — See [LICENSE](LICENSE)

## Part of ROSClaw

- [rosclaw](https://github.com/ros-claw/rosclaw) — Core framework (see [arXiv paper](https://arxiv.org/pdf/2604.04664))
- [unitree-sdk2-mcp](https://github.com/ros-claw/unitree-sdk2-mcp) — Unitree Multi-Robot (DDS)
- [rosclaw-ur-ros2-mcp](https://github.com/ros-claw/rosclaw-ur-ros2-mcp) — UR5 arm (ROS2)
- [rosclaw-gimbal-mcp](https://github.com/ros-claw/rosclaw-gimbal-mcp) — GCU Gimbal (Serial)
- [rosclaw-ur-rtde-mcp](https://github.com/ros-claw/rosclaw-ur-rtde-mcp) — UR5 via RTDE

---

**Generated by ROSClaw SDK-to-MCP Transformer**

*SDK Version: unitree_sdk2 2.1.0+ | Protocol: DDS | Robots: G1, Go2, Go2w, H1, H2, B2, B2w, A2, R1*
