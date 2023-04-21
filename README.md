# FogROS2 SGC Lite

FogROS2-SGC is a cloud robotics platform for connecting disjoint ROS2 networks across different physical locations, networks, and Data Distribution Services. 

\[[Website](https://sites.google.com/view/fogros2-sgc)\] \[[Video](https://youtu.be/hVVFVGLcK0c)\] \[[Arxiv](https://arxiv.org/abs/2210.11691)\] (TODO: arxiv link)

### Why Lite version 

The FogROS2-SGC carries a bag of protocols to support heterogenous demands and requirements. In this version, we build everything with webrtc protocol to streamline the routing setup. Webrtc is generally not compatible with the previous protocols. As a result, we make a lite version with only webrtc version. 

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [From SGC to SGC-lite](#from-sgc-to-sgc-lite)
    - [Making your own signaling server](#making-your-own-signaling-server)
- [Local Demo](#local-demo)
- [Build FogROS2 SGC](#build-fogros2-sgc)
  - [Install dependencies](#install-dependencies)
    - [Install Rust](#install-rust)
    - [Install ROS](#install-ros)
    - [Build the repo](#build-the-repo)
- [Run with Different Machines](#run-with-different-machines)
    - [Certificate Generation](#certificate-generation)
    - [Run with Environment Variables](#run-with-environment-variables)
    - [Run ROS2 talker and listener](#run-ros2-talker-and-listener)
  - [TODOs and Known issues](#todos-and-known-issues)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## From SGC to SGC-lite
The updated README file can be found [HERE](./src/resources/README.md), which we removed all the setup about protocols and gateways. Only one change on the signaling server field `signaling_server_address = "ws://128.32.37.42:8000"` is required to the old config file. The default signaling server is provided by Berkeley, but feel free to make your own server by the following instructions.

(Note: remember to migrate the credentials from `./scripts` to the new repo as well!)


#### Making your own signaling server
Signaling server faciliates the communication by exchanging the address information of webrtc. The details about how signaling server works can be found [HERE](./docs/webrtc.md).
```
git clone https://github.com/data-capsule/libdatachannel.git
cd libdatachannel/examples/signaling-server-rust/
cargo run
```

#### TODOs 
1. we assume the publishers start before and subscriber, and subscriber retry if the publisher's info does not exist. We may find a more clever way of handling this. 
2. video streaming
3. testing on raspberrry pi

## Local Demo 
If you want to get a taste of FogROS2 SGC without setting up the environment, just run 
```
docker compose build && docker compose up 
```
with docker([install](https://docs.docker.com/get-docker/)) and docker compose([install](https://docs.docker.com/compose/install/linux/)). 
It takes some time to build. You will see two docker containers running `talker` and `listener` are connected securely with FogROS2-SGC.



## Build FogROS2 SGC 
The following are instructions of building FogROS2 SGC. 

### Install dependencies 
```
sudo apt update
sudo apt install build-essential curl pkg-config libssl-dev protobuf-compiler clang
```

#### Install Rust 
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -y
source "$HOME/.cargo/env"
```

#### Install ROS 
ROS2 ~Dashing~ ~Eloquent~ Foxy Galactic Humble Rolling should work fine with FogROS2 SGC. 

Here we show the instruction of installing ROS2 rolling with Debian packages. 

First, install ROS2 from binaries with [these instructions](https://docs.ros.org/en/rolling/Installation/Ubuntu-Install-Debians.html).

Setup your environment with [these instructions](https://docs.ros.org/en/rolling/Installation/Ubuntu-Install-Debians.html#environment-setup).

Every terminal should be configured with 
```
source /opt/ros/rolling/setup.bash
````

#### Build the repo 

The repo is built with 
```
cargo build
```

## Run with Different Machines
In the example, we use two machines to show talker(machine A) and listener(machine B) example. 

#### Certificate Generation
The certificates can be generated by 
```
cd scripts
./generate_crypto.sh
```
Every directory in `./scripts/crypto` contains the cryptographic secrets needed for communication. 

Distribute the `crypto` directory by from machine A and machine B. Here is an example with `scp`: 
```
scp -r crypto USER@MACHINE_B_IP_ADDR:/SGC_PATH/scripts/
```
replace `USER`, `MACHINE_B_IP_ADDR`, `SGC_PATH` with the actual paths.

After the crypto secrets are delivered, go back to project main directory. 

#### Run with Environment Variables 
Run FogROS2-SGC routers on the root project directory. 
On the machine A
```
export SGC_CONFIG=talker.toml
cargo run router
```
On the machine B
```
export SGC_CONFIG=listener.toml
export GATEWAY_IP=MACHINE_A_IP
cargo run router
```
Replace `MACHINE_A_IP` with the IP address of Machine A. 

Note that A and B can configure with some intermediate machine C if they are not able to directly connect. Then configure `GATEWAY_IP` on both machines with machine C's ip address. 

The talker and listener toml configuration file can be found [here](./src/resources/README.md).


To disable the logs and run with benchmark mode, run with `release` option by 
```
cargo run --release
```

#### Run ROS2 talker and listener
Now run talker and listener on ROS2. 
```
# Machine A: 
ros2 run demo_nodes_cpp talker
```
and 
```
# Machine B
ros2 run demo_nodes_cpp listener
```

### TODOs and Known issues 
1. automatic topic discovery
2. segmentation fault / node creation failure: not caused by our project but our underlying framework or ROS rcl itself. Restart the program and the problem should be fixed. The hypothesis is asynchronous error when the nodes are created too fast in parallel. 

