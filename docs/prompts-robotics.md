# Robotics & Embedded Prompts — Production-Grade AI Prompt Templates

> Curated prompts for robotics (manipulation, aerial, navigation, safety) and embedded systems (microcontrollers, RTOS, MISRA-C). Based on the [CRTSE framework](best-practices.md) and Microsoft's [PromptCraft-Robotics](https://github.com/microsoft/PromptCraft-Robotics) design principles.

---

## Robotics — Design Principles (from Microsoft PromptCraft)

When prompting an LLM for robotics, define:

1. **Robot platform and capabilities** — degrees of freedom, sensors (camera, LiDAR, depth, IMU), actuators, gripper type, onboard compute
2. **Environment description** — physical space, obstacles, lighting, ground surface
3. **Safety constraints** — speed limits, forbidden zones, collision thresholds, emergency stop conditions
4. **Available API** — the function signatures the LLM can call (move_to, pick, place, scan, etc.)
5. **Task decomposition** — plan → sense → act → verify, with checkpoints between steps

---

## Manipulation

### Pick and place task
```
You are controlling a 6-DOF robotic arm (UR5e) with a Robotiq 2F-85 parallel gripper in a warehouse pick-and-place cell. The robot has:
- A wrist-mounted Intel RealSense D435 RGB-D camera
- Force-torque sensing at the wrist
- Available functions: move_to_joint_positions(joints), move_to_pose(x, y, z, roll, pitch, yaw), open_gripper(), close_gripper(force_N), get_image(), get_depth(), get_ft_reading()
- Workspace: 850mm reach
- Safety: collision stop at 50N contact force, max TCP speed 1.0 m/s

The work cell contains:
- A bin of mixed consumer products (boxes, bottles, polybags) — unknown pose
- A shipping carton at known position [0.5, -0.3, 0.1] — place target
- No overhead obstacles

Task: Generate a pick-and-place plan to pick ONE item from the bin and place it in the carton. Output as a step-by-step sequence using the available API calls, with sensor checkpoints between each motion step.

Safety requirements:
- Move to approach pose 100mm above the estimated grasp point before descending
- Monitor force-torque during grasp — stop and retry if force exceeds 20N before expected contact
- During transport, the TCP must stay at least 200mm above the bin walls
- Verify gripper closed on an object before transport (check gripper width after close)
- If three consecutive grasp attempts fail, report failure and return to home position

Output: a numbered sequence of function calls with inline sensor checks and contingency branches.
```

### Visual servoing with object tracking
```
You are controlling a robot arm with an eye-in-hand camera tracking a moving object on a conveyor belt. The camera feeds a bounding box detector: get_bbox() returns (x, y, w, h) of the tracked object in image coordinates (640x480).

Available functions:
- move_velocity(vx, vy, vz, wx, wy, wz) — Cartesian velocity control
- get_bbox() — returns latest detection
- stop() — halt all motion

Task: Implement a visual servoing controller that centers the object in the camera frame and converges to a fixed target depth (bbox height = 120px). The object moves at ~0.2 m/s — you must predict its position, not chase.

Requirements:
- Use proportional control with feed-forward velocity estimation
- If tracking is lost (no bbox for 5+ frames), stop and report
- Target convergence: bbox center within 20px of image center, height within 10px of 120px
- Max approach velocity: 0.3 m/s

Do NOT:
- Move blindly without visual feedback between steps
- Use open-loop velocity commands — always close the loop with get_bbox()

Output: the complete servo control function with its gain constants and state machine.
```

---

## Aerial Robotics

### Autonomous drone inspection
```
You are piloting a quadrotor drone (DJI M300 RTK) for structural inspection of a wind turbine. The drone has:
- RTK GPS (±2cm positioning)
- Forward/downward stereo cameras with 20m range
- 360° LiDAR
- Available functions: takeoff(), land(), goto_waypoint(lat, lon, alt, heading, speed), hover(duration_s), capture_image(), get_lidar_scan()

The turbine has three blades, each 45m long, hub at 80m AGL. You must inspect all three blades at 5m standoff distance.

Task: Generate an inspection flight plan that covers all three blades. For each blade:
- Approach the hub from the upwind side
- Traverse the leading edge at 5m standoff, capturing images every 5m
- Inspect the pressure side with a downward traverse
- Return to hub altitude before moving to next blade

Safety constraints:
- Minimum safe altitude: 30m AGL
- Maximum wind speed for flight: 10 m/s (check before takeoff)
- Geofence: 100m radius from turbine center
- Battery reserve for RTB: always maintain 25% minimum

Output: a sequence of waypoints and hover/capture commands. Include a pre-flight checklist and an RTB (return to base) trigger condition.
```

