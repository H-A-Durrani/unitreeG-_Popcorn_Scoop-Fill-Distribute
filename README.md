
### 1. Data Collection
Collect episodes using Meta Quest 3 teleoperation:
- Follow avp_teleoperate setup
- Run X episodes of the popcorn task
- Data saves automatically to /datasets/popcorn/

### 2. Convert Dataset to LeRobot Format
python unitree_lerobot/utils/convert_unitree_json_to_lerobot.py \
    --raw-dir $HOME/datasets \
    --repo-id yourname/popcorn_dataset \
    --robot_type Unitree_G1_Dex3 \
    --push_to_hub

### 3. Train ACT Policy
python lerobot/scripts/lerobot_train.py \
    --dataset.repo_id=yourname/popcorn_dataset \
    --policy.type=act

### 4. Run on Robot
python eval_robot/eval_g1.py \
    --policy.path=outputs/train/.../pretrained_model \
    --arm=G1_29 \
    --ee=dex3 \
    --visualization=true


# 1.  Environment Setup

## 1.1  LeRobot Environment Setup

```bash
# Clone the source code
git clone --recurse-submodules https://github.com/unitreerobotics/unitree_lerobot.git

# If already downloaded:
git submodule update --init --recursive

# Create a conda environment
conda create -y -n unitree_lerobot python=3.10
conda activate unitree_lerobot
conda install pinocchio -c conda-forge

conda install ffmpeg=7.1.1 -c conda-forge

# Install LeRobot
cd unitree_lerobot/lerobot && pip install -e .

# Install unitree_lerobot
cd ../../ && pip install -e .
```

## 1.2  unitree_sdk2_python

For `DDS communication` on Unitree robots, some dependencies need to be installed. Follow the installation steps below:

```bash
git clone https://github.com/unitreerobotics/unitree_sdk2_python.git
cd unitree_sdk2_python  && pip install -e .
```

# 2.  Data Collection and Conversion

## 2.1  Load Datasets

