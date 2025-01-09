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
> Use `--break-system-packages` if you are not using a virtual environment.

```zsh
cargo binstall cargo-ament-build
```

Now you can build your Rust packages like every other ROS2 package.
```zsh
colcon build --packages-select <your_package>
```

For release builds, you can pass the flag to Cargo (like you would with CMake).
```zsh
colcon build --packages-select <your_package> --cargo-args --release
```

# Create a new package

The ros2 CLI does not support creating Cargo packages yet. We will use a simple node to get started.

```zsh
mkdir -p ~/ros2_ws/src/<your-node-name>
cd ~/ros2_ws/src/
curl -L https://github.com/stelzo/imu-calib-ros/archive/refs/tags/v0.1.0.tar.gz | tar -xz -C <your-node-name> --strip-components=1
```

Open the <your-node-name> folder in your Code editor and change the node name in `package.xml` and `Cargo.toml`. Then investigate the source code to see a simple publish-subscribe-parameter-node. For more advanced topics, look at the [r2r examples](https://github.com/sequenceplanner/r2r/tree/master/r2r/examples).

I use r2r for building ROS nodes. When using messages, it is recommended to filter them based on your needs. Else r2r will build all messages that are sourced, which just needs time but provides no additional value to you. 
See the `.cargo/config.toml` file. You can add more messages there. If you need a message type, add it in here and in `package.xml`.

You can copy subfolders from your package like launch files or configs to the share directory with this added to your `Cargo.toml`.

```toml
[package.metadata.ros]
install_to_share = ["launch"]
```

> [!NOTE]
> Symlinks are not supported yet, so you will need to rebuild the package if you change anything in your installed files.

## Async vs Threads

The r2r crate returns futures for async programming. In Rust, async code allows a single thread to handle tasks concurrently. Async is nice for io heavy applications. However, in ROS, I personally favor threads for 2 reasons:

* No additional runtime, less dependencies, faster builds
* Non-blocking in processing-heavy pipelines

It is also easier for Rust newbies to understand.

To use threads, we need to call `block_on` on a minimal async implementation, so r2r can use the `await` keyword inside. So we actively block the current thread when awaiting, therefore nullifying async completely. But we still need to manage our processing pipeline with threads, for example with channels or `Arc<Mutex<T>>`.
