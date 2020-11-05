# Simple install guide for ROCm and Tensorflow, designed for Ubuntu 20.4.1 LTS and Radeon RX580 
This guide will show you how to set up your clean **Ubuntu 20.4.1 LTS** system ready to run **Tensorflow** projects, using **ROCm** to take advantage of the power of your **RX580 graphics card (gfx803)** in a tested, easy and fast way (It should work on other supported Ubuntu versions or other graphic cards too, with only slight changes).

This is basically a resume of the [official guide by AMD](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html), and the [unofficial (and great) guide by Mathieu Poliquin](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html). I highly recommend to check those links, in order to understand what you are doing and why, since I wont explain every single command.

There is another important thing to notice: **In this guide we downgrade ROCm to 3.5.1, since there are some bugs in posterior versions which *have not been fixed yet, not at all*** (bugs [1](https://github.com/RadeonOpenCompute/ROCm/issues/1269), [2](https://github.com/RadeonOpenCompute/ROCm/issues/1265)). Errors like `/src/external/hip-on-vdi/rocclr/hip_code_object.cpp:120: guarantee(false && "hipErrorNoBinaryForGpu: Coudn't find binary for current devices!") Aborted (core dumped)` are fixed this way.

Lets get started!
## Install ROCm
```
sudo apt update
sudo apt dist-upgrade
sudo apt install libnuma-dev
sudo reboot
```
```
wget -q -O - http://repo.radeon.com/rocm/apt/3.5.1/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] http://repo.radeon.com/rocm/apt/3.5.1/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
sudo apt update
sudo apt install rocm-dkms && sudo reboot
```
```
sudo usermod -a -G video $LOGNAME
sudo usermod -a -G render $LOGNAME
sudo reboot
```
```
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin' | sudo tee -a /etc/profile.d/rocm.sh
sudo ldconfig
```
- [x] **Check your progress:** At this point you should be able to enter `rocminfo` in a terminal without any error. You should also see your video card name in the command output, something like this:
> Name:                    gfx803                             
> Uuid:                    GPU-XX                             
> Marketing Name:          Ellesmere [Radeon RX 470/480/570/570X/580/580X/590]

If you made it this far congragulations! you are half the way!
## Install Tensorflow
```
sudo apt install python3 python3-pip
sudo apt install rocm-libs miopen-hip
pip3 install tensorflow-rocm
sudo apt install rccl
sudo apt install libtinfo5
sudo reboot
```
- [x] **Check your progress:** At this point you should be able to import Tensorflow in a python interactive session, make a simple operation, and exit without any error. Try the following in a terminal:
```python
import tensorflow as tf
tf.add(2,5)
exit()
```
You should see something like this: `<tf.Tensor: shape=(), dtype=int32, numpy=7>`

If you are reading this then you should be glad of yourself because your machine is now ready to use tensorflow-rocm! Anyway you should consider testing it with something more complex, like a benchmark.
## Test with a benchmark, the last frontier.
```
sudo apt install git
git clone https://github.com/tensorflow/benchmarks
cd ./benchmarks/scripts/tf_cnn_benchmarks
python3 tf_cnn_benchmarks.py --num_gpus=1 --batch_size=32 --model=resnet50
```
It will then start the benchmark. Expect it to take some time, specially if it is the first time, but no more than 5-10 minutes. If you still get stuck in the warm up for more than that time then give it some more. After that it wont take that long again.

- [x] **Check your progress:** If you get any error that prevents the script from above to complete successfully you should pray God for mercy while you google your error, hopefully someone already solved it. If you see something like the output bellow, go and get yourself a taco and be happy, you f@#$%&g did it!
> total images/sec: 87.92

***Have fun! :D***
