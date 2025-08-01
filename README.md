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
  <a href="#hugging-face-instructions">Hugging Face</a> •
  <a href="#ros1-instructions">ROS1</a> •
  <a href="#contributing">Contributing</a>  •
  <a href="#citation">Citation</a>
</p>

## References

More instructions can be found on the [official webpage](https://grand-tour.leggedrobotics.com/).

- Recording Setup
- Dataset Explorer
- Benchmarks

## Projects using the GrandTour Dataset

| Project                                                                                                  | Preview                                                                                           |
| -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| [**Physical Terrain Parameters Learning**](https://github.com/leggedrobotics/physical_terrain_parameter_learning) <br> *Learning simulation paramters from RGB and proprioception.*   | <img src="assets/projects/chen2024.png" height="64"/> |
| [**Fordward Dynamics Model Learning**](https://github.com/leggedrobotics/fdm) <br> *Learning dynamics model.* | <img src="assets/projects/roth2025.png" height="64"/> |
| [**Holistic Fusion**](https://github.com/leggedrobotics/holistic_fusion) <br> *Holistic State Estimation.*              |  <img src="assets/projects/nubert2025.png" height="64"/>|
| [**RESPLE: Recursive Spline LIO**](https://asig-x.github.io/resple_web/) <br> *SoTA LiDAR Inertial Odometry.*              | <img src="assets/projects/cao2025.png" height="64"/> |


## Hugging Face Instructions (Data comeing soon)

You can find Jupyter Notebooks with full instructions in the [`examples_hugging_face`](./examples_hugging_face).  
Requirements: Python>=3.11
Examples Tested with: zarr==3.0.7

<details>
<summary> Installation Instructions (UV with no preinstalled Python 3.11)</summary>

#### Step 1: Install `uv` for Dependency Management

```bash
pip3 install uv
uv install
```

#### Step 2: Install Python 3.11 (if not already installed)

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3.11-distutils
```

#### Step 3: Set Up the Virtual Environment

```bash
cd ~/git/grand_tour_dataset/examples_hugging_face
mkdir .venv; cd .venv
python3.11 -m venv grandtour
source grandtour/bin/activate
cd ..; uv pip install -r pyproject.toml
jupyter notebook
```

</details>

List of examples:

- [Accessing GrandTour Data](./examples_hugging_face/[0]_Accessing_GrandTour_Data.ipynb)
- [Exploring GrandTour Data](./examples_hugging_face/[1]_Exploring_GrandTour_Data.ipynb)

## ROS1 Instructions

### Downloading Rosbags

To access and download the GrandTour dataset rosbags, please follow these steps:

#### 1. Register for Access

- **Register here:** [Google Form Registration](https://forms.gle/2qJkGYJ6oxnBvdNq9)

#### 2. Download Rosbags

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

**Directly convert rosbags to PNG images (requires ROS1 installation):**

```bash
python3 examples_kleinkram/kleinkram_extract_images.py
```

---

**Option 2 – Web Interface:**

- Use the [GrandTour Dataset Web Interface](https://datasets.leggedrobotics.com/#/) to browse and download data directly.

---

### Create Folders

```shell
mkdir -p ~/grand_tour_ws/src
mkdir -p ~/git
```

### Clone and Link Submodules
> **⚠️ Note:** The `grand_tour_box` repository is currently private. We are actively working on making it public.

```shell
# Cloning the repository
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

### Run the Example

#### Terminal 1: Launch LiDARs

```shell
roslaunch grand_tour_ros1 lidars.launch

# URDFs are automaticlly loaded by:
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
#   tf_model contains frames requred for UDRF model of ANYmal and Boxi.
#   tf_minimal contains only core sensor frames.
```

You can also try the same for `cameras.launch`.

**Example Output:**

| **LiDAR Visualization**                                                                                 | **Camera Visualization**                                                                               |
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
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

We use rqt-multiplot to visualize the IMU measurments.

Install [rqt_multiplot](https://wiki.ros.org/rqt_multiplot):

```shell
sudo apt-get install ros-noetic-rqt-multiplot -y
```

Start rqt_multiplot and replay the bags:

```shell
roslaunch grand_tour_ros1 imus.launch
```

```shell
cd ~/grand_tour_ws/src/examples_ros1/data
rosrun grand_tour_ros1 rosbag_play.sh --imus --ap20
```

**Example Output:**
![assets/rqt-multiplot.png](assets/rqt-multiplot.png)

## Contributing

We warmly welcome contributions to help us improve and expand this project. Whether you're interested in adding new examples, enhancing existing ones, or simply offering suggestions — we'd love to hear from you! Feel free to open an issue or reach out directly.

We are particularly looking for contributions in the following areas:

- New and interesting benchmarks
- ROS2 integration and conversion
- Visualization tools (e.g., Viser, etc.)
- Hosting and deployment support in Asia

### Upcoming Plans

We're organizing a workshop at ICRA 2026 in Vienna and are currently looking for co-organizers and collaborators. We are also planning to write a community paper about this project. Everyone who contributes meaningfully will be included as a co-author.

Let’s build this together — your input matters!

## Citation

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

\*shared first authorship: Frey, Tuna, Fu.
