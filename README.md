# mocap_px4_relays

Mocap-agnostic relay nodes for bridging pose data to/from PX4. These nodes were extracted from [vicon4px4](https://github.com/evannsm/vicon4px4) so they can be reused with any motion capture source (Vicon, OptiTrack, etc.).

## Nodes

### visual_odometry_relay

Bridges a `geometry_msgs/PoseStamped` pose into the PX4 EKF as external vision input.

- **Subscribes:** `/vicon/drone/drone` (`geometry_msgs/msg/PoseStamped`)
- **Publishes:** `/fmu/in/vehicle_visual_odometry` (`px4_msgs/msg/VehicleOdometry`) at **35 Hz**

The node converts the ROS quaternion order (x, y, z, w) to PX4 order (w, x, y, z), sets `pose_frame` to `POSE_FRAME_NED`, and timestamps each message in microseconds. The fixed-rate timer decouples publishing from the mocap callback rate, always sending the most recent pose.

### full_state_relay

Merges fused EKF outputs into a single `mocap_msgs/FullState` message for convenient downstream consumption.

- **Subscribes:**
  - `/fmu/out/vehicle_odometry` (`px4_msgs/msg/VehicleOdometry`) — position, velocity, quaternion, angular velocity
  - `/fmu/out/vehicle_local_position` (`px4_msgs/msg/VehicleLocalPosition`) — acceleration (ax, ay, az)
- **Publishes:** `/merge_odom_localpos/full_state_relay` (`mocap_msgs/msg/FullState`) at **40 Hz**

A gating mechanism prevents publishing stale or low-rate data:

| Parameter            |  Type  | Default | Description                              |
| -------------------- | -----: | ------- | ---------------------------------------- |
| `min_rate_hz`        | double | `50.0`  | Minimum acceptable callback rate         |
| `recent_timeout_sec` | double | `0.10`  | Maximum age of data before it's stale    |
| `rate_ema_alpha`     | double | `0.9`   | EMA smoothing factor (higher = smoother) |

## Usage

```bash
# Launch visual odometry relay only
ros2 launch mocap_px4_relays visual_odometry_relay.launch.py

# Launch full state relay only
ros2 launch mocap_px4_relays full_state_relay.launch.py
```

## Dependencies

- `rclcpp`
- `geometry_msgs`
- `px4_msgs`
- `mocap_msgs`
