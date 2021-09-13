# Installation of ROCm and TensorFlow on Ubuntu 20.4 LTS for Radeon RX580 
**2021.9.13: Imrpoved kernel installation instructions for more clarity.**

This guide will show you how to set up your **fresh Ubuntu 20.4 LTS** OS to be ready to run **TensorFlow** projects, using **ROCm** to take advantage of the power of your **RX580 graphics card (or any gfx803)** in a tested, easy and fast way (It should work on other supported Ubuntu versions and other graphic cards too, with only slight changes).

It is basically a resume of the [official guide by AMD](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html), and the [unofficial guide by Mathieu Poliquin](https://www.videogames.ai/Install-ROCM-Machine-Learning-AMD-GPU). I highly recommend to check those links, in order to understand what you are doing and why.

There is another important thing to notice: **In this guide we downgrade ROCm to 3.5.1**. There are two reasons for this:
1. There are some bugs in posterior versions which have not been fixed yet, *not at all* (bugs: [1](https://github.com/RadeonOpenCompute/ROCm/issues/1269), [2](https://github.com/RadeonOpenCompute/ROCm/issues/1265)). 
2. AMD dropped support for the RX580 ([1](https://github.com/RadeonOpenCompute/ROCm/issues/1353)).

So, in the case you have a newer version already installed, you will have to remove it first.

Lets get started!
## Remove ROCm (you can skip this step if you don't have ROCm installed)
1. ```sudo apt autoremove rocm-dkms```
2. Make sure that all packages are removed under /opt/rocm-xxx
3. Check ```sudo dpkg -l | grep hsa``` (replace ```hsa``` with ```hip```, ```llvm```, ```rocm``` and ```rock```). Make sure that all packages are removed with ```sudo apt purge``` . I recommend to do the same for any other additional packages (if you installed anything explicitly).
*I have occasionally removed the GUI for Ubuntu, so be careful!*
4. Reboot the system

**It's preferrable to do a fresh Ubuntu reinstall instead of removing ROCm (strange bugs may occur).**
## Install the kernel
ROCm requieres the kernel version 5.4 (you can check your currently running kernel version with `uname -r`).

1. Install the requiered kernel: `sudo apt install --install-recommends linux-generic`
2. Reboot your computer. In the GRUB menu choose *"Additional options for Ubuntu"* and select *"Boot with kernel 5.4.0-x-generic"*
3. Remove the other kernels: `sudo apt remove --purge linux-generic-hwe-20.04 linux-oem-20.04 linux-hwe-* linux-oem-* linux-modules-5.1* linux-modules-5.8.0-* linux-modules-5.6.0-* `

At this point, if you check your currently running kernel you should see something like *"5.4.0-x-generic"*.

## Install ROCm 3.5.1
```
sudo apt update
sudo apt dist-upgrade
sudo apt install libnuma-dev
sudo reboot
```

Add the repo and install rocm-dkms:
```
wget -q -O - https://repo.radeon.com/rocm/apt/debian/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] http://repo.radeon.com/rocm/apt/3.5.1/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
sudo apt update
sudo apt install rocm-dkms && sudo reboot
```
Set user permissions to access video card features:
```
sudo usermod -a -G video $LOGNAME
sudo usermod -a -G render $LOGNAME
sudo reboot
```
Add rocm to PATH:
```
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin' | sudo tee -a /etc/profile.d/rocm.sh
sudo ldconfig
sudo reboot
```
- [x] **Check your progress:** At this point you should be able to enter `rocminfo` into a terminal without getting any error. You should also see your video card name in the command output (something like this):
> Name:                    gfx803                             
> Uuid:                    GPU-XX                             
> Marketing Name:          Ellesmere [Radeon RX 470/480/570/570X/580/580X/590]

You are half the way now!
## Install Tensorflow
```
sudo apt install python3 python3-pip
sudo apt install rocm-libs miopen-hip
pip3 install -Iv tensorflow-rocm==2.2.0
sudo apt install rccl
sudo apt install libtinfo5
sudo reboot
```
- [x] **Check your progress:** At this point you should be able to import Tensorflow in Python, make a simple operation, and exit without any error. Try the following in a Python interactive session:
```python
import tensorflow as tf
tf.add(2,5)
exit()
```
You should see something like this: 
> <tf.Tensor: shape=(), dtype=int32, numpy=7>

- If you see an error:
  ```
  Could not load dynamic library 'libhip_hcc.so'; dlerror: libhip_hcc.so: cannot open shared object file: No such file or directory
  ```
  You could use any workaround from [this issue](https://github.com/RadeonOpenCompute/ROCm/issues/1163). Here is one:
  ```
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/rocm/hip/lib
  sudo ldconfig
  ```
  If this fix works, you will need to [permanently set the variable](https://askubuntu.com/questions/887442/how-to-permanently-set-an-environment-variable).

Congrats! Your machine is now ready to use tensorflow-rocm! You should still consider testing it with something more complete, like a benchmark.
## Test with a benchmark
```
sudo apt install git
git clone https://github.com/tensorflow/benchmarks
cd ./benchmarks/scripts/tf_cnn_benchmarks
python3 tf_cnn_benchmarks.py --num_gpus=1 --batch_size=32 --model=resnet50
```
Expect it to take some time (5-10 minutes). If you think you got stuck in the warm up, be patient, it should not take that long the next time.

If you see something like the output bellow (numbers may vary a lot, someone got 36.56) then go and get yourself a taco 🌮, you did it!
> total images/sec: 87.92

***Have fun! :D***

-------
# Some additional stuff

## HIP Installation
**HIP** allows developers to convert CUDA code to portable C++. The same source code can be compiled to run on NVIDIA or AMD GPUs. HIP code can be developed either on AMD ROCm platform using HIP-Clang compiler, or a CUDA platform with NVCC installed. **HIP-Clang** is the compiler for compiling HIP programs on AMD platform (**Clang** as front end, **LLVM** as its back end).

If you want to compile aplications/samples using HIP or if you are having issues with Clang (not being installed, or being incorrectly installed), then [check this link](https://rocmdocs.amd.com/en/latest/Installation_Guide/HIP-Installation.html#amd-platform) for detailed instructions on installing/building HIP-Clang.
