# Unity ML-Agents for python mlagents API without X server
## Background
&ensp;&ensp;As we know, the environments of Unity ML-Agents are always along with GUI, which are hardly to run in the servers without X server when we don't have root access. Here I find a way to run ML-Agents with two tools, [Proot](https://proot-me.github.io/) and Xvfb.

## Environment
- A server without X server and root access, mine is one which provides Jupyter Lab only.

## Preparations
1. [Ubuntu rootfs](https://drive.google.com/file/d/1tUv06pQkeH-GTSIhlKJJrd4kEIxlNGD8/view?usp=sharing) where Xvfb is installed. 
2. ML-Agents environment binary for testing.
3. [Static proot binary](https://github.com/proot-me/proot-static-build/releases/download/v5.1.1/proot_5.1.1_x86_64_rc2).

## Guide
1. Upload ubuntu rootfs to server and uncompress.
2. Upload ML-Agents environment to server and uncompress to a directory in rootfs.
3. Download proot binary to server, rename to proot for convenience.
4. Run the command below to start Xvfb with display :1. The argument `-b /dev` means to bind /dev to the root of rootfs, as well as `-r ./ubuntu-rootfs/` means use ./ubuntu-rootfs/ as new root. Next is the command you want to run.

`./proot -b /dev -b /proc -r ./ubuntu-rootfs/ Xvfb :1 -screen 0 800x800x16`

5. New a file to customize our own ML-Agents script for python mlagents API as follows. Such as a file named 'myrollerball.x86_64'. The suffix of file is significant for python mlagents API to identify. The flag PROOT_NO_SECCOMP will received by proot, which means the kernel of my server is under SECCOMP mode. Without this flag, mine will raise error like 'proot info: pid 4349: terminated with signal 11'. When python mlagents API run this script, mlagents will pass two arguments, `--mlagents-port` and `{the port you used}`, which we receive with `$1` and `$2` and pass to real ML-Agents binary.
```shell
#!/bin/sh
export PROOT_NO_SECCOMP=1
export DISPLAY=:1
./proot -b /proc -b /dev -r ./ubuntu-rootfs/ /linux/rollerball.x86_64 $1 $2
```

6. Use python mlagents API to connect with ML-Agents environment as follows.
```python
from mlagents_envs.environment import UnityEnvironment
from mlagents_envs.side_channel.engine_configuration_channel import EngineConfig, EngineConfigurationChannel

env_name = './myrollerball.x86_64'

engine_configuration_channel = EngineConfigurationChannel()
env = UnityEnvironment(file_name=env_name, base_port=5004)
env.reset()
```
7. Enjoy it : ).
