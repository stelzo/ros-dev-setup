# ROS Development Setup (official)
See https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html.

Python 3.12 now wants a virtual environment before installing anything with pip.
```zsh
cd $HOME
python3 -m venv python3_env
source $HOME/python3_env/bin/activate # to .zshrc
```

But for system tools this does not work. We need to force:
```zsh
pip install --break-system-package <your-package>
```

# ROS Development Setup (via Mamba)

This repository contains instructions to setup an environment for ROS2 for Linux and MacOS. It is my personal setup for researching and developing on my machines, not a general guide for all use cases. On Robots ("production"), I would recommend running a native minimal installation without any third-party tooling around it.

## Prerequisites

Install `conda`: https://docs.anaconda.com/miniconda/

Install `mamba`
```zsh
conda install mamba -c conda-forge
mamba init zsh
```

## Install ROS2 (Humble)
```bash
mamba create -n humble_env python=3.12
mamba activate humble_env
conda config --env --add channels conda-forge
conda config --env --add channels robostack-staging

# this might return an error if it is not in the list which is ok
conda config --env --remove channels defaults

mamba install ros-humble-desktop
mamba install compilers cmake pkg-config make ninja colcon-common-extensions catkin_tools rosdep

mamba deactivate # reactivate to initialize the shell properly

# useful commands
mamba activate humble_env
mamba update --all
mamba remove humble_env # undo everything
```

For Rust, follow the instructions in [this](./rust/README.md) section.

Build
```zsh
colcon build --cmake-args -DPython3_EXECUTABLE=$(which python) -DCMAKE_C_COMPILER=/usr/bin/cc -DCMAKE_CXX_COMPILER=/usr/bin/c++ -DPython3_NumPy_INCLUDE_DIRS=$(python -c "import numpy; print(numpy.get_include())")
```

when you want to `cargo build`
```zsh
source install/local_setup.sh
export DYLD_LIBRARY_PATH=$CONDA_PREFIX/lib:$DYLD_LIBRARY_PATH
```

## (My) Best practices

- `mkdir -p ~/projects/humble_ws/src/thirds` Third-party ROS2 packages like drivers, etc. Build them from source to get a feeling how big the shoulders are you are standing on. If building is too complicated, look for alternatives or build it yourself - bloat is bad, hiding it is worse.
- `mkdir -p ~/projects/humble_ws/src` All ROS2 packages that I develop.
- `mkdir -p ~/projects/thirds` Third-party libraries from source, Pull Requests, etc. 
- `mkdir -p ~/projects` ROS-independent personal projects, experiments and libraries.
- Create complex logic as libraries and use ROS only as a communication layer or frontend. State Management Nodes like State Machines or Behavior Trees are the exception.
- Configuration is a logic-level concern. ROS has APIs to get them, but you give it via parameters to the library (e.g. structs). 
- Remove dependencies as much as possible without adding a huge maintenance burden. Copy code if only a small part is needed and avoid large libraries in general. Any step beyond `git clone` should be avoided or automated.

## Messages
... are language independent descriptions like meta-packages. I use CMake for them, since C++ toolchain is installed with ROS anyways and people may want to use the message but not the Rust package.

Messages use their own repository. This way they can be shared easily with other nodes via a simple `git clone`. [Here](https://github.com/stelzo/lifis_msgs) is an example repository used in one of my Rust package.

# TODO curl example tar.gz

## vcstool
A package is a repository. This makes a workspace just a collection of repositories. Use `vcstool` to manage them.

# TODO show vcstool
