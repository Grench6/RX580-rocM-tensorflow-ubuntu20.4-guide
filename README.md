# Simple install guide for ROCm and Tensorflow, designed for Ubuntu 20.4.1 LTS and Radeon RX580 
This guide will show you how to set up your clean **Ubuntu 20.4.1 LTS** system ready to run **Tensorflow** projects while using **ROCm** to take advantage of the power of your **RX580 graphics card (gfx803)** in an easy and fast way. (It should work on other supported Ubuntu versions or other graphic cards too, with only slight changes).

It is basically a resume of the official https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html by AMD, and the unofficial https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html by Mathieu Poliquin. The biggest change is the **downgrade of ROCm to 3.5.1 since there are some bugs and crashes in posterior versions which *have not been fixed yet***.

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

If you are reading this then you should be glad of yourself because you did it! your machine is ready to use tensorflow-rocm!
## Test with a benchmark
```
sudo apt install git
git clone https://github.com/tensorflow/benchmarks
cd ./benchmarks/scripts/tf_cnn_benchmarks
python3 tf_cnn_benchmarks.py --num_gpus=1 --batch_size=32 --model=resnet50
```
It will then start the benchmark. Expect it to take some time, specially if it is the first time, but no more than 5-10 minutes. If you still get stuck in the warm up for more than that time then give it some more. After that it wont take that long again.

At the end of the output you will see this with the performance of your graphic card:
> total images/sec: 87.92

***Have fun! :D***
