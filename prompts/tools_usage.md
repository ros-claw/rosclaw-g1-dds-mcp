# Tools Usage Guide: Unitree Multi-Robot Platform

## Multi-Robot Discovery

### `list_robots()`
List all supported Unitree robots.

**Returns**: Array of robot IDs, names, types, and DOF.

**When to use**: At start of session to see available robots.

---

### `get_robot_info(robot_id: str)`
Get detailed configuration for a specific robot.

**Parameters**:
- `robot_id`: Robot identifier (e.g., "g1", "go2", "h1")

**Returns**: Joint names, limits, velocity limits, capabilities.

**When to use**: Before connecting to understand robot capabilities.

---

### `connect_robot(robot_id: str, domain_id: int = 0)`
Connect to a robot via DDS.

**Parameters**:
- `robot_id`: Robot to connect to
- `domain_id`: DDS domain (default: 0)

**When to use**: Before any robot commands.

**Safety**: Verify robot is powered on and network is connected.

---

## Essential Tools (All Robots)

### `stand_up(robot_id: str)`
Transition robot from sitting/lying to standing position.

**When to use**: Before any walking or manipulation task.

**Supported**: G1, H1, H2, Go2, B2, A2

**Safety**: Ensure adequate clearance above (2m) and around (1m radius).

---

### `sit_down(robot_id: str)`
Safely lower robot to sitting position.

**When to use**: End of session, low battery, or emergency shutdown.

**Safety**: Robot will freeze until fully seated.

---

### `walk_with_velocity(robot_id: str, linear_x: float, linear_y: float, angular_z: float, duration: float)`
Walk with specified velocity for duration.

**Parameters**:
- `robot_id`: Target robot
- `linear_x`: Forward/backward velocity (m/s, max +-1.0)
- `linear_y`: Left/right velocity (m/s, max +-0.5)
- `angular_z`: Rotation velocity (rad/s, max +-1.0)
- `duration`: Seconds to walk

**Supported**: G1, Go2, H1, H2, B2, A2

**Safety**:
- Verify ground is flat and clear
- Start with slow speed (0.1-0.3 m/s)
- Monitor IMU for stability

---

### `move_joint(robot_id: str, joint_name: str, target_position: float, duration: float)`
Move a specific joint to target position.

**Parameters**:
- `robot_id`: Target robot
- `joint_name`: Joint name (see robot-specific lists)
- `target_position`: Target angle in radians
- `duration`: Movement duration in seconds

**Joint Names** (Humanoid): `left_hip_pitch`, `left_knee`, `right_shoulder_pitch`, `left_elbow`, `waist_yaw`

**Joint Names** (Quadruped): `front_left_hip`, `front_left_thigh`, `front_left_calf`

**When to use**: Precise manipulation or pose adjustment.

**Safety**: Check joint limits in e_urdf.json before commanding.

---

### `move_joints(robot_id: str, positions: dict, duration: float)`
Move multiple joints simultaneously.

**Parameters**:
- `robot_id`: Target robot
- `positions`: Dict of {joint_name: target_position}
- `duration`: Movement duration

**Example**:
```python
move_joints("g1", {
    "left_shoulder_pitch": -1.57,
    "left_elbow": -1.0
}, duration=2.0)
```

---

### `stop_movement(robot_id: str)`
Stop all movement and hold current position.

**When to use**: Emergency stop, end of motion sequence.

---

### `emergency_stop(robot_id: str)`
Immediate emergency halt - disables motors.

**When to use**: CRITICAL safety situations only.

**Warning**: Robot may fall. Use stop_movement() for normal stops.

---

## Humanoid Special Tools

### `wave_hand(robot_id: str, hand: str = "right", times: int = 3)`
Perform waving motion with specified hand.

**Parameters**:
- `robot_id`: Target robot (G1, H1, H2 only)
- `hand`: "left" or "right"
- `times`: Number of waves (default: 3)

**When to use**: Greeting gesture.

---

## Common Workflows

### Safe Walking Sequence (G1)
```python
# 1. Check prerequisites
list_robots()  # See available robots
get_robot_info("g1")  # Get G1 configuration

# 2. Connect
connect_robot("g1", domain_id=0)

# 3. Stand up
stand_up("g1")

# 4. Walk slowly
walk_with_velocity("g1", linear_x=0.1, linear_y=0, angular_z=0, duration=5.0)

# 5. Stop
stop_movement("g1")
```

### Go2 Trotting
```python
connect_robot("go2", domain_id=0)
stand_up("go2")
walk_with_velocity("go2", linear_x=0.5, linear_y=0, angular_z=0, duration=10.0)
stop_movement("go2")
```

### Arm Manipulation (G1)
```python
# 1. Connect and stand
connect_robot("g1", domain_id=0)
stand_up("g1")

# 2. Move shoulder (slowly)
move_joint("g1", "right_shoulder_pitch", -1.57, duration=2.0)

# 3. Move elbow
move_joint("g1", "right_elbow", -1.0, duration=1.0)

# 4. Wave
wave_hand("g1", hand="right", times=3)
```

### Multi-Joint Pose
```python
move_joints("g1", {
    "left_shoulder_pitch": -0.5,
    "left_shoulder_roll": 0.3,
    "left_elbow": -1.2,
    "right_shoulder_pitch": -0.5,
    "right_shoulder_roll": -0.3,
    "right_elbow": -1.2
}, duration=3.0)
```

## Error Codes

- `CONNECTION_FAILED`: Failed to connect to DDS domain
- `UNKNOWN_ROBOT`: Invalid robot_id specified
- `NOT_INITIALIZED`: Not connected to robot
- `JOINT_LIMIT_ERROR`: Target exceeds joint limits
- `SAFETY_VIOLATION`: Command violates safety constraints
- `DDS_TIMEOUT`: Communication error

## Best Practices

1. **Always list robots first** to confirm target is available
2. **Always get robot info** before connecting
3. **Check battery** before movement (use status resource)
4. **Start slow** - Use low velocities for testing
5. **Verify state** after each command
6. **Plan paths** - Avoid rapid direction changes
7. **Emergency stop** - `emergency_stop()` is always available

## Robot-Specific Notes

**G1**: 23 DOF humanoid, supports all features including arm control and waving.

**Go2**: 12 DOF quadruped, fast and agile. No arm control.

**H1/H2**: 20 DOF humanoids, similar to G1 but larger frame.

**B2**: 60kg industrial quadruped. Strong but slower. Supports heavy payloads.

**A2**: 15kg agile quadruped. Fast and responsive, good for education.

**R1**: 4 DOF wheeled robot. Use velocity commands for navigation.
