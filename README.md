# Simple install guide for ROCm and Tensorflow, designed for Ubuntu 20.4.1 LTS and Radeon RX580 
## 

This guide will show you how to set up your clean **Ubuntu 20.4.1 LTS** system ready to run **Tensorflow** projects while using **ROCm** to take advantage of the power of your **RX580 graphics card (gfx803)** in an easy and fast way. (It should work on other supported Ubuntu versions or other graphic cards too, with only slight changes).
It is basically a resume of the official https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html by AMD, and the unofficial https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html by Mathieu Poliquin, with a few considerations and fixes.

Lets get started!

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
```
