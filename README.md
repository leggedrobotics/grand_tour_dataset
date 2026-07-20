<p align="center">
  <img src="assets/2026_GRANDTOUR_LOGO.png" alt="GrandTour Logo" style="width:15%;" />
</p>

<h1 align="center" style="margin-bottom: 0;">
  <br>
  The GrandTour Dataset
  <br>
</h1>
<p align="center">
  <em><small>A project brought to you by <a href="https://rsl.ethz.ch/">RSL - ETH Zurich</a>.</small></em>
</p>
<p align="center">
  <a href="#references">References</a> •
  <a href="#comfort-localization-benchmark-iros-2026">Benchmark</a> •
  <a href="#hugging-face-instructions">Hugging Face</a> •
  <a href="#ros1-instructions">ROS1</a> •
  <a href="#contributing">Contributing</a> •
  <a href="#news">News</a> •
  <a href="#citation">Citation</a>
</p>

## References

Please first visit the [official webpage](https://grand-tour.leggedrobotics.com/) to learn more about the available data, hardware setup, and registration.

- 📄 **GrandTour dataset paper (arXiv preprint):** [https://arxiv.org/abs/2602.18164](https://arxiv.org/abs/2602.18164)
- 🌐 **Official dataset page:** [https://grand-tour.leggedrobotics.com/dataset](https://grand-tour.leggedrobotics.com/dataset)
- 🧭 **COMFORT Localization Benchmark:** [https://grand-tour.leggedrobotics.com/tasks/localization](https://grand-tour.leggedrobotics.com/tasks/localization)
- 🤖 **IROS 2026 workshop — Data in Field Robotics:** [https://jonasfrey96.github.io/data_in_field_robotics/](https://jonasfrey96.github.io/data_in_field_robotics/)
- 🤗 **Hugging Face dataset:** [https://huggingface.co/datasets/leggedrobotics/grand_tour_dataset](https://huggingface.co/datasets/leggedrobotics/grand_tour_dataset)
- 💻 **GrandTour examples repository:** [https://github.com/leggedrobotics/grand_tour_dataset](https://github.com/leggedrobotics/grand_tour_dataset)
- 🔩 **Boxi software (sensor payload):** [https://github.com/leggedrobotics/grand_tour_box](https://github.com/leggedrobotics/grand_tour_box)

Visit our [sponsors and partners](https://grand-tour.leggedrobotics.com/about), including **Leica Geosystems (Hexagon)**.

---

<br>

## COMFORT Localization Benchmark (IROS 2026)

The [COMFORT Localization Benchmark](https://grand-tour.leggedrobotics.com/tasks/localization)
evaluates state-estimation and localization methods on GrandTour. It is
organized with the IROS 2026 workshop
[**Data in Field Robotics: From State Estimation to Navigation – IROS 2026**](https://jonasfrey96.github.io/data_in_field_robotics/).

| Phase | Dates | Files for a complete score | Submission limit |
| --- | --- | --- | --- |
| Validation | July 13, 2026 at 00:00 UTC – August 3, 2026 at 00:00 UTC | `heap-1.tum`, `arc-3.tum` | 20/day, 100 total |
| Test | August 3, 2026 at 00:00 UTC – October 30, 2026 at 23:59 UTC | `arc-2.tum`, `arc-7.tum`, `con-4.tum`, `eig-1.tum`, `snow-2.tum`, `spx-2.tum` | 15/day, 60 total |

Only test-phase submissions count toward the final ranking, which uses average
Absolute Trajectory Error (ATE); lower is better. Test reference trajectories
remain hidden. Award-eligible submissions must be made from August 3, 2026 at
00:00 UTC through September 21, 2026 at 23:59 Anywhere on Earth (September 22
at 11:59 UTC). Award recipients must present their results at the IROS 2026
workshop. Novel methods are prioritized during award selection, which is
distinct from leaderboard ranking.

### Critical trajectory-frame requirement

Every submitted TUM row must represent `T_world_prism(t)`: the pose of the
robot's `prism` frame at the Leica RTS reference point, expressed in the
estimator's map or odometry frame. Do not directly submit the origin of `base`,
an IMU, LiDAR, camera, or another sensor frame.

If an estimator tracks a rigidly attached frame named `source`, obtain the
mission-specific `T_source_prism` from TF and right-multiply every pose:

```text
T_world_prism(t) = T_world_source(t) T_source_prism

p_world_prism(t) = p_world_source(t)
                   + R_world_source(t) p_source_prism
```

The source-to-prism lever arm is expressed in `source` and must rotate with the
robot. A position-only source trajectory cannot generally be converted exactly
without `R_world_source(t)`. See the
[ROS 1 prism-conversion guide](examples_ros1/grand_tour_ros1/README.md#convert-any-estimated-trajectory-to-the-prism-frame)
for TF commands, transform conventions, common mistakes, and a complete Python
converter.

### Submission essentials

- Upload one zip archive containing the phase's TUM files at the archive root;
  do not add subdirectories.
- Each non-comment row must contain exactly eight finite numeric values:
  `timestamp x y z qx qy qz qw`.
- Each mission file needs at least 100 poses and nonzero quaternions. Strictly
  increasing timestamps are recommended.
- Each missing phase mission receives `9999 m` for ATE, RTE, and Last Error,
  including in the phase average.
- The evaluator performs rigid SE(3) alignment, so a different fixed world
  origin or orientation is acceptable; scale is not corrected.
- All released sensor streams may be used. The `CPT7 IMU` stream is redacted
  and unavailable. Methods must be applicable in real time; post-processing is
  not allowed.

The official [benchmark page](https://grand-tour.leggedrobotics.com/tasks/localization)
and its linked Codabench pages are authoritative if the rules change.

---

<br>

## Projects using the GrandTour Dataset

> This is a growing list of known public projects and papers that use GrandTour
> data directly or as curated source data. Last literature check: **July 19,
> 2026**. Citation-only mentions and dataset-comparison tables are not included.

| Project / paper | How GrandTour is used | Preview |
| --- | --- | --- |
| [**Physical Terrain Parameter Learning**](https://github.com/leggedrobotics/physical_terrain_parameter_learning) | Learns terrain and simulation parameters from RGB and proprioception. | <img src="assets/projects/chen2024.png" height="64"/> |
| [**Forward Dynamics Model Learning (FDM)**](https://github.com/leggedrobotics/fdm) | Learns platform-aware forward dynamics for safer navigation. | <img src="assets/projects/roth2025.png" height="64"/> |
| [**Holistic Fusion**](https://github.com/leggedrobotics/holistic_fusion) | Evaluates task- and setup-agnostic factor-graph state estimation. | <img src="assets/projects/nubert2025.png" height="64"/> |
| [**RESPLE: Recursive Spline LIO**](https://asig-x.github.io/resple_web/) | Benchmarks recursive-spline LiDAR-inertial odometry on GrandTour sequences. | <img src="assets/projects/cao2025.png" height="64"/> |
| [**GaussGym**](https://escontrela.me/gauss_gym/) | Converts GrandTour scenes into collision-aware 3D Gaussian Splatting simulators for high-throughput navigation and locomotion. | — |
| [**LiMo: Less Is More (Scalable Visual Navigation)**](https://arxiv.org/abs/2601.17815) | Extends GrandTour deployments with large-scale embodiment-specific navigation trajectories. | — |
| [**VLD: Visual-Language Goal Distance for RL Navigation**](https://arxiv.org/abs/2512.07976) | Adds five hours of GrandTour trajectories to its self-supervised visual-language distance training corpus. | — |
| [**ViLiNT: Multimodal Embodiment-Aware Navigation Transformer**](https://github.com/LARIAD/ViLiNT) | Adds six hours of GrandTour RGB and LiDAR trajectories to heterogeneous navigation-policy training. | — |
| [**DeFM: Learning Foundation Representations from Depth**](https://arxiv.org/abs/2601.18923) | Uses GrandTour in a large real-world depth pretraining corpus. | — |
| [**WildOS: Open-Vocabulary Object Search in the Wild**](https://arxiv.org/abs/2602.19308) | Trains frontier prediction using annotated GrandTour images. | — |
| [**NaviTrace: Evaluating Embodied Navigation of VLMs**](https://arxiv.org/abs/2510.26909) | Curates GrandTour image scenarios for embodied VLM navigation evaluation. | — |
| [**Green for Go, Red for No**](https://arxiv.org/abs/2607.05122) | Uses the `ETH-2` mission to evaluate visual grounding and waypoint selection for OmniVLA. | — |
| [**CaLiSym**](https://arxiv.org/abs/2607.06824) | Learns and evaluates contact-constrained quadruped dynamics from GrandTour locomotion trajectories. | — |
| [**LIMOncello**](https://github.com/CPerezRuiz335/LIMOncello) | Evaluates an iterated error-state LiDAR-inertial odometry method on GrandTour. | — |
| [**BIEVR-LIO**](https://github.com/ethz-asl/bievr-lio) | Evaluates robust LiDAR-inertial odometry on `ARC-4`, `LEE-1`, `LEICA-2`, and `TRIM-1`. | — |
| [**Iterated Invariant EKF for Quadruped Robot Odometry**](https://arxiv.org/abs/2604.15449) | Evaluates proprioceptive odometry on the `Pilatus-Hike2` mission. | — |
| [**A Proprioceptive-Only State-Estimation Benchmark**](https://github.com/iit-DLSLab/state_estimation_benchmark) | Compares MUSE, IEKF, and an invariant smoother on `CYN-1`, including accuracy and runtime. | — |
| [**Sensor Configuration Matters**](https://arxiv.org/abs/2606.19067) | Systematically evaluates visual, visual-inertial, and LiDAR-visual-inertial SLAM across seven GrandTour missions and multiple sensor configurations. | — |
| [**Ultra-Fusion**](https://sjtuyinjie.github.io/ultrafusion-web/) | Evaluates robust cross-platform sensor fusion on multiple GrandTour legged-robot missions. | — |
| [**G-PROBE**](https://arxiv.org/abs/2607.06782) | Evaluates cross-sensor LiDAR place recognition and localization between `SBB-1` and `SBB-2`. | — |
| [**Chalito**](https://arxiv.org/abs/2607.09968) | Validates an extensible filter-based quadruped state-estimation library on `Pilatus-Hike2`. | — |
| [**Model-Based and Neural-Aided Approaches for Dog Dead Reckoning**](https://arxiv.org/abs/2603.07582) | Trains and evaluates inertial dead reckoning on 17 GrandTour ANYmal trajectories. | — |

---

<br>

## Hugging Face Instructions

You can find Jupyter notebooks and scripts with full instructions in the [`examples_hugging_face`](./examples_hugging_face) directory.

### Installation Instructions

<details>
<summary>Click for installation details</summary>

These steps assume you are using **[uv](https://github.com/astral-sh/uv)** for dependency management.

### 1. Install using `uv`

```bash
pip3 install uv
uv sync
cd examples_hugging_face
uv sync
uv run scripts/download_data.py
```

</details>

### 📒 Jupyter Notebook Examples

* **[Accessing/Downloading GrandTour Data](./examples_hugging_face/notebooks/access.ipynb)**  
  *Learn how to download the GrandTour datasets from Hugging Face.*

* **[Exploring GrandTour Data](./examples_hugging_face/notebooks/explore.ipynb)**  
  *Explore the dataset structure and learn how to work with Zarr data.*

### 🐍 Python Scripts

* **[zarr_transforms.py](./examples_hugging_face/scripts/zarr_transforms.py)**  
  *Demonstrates how to use transforms and provides helper functions for Zarr data.*

* **[plot_lidar_3d.py](./examples_hugging_face/scripts/plot_lidar_3d.py)**  
  *Visualize LiDAR data in 3D space.*

* **[project_lidar_on_image.py](./examples_hugging_face/scripts/project_lidar_on_image.py)**  
  *Project LiDAR points onto camera images, accounting for camera distortion and relative motion.*

* **[dynamic_points_filtering_using_images.py](./examples_hugging_face/scripts/dynamic_points_filtering_using_images.py)**  
  *Removes dynamic objects from LiDAR point clouds using image segmentation and saves results in Zarr format.*

* **[generate_elevation_maps.py](./examples_hugging_face/scripts/generate_elevation_maps.py)**  
  *Generates elevation maps from LiDAR and depth cameras.*

* **[nerfstudio_convert.py](./examples_hugging_face/scripts/nerfstudio_convert.py)**  
  *Converts datasets into nerfstudio format for training Gaussian Splatting models.*

---

<br>

## ROS1 Instructions

For ROS 1 replay tools, TF conventions, calibration extraction, and detailed
COMFORT prism-frame conversion instructions, see the
[`grand_tour_ros1` package guide](examples_ros1/grand_tour_ros1/README.md).

### Download

<details>
<summary>Click here</summary>

To access and download the GrandTour dataset ROS bags, please follow these steps:

#### 1. Register for Access

- **Register here:** [Google Form Registration](https://forms.gle/2qJkGYJ6oxnBvdNq9)

#### 2. Download ROS bags

**Option 1 – Command Line Interface (Recommended):**

Install the CLI tool and log in:

```bash
pip3 install kleinkram
klein login
```

- You can now explore the CLI using tab-completion or the `--help` flag.

**Download multiple files via Python scripting:**

```bash
python3 examples_kleinkram/kleinkram_cli_example.py
```

**Directly convert ROS bags to PNG images (requires ROS1 installation):**

```bash
python3 examples_kleinkram/kleinkram_extract_images.py
```

**Option 2 – Web Interface:**

- Use the [GrandTour Dataset Web Interface](https://datasets.leggedrobotics.com/#/) to browse and download data directly.

</details>

### Installation

<details>
<summary>Click here</summary>

### Create Folders
```shell
mkdir -p ~/grand_tour_ws/src
mkdir -p ~/git
```

### Clone and Link Submodules
> **⚠️ Note:** The repository structure is under active development (ROS2/MCAP support is being added). Commands below may evolve.

```shell
# Clone the repository
cd ~/git
git clone git@github.com:leggedrobotics/grand_tour_dataset.git
cd grand_tour_dataset; git submodule update --init

# Checkout only the required packages from the grand_tour_box repository for simplicity
cd ~/git/grand_tour_dataset/examples_ros1/submodules/grand_tour_box
git sparse-checkout init --cone
git sparse-checkout set box_model box_calibration box_drivers/anymal_msgs box_drivers/gnss_msgs

# Link the repository to the workspace
ln -s ~/git/grand_tour_dataset/examples_ros1 ~/grand_tour_ws/src/
```

### Setup and Build Catkin Workspace

```shell
cd ~/grand_tour_ws
catkin init
catkin config --extend /opt/ros/noetic
catkin config --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
catkin build grand_tour_ros1
source devel/setup.bash
```

### Download Example Mission

```shell
mkdir -p ~/grand_tour_ws/src/examples_ros1/data
cd ~/grand_tour_ws/src/examples_ros1/data
pip3 install kleinkram
klein login
klein download --mission 3c97a27e-4180-4e40-b8af-59714de54a87
```

</details>

### Examples

#### Terminal 1: Launch LiDARs

```shell
roslaunch grand_tour_ros1 lidars.launch

# URDFs are automatically loaded by:
#    Boxi:     box_model box_model.launch
#    ANYmal:   anymal_d_simple_description load.launch
```

#### Terminal 2: Replay Bags

```shell
cd ~/grand_tour_ws/src/examples_ros1/data
# We provide an easy interface to replay the bags
rosrun grand_tour_ros1 rosbag_play.sh --help
rosrun grand_tour_ros1 rosbag_play.sh --lidars --tf_model

# We provide two tf_bags
#   tf_model contains frames required for the URDF model of ANYmal and Boxi.
#   tf_minimal contains only core sensor frames.
```

You can also try the same for `cameras.launch`.

**Example Output:**

| **LiDAR Visualization** | **Camera Visualization** |
| --- | --- |
| ![LiDAR Visualization](assets/rviz-lidar.gif) <br> _Visualization of LiDAR data using `lidars.launch`._ | ![Camera Visualization](assets/rviz-camera.gif) <br> _Visualization of images using `cameras.launch`._ |

#### Image Uncompression and Rectification

We provide a launch file to uncompress images and publish rectified images. Install the required dependencies:

```shell
sudo apt-get install ros-noetic-image-transport
sudo apt-get install ros-noetic-compressed-image-transport
```

```shell
roslaunch grand_tour_ros1 cameras_helpers.launch
```

#### IMUs Visualization

We use `rqt-multiplot` to visualize the IMU measurements.

Install [rqt_multiplot](https://wiki.ros.org/rqt_multiplot):

```shell
sudo apt-get install ros-noetic-rqt-multiplot -y
```

Start `rqt_multiplot` and replay the bags:

```shell
roslaunch grand_tour_ros1 imus.launch
```

```shell
cd ~/grand_tour_ws/src/examples_ros1/data
rosrun grand_tour_ros1 rosbag_play.sh --imus --ap20
```

**Example Output:**  
![assets/rqt-multiplot.png](assets/rqt-multiplot.png)

---

<br>

## Contributing

We welcome contributions to help improve and expand this project. Whether you're interested in adding new examples, enhancing existing ones, or offering suggestions, feel free to open an issue or reach out directly.

We are particularly looking for contributions in the following areas:

- New and interesting benchmarks
- ROS2 integration and conversion (MCAP / ROS2 tooling)
- Visualization tools (e.g., Viser, etc.)
- Hosting and deployment support in Asia

---

<br>

## News

- **2026-07:** The [COMFORT Localization Benchmark](https://grand-tour.leggedrobotics.com/tasks/localization) launched with validation and hidden-test phases for the IROS 2026 workshop [**Data in Field Robotics: From State Estimation to Navigation**](https://jonasfrey96.github.io/data_in_field_robotics/).
- **2026-02:** The **GrandTour dataset paper** is now available as an **arXiv preprint**: [arXiv:2602.18164](https://arxiv.org/abs/2602.18164) (submitted to IJRR).
- **MCAP (ROS2)** support is listed as **coming soon** on the official dataset page.
- We continue to expand GrandTour with new deployments, examples, and downstream benchmarks.

---

<br>

## Citation

If you use the **GrandTour dataset**, please cite the dataset paper:

```bibtex
@article{frey2026grandtour,
  title={GrandTour: A Legged Robotics Dataset in the Wild for Multi-Modal Perception and State Estimation},
  author={Frey, Jonas and Tuna, Turcan and Fu, Frank and Patterson, Katharine and Xu, Tianao and Fallon, Maurice and Cadena, Cesar and Hutter, Marco},
  journal={arXiv preprint arXiv:2602.18164},
  year={2026}
}
```

\*Equal contribution: **Jonas Frey** and **Turcan Tuna**.

If you use the **Boxi payload/software** or cite hardware/payload design decisions, please also cite:

```bibtex
@INPROCEEDINGS{Tuna-Frey-Fu-RSS-25,
    AUTHOR    = {Jonas Frey AND Turcan Tuna AND Lanke Frank Tarimo Fu AND Cedric Weibel AND Katharine Patterson AND Benjamin Krummenacher AND Matthias Müller AND Julian Nubert AND Maurice Fallon AND Cesar Cadena AND Marco Hutter},
    TITLE     = {{Boxi: Design Decisions in the Context of Algorithmic Performance for Robotics}},
    BOOKTITLE = {Proceedings of Robotics: Science and Systems},
    YEAR      = {2025},
    ADDRESS   = {Los Angeles, United States},
    MONTH     = {June}
}
```

\*Shared first authorship: **Frey, Tuna, Fu**.
