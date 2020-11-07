# Installation of ROCm and Tensorflow on Ubuntu 20.4.1 LTS for Radeon RX580 
This guide will show you how to set up your **clean Ubuntu 20.4.1 LTS** OS to be ready to run **Tensorflow** projects, using **ROCm** to take advantage of the power of your **RX580 graphics card (or any gfx803)** in a tested, easy and fast way (It should work on other supported Ubuntu versions and other graphic cards too, with only slight changes).

It is basically a resume of the [official guide by AMD](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html), and the [unofficial guide by Mathieu Poliquin](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html). I highly recommend to check those links, in order to understand what you are doing and why.

There is another important thing to notice: **In this guide we downgrade ROCm to 3.5.1 since there are some bugs in posterior versions which have not been fixed yet, *not at all*** (bugs: [1](https://github.com/RadeonOpenCompute/ROCm/issues/1269), [2](https://github.com/RadeonOpenCompute/ROCm/issues/1265)).

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
- [x] **Check your progress:** At this point you should be able to enter `rocminfo` into a terminal without getting any error. You should also see your video card name in the command output (something like this):
> Name:                    gfx803                             
> Uuid:                    GPU-XX                             
> Marketing Name:          Ellesmere [Radeon RX 470/480/570/570X/580/580X/590]

You are half the way now!
## Install Tensorflow
```
sudo apt install python3 python3-pip
sudo apt install rocm-libs miopen-hip
pip3 install tensorflow-rocm
sudo apt install rccl
sudo apt install libtinfo5
sudo reboot
```
- [x] **Check your progress:** At this point you should be able to import Tensorflow in Python, make a simple operation, and exit without any error. Try the following in a python interactive session:
```python
import tensorflow as tf
tf.add(2,5)
exit()
```
You should see something like this: 
> <tf.Tensor: shape=(), dtype=int32, numpy=7>

Congrats, your machine is now ready to use tensorflow-rocm! You should still consider testing it with something more complex, like a benchmark.
## Test with a benchmark
```
sudo apt install git
git clone https://github.com/tensorflow/benchmarks
cd ./benchmarks/scripts/tf_cnn_benchmarks
python3 tf_cnn_benchmarks.py --num_gpus=1 --batch_size=32 --model=resnet50
```
Expect it to take some time (5-10 minutes), specially if it is the first time. You may think you got stuck in the warm up but be patient, it should not take that long next time.

If you see something like the output bellow then go and get yourself a taco, you did it!
> total images/sec: 87.92

***Have fun! :D***