If you want to directly load the dataset we have already recorded,
Load the [`unitreerobotics/G1_Dex3_ToastedBread_Dataset`](https://huggingface.co/datasets/unitreerobotics/G1_Dex3_ToastedBread_Dataset) dataset from Hugging Face. The default download location is `~/.cache/huggingface/lerobot/unitreerobotics`. If you want to load data from a local source, please change the `root` parameter.

```python
from lerobot.datasets.lerobot_dataset import LeRobotDataset
import tqdm

episode_index = 1
dataset = LeRobotDataset(repo_id="unitreerobotics/G1_Dex3_ToastedBread_Dataset")

from_idx = dataset.meta.episodes["dataset_from_index"][episode_index]
to_idx = dataset.meta.episodes["dataset_to_index"][episode_index]

for step_idx in tqdm.tqdm(range(from_idx, to_idx)):
    step = dataset[step_idx]
```

`visualization`

```bash
cd unitree_lerobot/lerobot

python src/lerobot/scripts/lerobot_dataset_viz.py \
    --repo-id unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --episode-index 0
```

## 2.2  Data Collection

If you want to record your own dataset. The open-source teleoperation project [avp_teleoperate](https://github.com/unitreerobotics/avp_teleoperate/tree/g1) can be used to collect data using the Unitree G1 humanoid robot. For more details, please refer to the [avp_teleoperate](https://github.com/unitreerobotics/avp_teleoperate/tree/g1) project.

## 2.3 Data Processing

```bash
conda activate unitree_lerobot
pip install PyQt5
```

Launch the data editor as below.

```bash
cd data_editor
python data_editor_EN.py
```

At first, click the button `Select Dataset Path`, choose the path of your dataset. For example, at the path tree below, `test_dataset/`is the path you need to choose.

```bash
test_dataset/
    ├── episode_0001
    ├── episode_0003
    ├── episode_0004
    ├── ...
```


## 2.4  Data Conversion

The data collected using [avp_teleoperate](https://github.com/unitreerobotics/avp_teleoperate/tree/g1) is stored in JSON format. Assuming the collected data is stored in the `$HOME/datasets/task_name`, the format is as follows

```
datasets/                               # Dataset folder
    └── task_name /                     # Task name
        ├── episode_0001                # First trajectory
        │    ├──audios/                 # Audio information
        │    ├──colors/                 # Image information
        │    ├──depths/                 # Depth image information
        │    └──data.json               # State and action information
        ├── episode_0002
        ├── episode_...
        ├── episode_xxx
```

### 2.4.1  Sort and Rename

When generating datasets for LeRobot, it is recommended to ensure that the data naming convention, starting from `episode_0`, is sequential and continuous. You can use the following script to `sort and rename` the data accordingly.

```bash
python unitree_lerobot/utils/sort_and_rename_folders.py \
        --data_dir $HOME/datasets/task_name
```

#### 2.4.2  Conversion

Convert `Unitree JSON` Dataset to `LeRobot` Format. You can define your own `robot_type` based on [ROBOT_CONFIGS](https://github.com/unitreerobotics/unitree_lerobot/blob/main/unitree_lerobot/utils/convert_unitree_json_to_lerobot.py#L154).

```bash
# --raw-dir     Corresponds to the directory of your JSON dataset
# --repo-id     Your unique repo ID on Hugging Face Hub
# --push_to_hub Whether or not to upload the dataset to Hugging Face Hub (true or false)
# --robot_type  The type of the robot used in the dataset (e.g., Unitree_Z1_Single, Unitree_Z1_Dual, Unitree_G1_Dex1, Unitree_G1_Dex3, Unitree_G1_Brainco, Unitree_G1_Inspire,Unitree_G1_Dex1_Sim)

python unitree_lerobot/utils/convert_unitree_json_to_lerobot.py \
    --raw-dir $HOME/datasets \
    --repo-id your_name/repo_task_name \
    --robot_type Unitree_G1_Dex3 \
    --push_to_hub
```

# 3.  Training

[For training, please refer to the official LeRobot training example and parameters for further guidance.](https://github.com/huggingface/lerobot/tree/main/docs/source)

- `Train Act Policy` [Please refer to it in detail](https://github.com/huggingface/lerobot/blob/main/docs/source/act.mdx)

```bash
cd unitree_lerobot/lerobot

python src/lerobot/scripts/lerobot_train.py \
    --dataset.repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --policy.push_to_hub=false \
    --policy.type=act
```

- `Train Diffusion Policy` [Please refer to it in detail](https://github.com/huggingface/lerobot/blob/main/docs/source/policy_diffusion_README.md)

```bash
cd unitree_lerobot/lerobot

python src/lerobot/scripts/lerobot_train.py\
    --dataset.repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --policy.push_to_hub=false \
    --policy.type=diffusion
```

- `Train Pi0 Policy` [Please refer to it in detail](https://github.com/huggingface/lerobot/blob/main/docs/source/pi0.mdx)

```bash
cd unitree_lerobot/lerobot

python src/lerobot/scripts/lerobot_train.py \
    --dataset.repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --policy.push_to_hub=false \
    --policy.type=pi0
```

- `Train Pi05 Policy` [Please refer to it in detail](https://github.com/huggingface/lerobot/blob/main/docs/source/pi05.mdx)

```bash
cd unitree_lerobot/lerobot

python src/lerobot/scripts/lerobot_train.py \
    --dataset.repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --policy.type=pi05 \
    --output_dir=./outputs/pi05_training \
    --job_name=pi05_training \
    --policy.pretrained_path=lerobot/pi05_base \
    --policy.compile_model=true \
    --policy.gradient_checkpointing=true \
    --policy.dtype=bfloat16 \
    --policy.device=cuda \
    --policy.push_to_hub=false
```

- `Train Gr00t Policy` [Please refer to it in detail](https://github.com/huggingface/lerobot/blob/main/docs/source/groot.mdx)

```bash
cd unitree_lerobot/lerobot

python src/lerobot/scripts/lerobot_train.py \
    --dataset.repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --output_dir=./outputs/groot_training \
    --policy.push_to_hub=false \
    --policy.type=groot \
    --policy.tune_diffusion_model=false \
    --job_name=groot_training
```

If you want to use multi-GPU training, please refer to the details [here](https://github.com/huggingface/lerobot/blob/main/docs/source/multi_gpu_training.mdx)

# 4.  Real-World Testing

To test your trained model on a real robot, you can use the eval_g1.py script located in the eval_robot folder. Here’s how to run it:

[To open the image_server, follow these steps](https://github.com/unitreerobotics/avp_teleoperate?tab=readme-ov-file#31-%EF%B8%8F-image-server)

```bash

# --policy.path: Specifies the path to the pre-trained model, used for evaluating the policy.
# --repo_id: The repository ID of the dataset, used to load the dataset required for evaluation.
# --root: The root directory path of the dataset, defaults to an empty string.
# --episodes: The number of evaluation episodes; setting it to 0 uses the default value.
# --frequency: The evaluation frequency (in Hz), used to control the time step of the evaluation.
# --arm: The model of the robotic arm, (e.g., G1_29, G1_23).
# --ee: The type of end-effector, (e.g., dex3, dex1, inspire1, brainco).
# --visualization: Whether to enable visualization; setting it to true enables it.
# --send_real_robot: Whether to send commands to the real robot.


python unitree_lerobot/eval_robot/eval_g1.py  \
    --policy.path=unitree_lerobot/lerobot/outputs/train/2025-03-25/22-11-16_diffusion/checkpoints/100000/pretrained_model \
    --repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --root="" \
    --episodes=0 \
    --frequency=30 \
    --arm="G1_29" \
    --ee="dex3" \
    --visualization=true \

If you want to run inference tests in the unitree_sim_isaaclab simulation environment, please execute:

# --save_data: Allows recording data while running inference. At present, this option is limited to the sim environment.
# --task_dir: the directory where data is stored
# --max_episodes: the maximum number of inference runs per task; if exceeded, the task is considered failed by default

python unitree_lerobot/eval_robot/eval_g1_sim.py  \
    --policy.path=unitree_lerobot/lerobot/outputs/train/2025-03-25/22-11-16_diffusion/checkpoints/100000/pretrained_model \
    --repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --root="" \
    --episodes=0 \
    --frequency=30 \
    --arm="G1_29" \
    --ee="dex3" \
    --visualization=true \
    --save_data=false \
    --task_dir="./data" \
    --max_episodes=1200

# If you want to evaluate the model's performance on the dataset, use the command below for testing
python unitree_lerobot/eval_robot/eval_g1_dataset.py  \
    --policy.path=unitree_lerobot/lerobot/outputs/train/2025-03-25/22-11-16_diffusion/checkpoints/100000/pretrained_model \
    --repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --root="" \
    --episodes=0 \
    --frequency=30 \
    --arm="G1_29" \
    --ee="dex3" \
    --visualization=true \
    --send_real_robot=false
```

# 5.  Replay Datasets On Robot

This section provides instructions on how to replay datasets on the robot.
It is useful for testing and validating the robot's behavior using pre-recorded data.

```bash

# --repo_id         Dataset repository ID on Hugging Face Hub (e.g., unitreerobotics/G1_Dex3_ToastedBread_Dataset)
# --root            Path to the root directory of the dataset (leave empty to use the default cache path)
# --episodes        Index of the episode to replay (e.g., 0 for the first episode)
# --frequency       Replay frequency in Hz (e.g., 30 for 30 frames per second)
# --arm             Type of robot arm used (e.g., G1_29, G1_23)
# --ee              Type of end-effector used (e.g., dex3, dex1, inspire1, brainco)
# --visualization   Enable or disable visualization during replay (true for enabling, false for disabling)

python unitree_lerobot/eval_robot/replay_robot.py \
    --repo_id=unitreerobotics/G1_Dex3_ToastedBread_Dataset \
    --root="" \
    --episodes=0 \
    --frequency=30 \
    --arm="G1_29" \
    --ee="dex3" \
    --visualization=true
```

# 6. Acknowledgement

This code builds upon following open-source code-bases. Please visit the URLs to see the respective LICENSES (If you find these projects valuable, it would be greatly appreciated if you could give them a star rating.):

1. https://github.com/huggingface/lerobot
2. https://github.com/unitreerobotics/unitree_sdk2_python
3. https://github.com/unitreerobotics/xr_teleoperate
4. https://github.com/unitreerobotics/unitree_sim_isaaclab
