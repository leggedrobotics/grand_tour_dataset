# GrandTour ROS 1 examples

This package contains ROS 1 tools and launch files for replaying and inspecting
the [GrandTour dataset](https://grand-tour.leggedrobotics.com/dataset). The
repository-level [quick-start guide](../../README.md#ros1-instructions) covers
registration, downloading ROS bags, workspace setup, and visualization.

The dataset is also used by the
[COMFORT Localization Benchmark](https://www.codabench.org/competitions/17505/),
organized with the IROS 2026 workshop
[**Data in Field Robotics: From State Estimation to Navigation – IROS 2026**](https://jonasfrey96.github.io/data_in_field_robotics/).
The most important benchmark requirement is that submitted trajectories must
represent the robot's `prism` frame, not the origin of the frame tracked by the
estimator.

## ROS 1 examples

After building and sourcing the catkin workspace, the provided launch files can
be used to inspect the sensor streams:

```bash
roslaunch grand_tour_ros1 lidars.launch
roslaunch grand_tour_ros1 cameras.launch
roslaunch grand_tour_ros1 cameras_helpers.launch
roslaunch grand_tour_ros1 imus.launch
```

The replay helper selects the relevant bags and enables simulated time:

```bash
rosrun grand_tour_ros1 rosbag_play.sh --help
rosrun grand_tour_ros1 rosbag_play.sh --lidars --tf_model
```

Use `tf_model.bag` when the complete robot-model tree is needed, or
`tf_minimal.bag` for the core sensor frames.

## Inspecting transforms

### Live TF tree

While replaying the mission's `/tf` and `/tf_static` topics, visualize the
complete tree with:

```bash
rosrun rqt_tf_tree rqt_tf_tree
```

Query a transform with the reference frame first and target frame second:

```bash
rosrun tf tf_echo <reference_frame> <target_frame>
# Alternatively:
rosrun tf2_ros tf2_echo <reference_frame> <target_frame>
```

For example, this reports `T_hesai_lidar_prism`:

```bash
rosrun tf tf_echo hesai_lidar prism
```

For offline bags, run the following commands in separate terminals and unpause
the bag long enough to publish the transforms:

```bash
roscore
```

```bash
rosparam set use_sim_time true
rosbag play --clock --pause /path/to/mission_tf_model.bag
```

```bash
rosrun tf tf_echo <reference_frame> <target_frame>
```

Always use the calibration bag belonging to the same mission as the trajectory;
calibrations can differ between missions.

### Extract a direct static transform from a bag

The included script can read a direct static transform (or its inverse) from a
bag containing `/tf_static`:

```bash
python3 scripts/extract_transforms_from_tf_static.py \
  /path/to/mission_tf_model.bag \
  livox_imu livox_lidar \
  --format matrix
```

It requires the ROS 1 Python API, NumPy, PyYAML, and `pyquaternion`. Unlike
`tf_echo`, the script does not compose a chain of intermediate TF edges, so use
`tf_echo` for arbitrary pairs of connected frames.

### Transform convention

This document uses `T_A_B` for the pose of frame `B` expressed in frame `A`:

```text
T_A_B = [ R_A_B  p_A_B ]
        [   0       1   ]
```

It maps coordinates of a point from `B` into `A`:

```text
p_A = R_A_B p_B + p_A_B
```

Accordingly, `tf_echo A B` reports `T_A_B`: its translation is the position of
the origin of `B` expressed in `A`, and its quaternion is the orientation of
`B` relative to `A`. ROS and TUM text use quaternion order
`qx qy qz qw` (`x y z w`).

## Convert any estimated trajectory to the prism frame

### Required output

Every benchmark row must contain `T_world_prism(t)`, where:

- `world` is the estimator's map, odometry, or world frame;
- `source` is the body, IMU, LiDAR, camera, or other frame whose pose the
  estimator produces; and
- `prism` is the robot frame at the Leica RTS prism point used by the reference
  trajectory.

Do **not** submit `T_world_base`, `T_world_imu`, `T_world_lidar`, or another
sensor-origin trajectory without converting it.

Read `T_source_prism` from the mission's TF tree with `source` first and
`prism` second, then right-multiply each estimated pose:

```text
T_world_prism(t) = T_world_source(t) T_source_prism(t)
```

This assumes the estimator reports the pose of `source` in `world`. If it
instead reports `T_source_world` (the world frame expressed in `source`), invert
that pose first:

```text
T_world_prism(t) = inverse(T_source_world(t)) T_source_prism(t)
```

Expanded into rotation and translation, this is:

```text
R_world_prism(t) = R_world_source(t) R_source_prism(t)

p_world_prism(t) = p_world_source(t)
                   + R_world_source(t) p_source_prism(t)
```

The source-to-prism translation is expressed in `source`, so the lever arm must
rotate with the robot. Adding `p_source_prism` directly in world coordinates is
wrong. The multiplication order also matters: do not use `T_prism_source` and
do not left-multiply the estimated pose.

For example:

```bash
# Estimator tracks the base frame:
rosrun tf tf_echo base prism
# T_world_prism(t) = T_world_base(t) T_base_prism

# Estimator tracks the Hesai LiDAR frame:
rosrun tf tf_echo hesai_lidar prism
# T_world_prism(t) = T_world_hesai_lidar(t) T_hesai_lidar_prism
```

If `T_source_prism` is static, query it once from the matching mission's
calibration. If it is time-varying, query it at every trajectory timestamp.

> [!IMPORTANT]
> A trajectory containing only the position of `source` cannot in general be
> converted exactly to the prism position. The conversion also needs
> `R_world_source(t)` so that the source-to-prism lever arm can rotate. A
> position-only trajectory is sufficient only when its positions already refer
> to the prism point (or when the source and prism origins coincide).

### Python conversion example

The following example converts an arbitrary TUM pose trajectory. Replace the
example `T_source_prism` values with the translation and quaternion returned by
`tf_echo source prism` for the relevant mission.

```python
import numpy as np
from scipy.spatial.transform import Rotation


def pose_matrix(xyz, q_xyzw):
    T = np.eye(4)
    T[:3, :3] = Rotation.from_quat(q_xyzw).as_matrix()
    T[:3, 3] = xyz
    return T


# Replace with the mission-specific result of: tf_echo source prism
p_source_prism = np.array([0.0, 0.0, 0.0])
q_source_prism = np.array([0.0, 0.0, 0.0, 1.0])
T_source_prism = pose_matrix(p_source_prism, q_source_prism)

with open("source.tum") as src, open("mission-name.tum", "w") as dst:
    for line in src:
        if not line.strip() or line.lstrip().startswith("#"):
            continue

        t, x, y, z, qx, qy, qz, qw = map(float, line.split())
        T_world_source = pose_matrix(
            np.array([x, y, z]), np.array([qx, qy, qz, qw])
        )

        # Order matters: right-multiply by the source-to-prism transform.
        T_world_prism = T_world_source @ T_source_prism

        p = T_world_prism[:3, 3]
        q = Rotation.from_matrix(T_world_prism[:3, :3]).as_quat()
        dst.write(
            f"{t:.17g} {p[0]:.17g} {p[1]:.17g} {p[2]:.17g} "
            f"{q[0]:.17g} {q[1]:.17g} {q[2]:.17g} {q[3]:.17g}\n"
        )
```

SciPy accepts and returns quaternions in `[qx, qy, qz, qw]` order. Preserve the
dataset timestamp time base and write timestamps in seconds; rebasing them to
start at zero prevents association with the reference trajectory.

The evaluator applies one rigid SE(3) alignment to synchronized poses. A
different fixed world origin or orientation is therefore acceptable, but scale
is not corrected. Alignment also cannot remove the time-varying position error
caused by omitting the rotating source-to-prism lever arm.

## COMFORT benchmark quick reference

Use the official
[Codabench benchmark page](https://www.codabench.org/competitions/17505/) for
submissions and the leaderboard. The
[GrandTour localization page](https://grand-tour.leggedrobotics.com/tasks/localization)
contains the benchmark information and rules; these official pages are
authoritative if anything changes.

### Schedule and mission splits

| Phase | Dates | Files for a complete score | Submission limit |
| --- | --- | --- | --- |
| Validation | July 13, 2026 at 00:00 UTC – August 3, 2026 at 00:00 UTC | `heap-1.tum`, `arc-3.tum` | 20/day, 100 total |
| Test | August 3, 2026 at 00:00 UTC – October 30, 2026 at 23:59 UTC | `arc-2.tum`, `arc-7.tum`, `con-4.tum`, `eig-1.tum`, `snow-2.tum`, `spx-2.tum` | 15/day, 60 total |

Test reference trajectories remain hidden. Only test-phase submissions count
toward the final ranking. Award-eligible submissions must be made from August
3, 2026 at 00:00 UTC through September 21, 2026 at 23:59 Anywhere on Earth
(September 22 at 11:59 UTC). Novel methods are prioritized during award
selection, and award recipients must present their results at the
[IROS 2026 workshop](https://jonasfrey96.github.io/data_in_field_robotics/).
Leaderboard rank and award selection are separate.

### Submission package

Upload one zip archive with all trajectory files at the archive root and no
subdirectories. The archive can have any name; `comfort_submission.zip` is the
preferred name. For example:

```text
comfort_submission.zip
├── heap-1.tum
└── arc-3.tum
```

Mission-name matching is case-insensitive and accepts `.tum` or `.txt`. Unknown
missions and files from the wrong phase fail the pre-check. Include every
mission in the selected phase for a complete score; each missing mission is
assigned `9999 m` for ATE, RTE, and LE, and that penalty is included in the
phase average.

Each non-comment trajectory row must have exactly eight finite numeric values:

```text
# timestamp x y z qx qy qz qw
1731588412.024509 -13.128140 -4.550792 -0.846991 0 0 0 1
```

Each mission file must contain at least 100 poses, and every quaternion must be
nonzero. Strictly increasing timestamps are recommended. Duplicate or
non-increasing timestamps produce a warning and can be scored only if timestamp
association still succeeds.

### Evaluation and method policy

- ATE is Absolute Translation Error RMSE after trajectory alignment.
- RTE is Relative Translation Error RMSE over relative pose differences.
- LE is the final absolute trajectory-error sample.
- All metrics are in meters and lower is better. The leaderboard is sorted by
  average ATE over every mission in the submitted phase.
- All released GrandTour sensor streams may be used. The `CPT7 IMU` stream is
  redacted and unavailable.
- Submitted methods must be applicable in real time; post-processing is not
  allowed.

See the [GrandTour dataset page](https://grand-tour.leggedrobotics.com/dataset)
for mission access and sensor details, the
[GrandTour localization page](https://grand-tour.leggedrobotics.com/tasks/localization)
for the current rules and terms, and the
[Codabench benchmark page](https://www.codabench.org/competitions/17505/) for
submissions and the leaderboard.
