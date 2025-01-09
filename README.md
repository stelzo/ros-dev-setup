# ROS Development Setup

This is a description how to setup my ROS environment. I make some assumptions like shell etc. based on my needs. Don't just blindly copy paste the commands if you don't have the same environment.

Python 3.12 now wants a virtual environment before installing anything with pip. I recommend using venv.
```zsh
cd $HOME
python3 -m venv python3_env
source $HOME/python3_env/bin/activate # to .zshrc
```

When installing system tools, venv may not be enough. We need to force:
```zsh
pip install --break-system-package <your-package>
```

# Setup via Distrobox [Linux only]

These versions are available. Change the commands depending on your needed ROS version.
- ROS2 Jazzy
- ROS1 Melodic

Install distrobox and podman. Example with Ubuntu >=22.10. 
```zsh
sudo apt install distrobox podman
```

Init podman
```zsh
podman machine init
podman machine start
```

If you get an error that gvproxy is not installed, do it with `curl -L 'https://github.com/containers/gvisor-tap-vsock/releases/download/v0.8.1/gvproxy-linux-amd64' -o gvproxy`, `chmod +x gvproxy`, `sudo mv gvproxy /usr/libexec/podman/gvproxy`.

Create the environment container. Add `--nvidia` if you have a nvidia graphics card.
```zsh
distrobox create --image ubuntu:24.04 --name jazzy
```

Enter and run the install script for the environment. If [my dev setup](https://github.com/stelzo/dev) is used, you can just type `init_jazzy_image.sh` anywhere.
```zsh
distrobox enter jazzy
curl -s https://raw.githubusercontent.com/stelzo/dev/refs/heads/main/bin/init_jazzy_image.sh | bash
```

In the future you should enter the container like this. You may use an alias for that.
```zsh
distrobox enter jazzy -- zsh -c "source /opt/ros/jazzy/setup.zsh && exec zsh"
```

> [!NOTE]
> The upper command is already shortened in my dev environment and can be called by just typing `jazzy` anywhere.

For Rust, follow the instructions in [this](./rust/README.md) section. The script already installs the toolchains.

## My Best practices

- `mkdir -p ~/projects/ros2_ws/src/thirds` Third-party ROS2 packages like drivers, etc. Build them from source where possible. If building is too complicated, look for alternatives or build them yourself - bloat is bad, hiding it is worse.
- `mkdir -p ~/projects/ros2_ws/src` All ROS2 packages that I develop.
- `mkdir -p ~/projects/thirds` Third-party libraries from source, Pull Requests, etc. 
- `mkdir -p ~/projects` ROS-independent personal projects, experiments and libraries.
- Create complex logic as libraries and use ROS only as a communication layer or frontend. State Management Nodes like State Machines or Behavior Trees are the exception.
- Configuration is a logic-level concern. ROS has APIs to get them, but you give it via parameters to the library (e.g. structs). 
- Remove dependencies as much as possible without adding a huge maintenance burden. Copy code if only a small part is needed and avoid large libraries in general. Any step beyond `git clone` should be avoided or automated.

### Messages
... are language independent descriptions like meta-packages. I use CMake for them, since C++ toolchain is installed with ROS anyways and people may want to use the message but not the Rust package.

Messages use their own repository. This way they can be shared easily with other nodes via a simple `git clone`. [Here](https://github.com/stelzo/lifis_msgs) is an example repository used in one of my Rust package.

## Managing this setup

Export a new image with 
```zsh
podman container commit -p jazzy <your-image-name>
podman save <your-image-name>:latest | bzip2 > <your-image-name>.tar.bz
```

### ROS1 -> ROS2

For comparing old vs new methods, the datasets should be the same. Bagfiles can be converted with
```bash
pip3 install rosbags --break-system-packages
rosbags-convert --src <bagfile>.bag --dst <outputfolder>
```

