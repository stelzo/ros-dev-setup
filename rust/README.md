# Setup

## General
Install Rust if you haven't already. You can check if you have it installed by running `rustc --version`.
```zsh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install cargo-binstall
```

Make sure you have `libclang-dev` installed for building bindings (bindgen). For Ubuntu just run `sudo apt install libclang-dev`, for MacOS you can install it with `brew install llvm`.

## ROS2 specific

```zsh
pip install colcon-ros-cargo
```
> [!NOTE]
> Use `--break-system-package` if you are not using a virtual environment.

```zsh
cargo binstall cargo-ament-build
```

Now you can build your Rust packages like every other ROS2 package.
```zsh
colcon build --packages-select <your_package>
```

For release builds, you can pass the flag to cargo (like you would with CMake).
```zsh
colcon build --packages-select <your_package> --cargo-args --release
```

# Create package

## Node quickstart

Messages
```zsh
mkdir lifis_msgs
curl -L https://github.com/stelzo/lifis_msgs/archive/refs/tags/v0.1.0.tar.gz | tar -xz -C lifis_msgs --strip-components=1
```


```zsh

```

Messages are filtered in the `.cargo/config.toml` file. You can add more messages there.

