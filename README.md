# simple_mpc_msgs

ROS 2 message definitions for the VLA → MPC pipeline in [simple_mpc](https://gitlab.laas.fr/agimus/simple-mpc).

## Messages

### `VlaChunk`

A single inference output from a Vision-Language-Action model: a fixed-length window
of joint-space waypoints published at roughly 10 Hz.

```
std_msgs/Header header          # stamp = VLA inference completion time (not receive time)
uint32          chunk_index     # monotonically increasing counter
float64         dt_between_actions  # seconds between consecutive waypoints
string          action_space    # "absolute_joint_position" | "delta_joint_position"
uint32          chunk_rows      # number of waypoints
uint32          chunk_cols      # number of joints (7 arm + optional gripper)
float64[]       actions         # row-major flat array, shape (chunk_rows, chunk_cols)
```

**`header.stamp`** must be the time at which the VLA produced the chunk, not the
local clock at publish time. `TrajectoryBufferNode` uses this to detect cross-machine
clock skew and will log a warning if `|stamp − receive_time| > 200 ms`.

**`action_space`** controls how `TrajectoryBufferNode` interprets the waypoints:

| Value | Meaning |
|---|---|
| `absolute_joint_position` | Waypoints are absolute joint angles (rad) |
| `delta_joint_position` | Waypoints are incremental offsets from the current robot state |

## Dependencies

- ROS 2 Humble
- `std_msgs`

## Build

```bash
cd /path/to/ros2_ws
colcon build --packages-select simple_mpc_msgs
```

## Usage

Both `simple_mpc_ros` (consumer) and `mpc_vla_ros` (producer) depend on this package.
Add to your `package.xml`:

```xml
<depend>simple_mpc_msgs</depend>
```

The topic convention is `vla/trajectory_chunk` with RELIABLE QoS, depth 10.
