# NVIDIA_Tesla-P4_Devices
适用于 Linux Ubuntu 22.04 x86_64 安装Tesla-P4的显卡Cuda和驱动

# 下载适用于 Linux Ubuntu 22.04 x86_64 的安装程序
### 基础安装程序
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.3.0/local_installers/cuda-repo-ubuntu2204-12-3-local_12.3.0-545.23.06-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2204-12-3-local_12.3.0-545.23.06-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2204-12-3-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-3
```
### 驱动安装程序
```bash
sudo apt-get install -y cuda-drivers
sudo apt-get install -y nvidia-kernel-open-545
sudo apt-get install -y cuda-drivers-545
```
最后完成重启```sudo reboot```

重启后再次运行```nvidia-smi``` 可以看到显卡信息。

```bash
d@d:~$ nvidia-smi
Fri Nov  3 12:46:16 2023
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 545.23.06              Driver Version: 545.23.06    CUDA Version: 12.3     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  Tesla P4                       On  | 00000000:03:00.0 Off |                  Off |
| N/A   38C    P8               6W /  75W |      0MiB /  8192MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```

测试Jellyfin开启硬件加速：
```bash
d@d:/fuck$ nvidia-smi
Fri Nov  3 12:54:25 2023
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 545.23.06              Driver Version: 545.23.06    CUDA Version: 12.3     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  Tesla P4                       On  | 00000000:03:00.0 Off |                  Off |
| N/A   41C    P0              25W /  75W |    197MiB /  8192MiB |      5%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      8975      C   /usr/lib/jellyfin-ffmpeg/ffmpeg             195MiB |
+---------------------------------------------------------------------------------------+
```
已成功视频硬解

### Tesla GPU 如何关闭 ECC

Tesla系列GPU默认开启了ECC(error correcting code, 错误检查和纠正)功能，该功能可以提高数据的正确性，随之而来的是可用内存的减少和性能上的损失。

ECC内存支持：P4支持ECC校验，开启后会损失一部分显存

开启过后，显存可用为7611MB，关闭后可用为8121MB。

通过命令 nvidia-smi可以看到设备上所有GPU的信息参数

通过 nvidia-smi -i n -e 0/1 可关闭(0)/开启(1)  第n号GPU的编号。

执行```nvidia-smi | grep Tesla```查看前面GPU编号：0
```bash
d@d:/fuck$ nvidia-smi | grep Tesla
|   0  Tesla P4                       On  | 00000000:03:00.0 Off |                  Off |
```

执行关闭ECC```sudo nvidia-smi -i 0 -e 0```重启后该设置生效。

**值得注意的是开启ECC关闭ECC这个操作是有寿命的，开关几千次就不能再继续开关了。**
