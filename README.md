# Replication of "KING: Generating Safety-Critical Driving Scenarios for Robust Imitation via Kinematics Gradients"

This is a modified version of the [original project](https://lasnik.github.io/king/) to support the following:
- Compatible with CARLA 0.9.14
- Easy visualisation (gif generation)
- Minor fixes

## Requirements

### Hardware
- GPU: NVIDIA Tesla T4
- Memory: 16GB+
- Storage: 150GB+

### Software
- Ubuntu 20.04
- nvidia driver

## Setup
Clone the repo
```Shell
git clone https://github.com/ADS-Testing/king.git
cd king
```

### Environment
Install drivers and reboot. If the appropriate version of the driver is already installed(Check with the command `nvidia-smi`), you can skip this step.
```Shell
sudo apt update
sudo apt install ubuntu-drivers-common
sudo ubuntu-drivers autoinstall
sudo reboot
```

Install anaconda and build the environment.
```Shell
wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh
bash Anaconda3-2022.05-Linux-x86_64.sh
source ~/.profile
conda env create -f environment.yml
conda activate king
```

### Carla
Download and setup CARLA 0.9.14.
```Shell
chmod +x setup_carla.sh
./setup_carla.sh
```

### Transfuser
To generate scenarios for [TransFuser](https://github.com/autonomousvision/transfuser), you need to download the model weights:
```Shell
mkdir -p driving_agents/king/transfuser/model_checkpoints/regular
cd driving_agents/king/transfuser/model_checkpoints/regular
wget https://s3.eu-central-1.amazonaws.com/avg-projects/transfuser/models.zip
unzip models.zip
rm -rf models.zip late_fusion geometric_fusion cilrs aim
cd -
```

## How to run

### Scenario Generation
We provide bash scripts for the experiments for convenience. Please make sure the "CARLA_ROOT" ("./carla_server" by default) and "KING_ROOT" (if present) environment variables are set correctly in all of those scripts.

#### Running the code
For all of the generation scripts, first spin up a carla server in a separate shell:
```Shell
carla_server/CarlaUE4.sh --world-port=2000 -RenderOffScreen
```
If you cannot separate the shell, execute the script in the background.
```Shell
nohup carla_server/CarlaUE4.sh --world-port=2000 -RenderOffScreen &
```
Following scripts will run generation and automatically evaluate the results.

##### AIM-BEV generation
For AIM-BEV generation, run:
```Shell
bash run_generation.sh
```

##### AIM-BEV generation using both gradient paths
For AIM-BEV generation using both gradient paths, run:
```Shell
bash run_generation_both_paths.sh
```

##### TransFuser generation
For Transfuser generation using both gradient paths, run:
```Shell
bash run_generation_transfuser.sh
```

#### Getting results
```Shell
generation_results/
├── agents_1
│   ├── RouteScenario_112_to_112
│   │   ├── results.json
│   │   └── scenario_records.json
│   ├── RouteScenario_114_to_114
│   │   ├── results.json
│   │   └── scenario_records.json
│   ...
│   ├── opt.pkl
│   └── opt.txt
├── agents_2
│   ├── RouteScenario_112_to_112
│   │   ├── results.json
│   │   └── scenario_records.json
│   ...
│   ├── opt.pkl
│   └── opt.txt
└── agents_4
    ├── RouteScenario_112_to_112
    │   ├── results.json
    │   └── scenario_records.json
    ...
    ├── opt.pkl
    └── opt.txt
```

### Scenario Visualization

#### Running the code
First spin up a carla server in a separate shell:
```Shell
carla_server/CarlaUE4.sh --world-port=2000 -RenderOffScreen
```
After providing the directory name you want to visualize as an argument, run the following script. The default directory is set to "generation_results".
```Shell
bash run_visualization.sh generation_results_transfuser
```

#### Getting results
```Shell
generation_results_transfuser/
└── agents_4
    ├── RouteScenario_112_to_112
    │   ├── RouteScenario_112_iter_4.gif
    │   ├── results.json
    │   └── scenario_records.json
    ├── RouteScenario_114_to_114
    │   ├── RouteScenario_114_iter_1.gif
    │   ├── results.json
    │   └── scenario_records.json
    ...
    ├── opt.pkl
    └── opt.txt
```

### Fine-tuning
To fine-tune the original agent on KING scenarios, first download the regular data for AIM-BEV:
```
chmod +x download_regular_data.sh
./download_regular_data.sh
```
Then run the fine-tuning script (and adjust it for the correct dataset path, if necessary):
```
bash run_fine_tuning.sh
```
This script also automatically runs evaluation on KING scenarios.

### Town10 Intersections 
To evaluate the original checkpoint for AIM-BEV on the Town10 intersections benchmark, spin up a carla server and run
```Shell
leaderboard/scripts/run_evaluation.sh
```
To evaluate a fine-tuned model, run the fine-tuning script above and toggle the commented "TEAM_CONFIG" variable in the evaluation script to change the model weights.

## Troubleshooting
- [Carla terminates immediately](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-Carla-terminates-immediately)
- [CondaValueError: prefix already exists:](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-CondaValueError:-prefix-already-exists:)
- [ego_bp = self.bp_library.filter("vehicle.lincoln.mkz2017")[0] IndexError: index out of range](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-ego_bp-=-self.bp_library.filter(%22vehicle.lincoln.mkz2017%22)%5B0%5D-IndexError:-index-out-of-range)
- [error while loading shared libraries: libomp.so.5](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-error-while-loading-shared-libraries:-libomp.so.5)
- [error: command 'gcc' failed with exit status 1](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-error:-command-'gcc'-failed-with-exit-status-1)
- [Exception thrown: bind: Address already in use](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-Exception-thrown:-bind:-Address-already-in-use)
- [FileNotFoundError: [Errno 2] No such file or directory: 'aplay'](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-FileNotFoundError:-%5BErrno-2%5D-No-such-file-or-directory:-'aplay')
- [ImportError: cannot import name 'TrafficLightState' from 'carla' (unknown location)](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-ImportError:-cannot-import-name-'TrafficLightState'-from-'carla'-(unknown-location))
- [ModuleNotFoundError: No module named 'agents.navigation.global_route_planner_dao'](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-ModuleNotFoundError:-No-module-named-'agents.navigation.global_route_planner_dao')
- [No space left on device](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-No-space-left-on-device)
- [Unable to install gcc](https://github.com/ADS-Testing/Main/wiki/%5BKing%5D-Unable-to-install-gcc)

## Acknowledgements
Related pages:
- [Project Page](https://lasnik.github.io/king/) 
- [Paper](https://arxiv.org/pdf/2204.13683.pdf) 
- [Supplementary](https://lasnik.github.io/king/data/docs/KING_supplementary.pdf)

This implementation is based on code from several repositories.
- [CARLA Leaderboard](https://github.com/carla-simulator/leaderboard)
- [Scenario Runner](https://github.com/carla-simulator/scenario_runner)
- [Learning by Cheating](https://github.com/dotchen/LearningByCheating)
- [World on Rails](https://github.com/dotchen/WorldOnRails)

## Authors

### Originally written by
```bibtex
@inproceedings{Hanselmann2022ECCV,
  author = {Hanselmann, Niklas and Renz, Katrin and Chitta, Kashyap and Bhattacharyya, Apratim and Geiger, Andreas},
  title = {KING: Generating Safety-Critical Driving Scenarios for Robust Imitation via Kinematics Gradients},
  booktitle = {European Conference on Computer Vision(ECCV)},
  year = {2022}
}
```

### Replicated and modified by
- [Taehyeon An](https://dev.paxtaeo.com/en)
- [Kyungwook Nam](https://github.com/nkwook)
- Advised by [Donghwan Shin](https://dshin.info)

## License
[MIT License](LICENSE)