### Obstacle avoidance with reactive planning
```
You are controlling a small indoor inspection drone (250g, 4" props) in a GPS-denied warehouse. The drone has:
- Forward-facing depth camera (Intel RealSense D435, 87° HFOV)
- Downward-facing optical flow sensor
- Onboard SLAM (provides odometry: get_pose() → x, y, z, yaw)
- Available functions: set_velocity(vx, vy, vz, yaw_rate), get_depth_image(), get_pose()

The warehouse is 50m × 30m with 8m ceiling. You must navigate from the entry point (0, 0, 2) to the far corner (45, 25, 3) while avoiding shelves (metal racking, 2m spacing, 3m tall).

Task: Implement a reactive obstacle avoidance controller using the depth camera. The drone should:
- Maintain a target cruise altitude of 3m
- Detect obstacles within a 3m safety bubble (any depth value < 3m → avoid)
- When an obstacle is detected, find the largest gap in the depth field and steer toward it
- If surrounded (no gap wider than the drone diameter + 0.5m margin), hold position and request help
- Log the SLAM pose every 0.5s for post-flight analysis

Do NOT:
- Use pre-planned waypoints (the shelf layout is unknown)
- Respond to obstacles frame-by-frame without temporal filtering (smooth the avoidance vector over 5 frames)

Output: the control loop function with the depth processing pipeline.
```

---

## Embedded Systems

### FreeRTOS task with watchdog
```
Act as a senior embedded systems engineer (ARM Cortex-M, FreeRTOS, MISRA-C).

I'm writing firmware for a safety-critical medical device pump controller (STM32H7, FreeRTOS 11, ARM GCC).

Task: Implement the pump motor control task that:
- Reads a target flow rate from a message queue (posted by the therapy controller)
- Controls a stepper motor via PWM on TIM1 (frequency maps to flow rate)
- Reads a hall-effect flow sensor on ADC1 every 50ms to verify actual flow
- If actual flow deviates from target by >5% for 3 consecutive readings → emergency stop (disable PWM, set error flag, notify therapy controller via queue)
- Services the independent watchdog (IWDG) within 500ms — if the task hangs, the watchdog reboots the system
- On startup, performs a self-test: run motor at 10% for 100ms, verify flow > 0

Requirements:
- C99 or C11, MISRA-C:2012 compliant (note any unavoidable deviations)
- No dynamic memory allocation (no malloc/calloc)
- All timing uses FreeRTOS ticks (pdMS_TO_TICKS), not busy-wait loops
- Critical sections protected with mutex (PWM register and flow reading must be atomic relative to emergency stop)
- The task must be stack-depth verified (print high-water mark on startup)

Do NOT:
- Use floating-point in ISRs
- Block in ISRs (use task notifications or queues)
- Use recursion (stack depth is pre-allocated)
- Leave the PWM enabled after emergency stop without explicit re-arm

Output: the complete `pump_control.c` with its FreeRTOS task function, initialization, and self-test sequence.
```

### Sensor fusion on microcontroller
```
Act as a senior embedded firmware engineer.

I'm fusing accelerometer, gyroscope, and magnetometer data from a BMI270 + BMM150 combo sensor on an nRF52840 (Cortex-M4F, FPU available).

Task: Implement a Madgwick AHRS filter to compute orientation quaternion (roll, pitch, yaw) at 200Hz.

Requirements:
- Use fixed-point Q15.16 math (no floating-point divide in the filter loop — the FPU is used for multiply-add only, to preserve determinism)
- The filter runs in a FreeRTOS task triggered by the BMI270 data-ready interrupt (via task notification)
- Calibrate the magnetometer on startup: collect 30 seconds of data, fit an ellipsoid, compute hard-iron and soft-iron correction
- Output: quaternion published to a message queue for the navigation task
- If the magnetometer reports saturated readings (field strength > 4900µT) or the gyro reports rates > 2000°/s (likely impact), flag the output as degraded

Do NOT:
- Use the CMSIS-DSP library if it introduces floating-point variability — implement the filter directly
- Skip the magnetometer calibration (raw readings are unusable near metal)
- Publish quaternions with NaN values — validate before publishing

Output: the Madgwick filter implementation, the magnetometer calibration routine, and the sensor task.
```