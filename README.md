# Windows Ubuntu subsystem installation guide

This guide contains similar steps as described [by NVIDIA](https://docs.nvidia.com/cuda/wsl-user-guide/index.html) but with more details. You can also follow [YouTube guide](https://www.youtube.com/watch?v=mWd9Ww9gpEM) made by Jeff Heaton.

Windows10 *prerelease* build 

## WSL2 Installation

Complete installation guide [by Microsoft](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

To install WSL2 you must be sure that your CPU supports *CPU Virtualization feature* and also this option must be enabled in UEFI.

If you want to install  WSL2 you must have at least ***version 1903 (build 18362)***.

If you want to also install CUDA in your WSL2 you must have at least ***version 2004 with kernel 4.19.121+ (build 20145, prerelease)***.

WSL2

As long as Windows prerelease builds are the part of Windows Insider Program you need to register in this program.
In Windows:
**Start** > **Settings** > **Privacy** > **Diagnostics & feedback** > :ballot_box_with_check: ✔**Optional**

[//]: # ":ballot_box_with_check:"

Enable Dev Channel to get latest update from Inside Program.
**Start** > **Settings** > **Update & security** > **Windows 10 Insider Preview** > ✔**Dev Channel**

If changing the parameter by standard methods is unattainable, it is recommended to use the Windows regedit:
In Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DataCollection create REG_DWORD type variable with name "AllowTelemetry" and change it to "1" value.

Microsoft profile registration required to participate in the Inside Program. For the most convenient centralized management using [Outlook](https://www.microsoft.com/en-us/microsoft-365/outlook/email-and-calendar-software-microsoft-outlook) is recommended.

Windows will switch build and download new updates. It requires multiple reboots.

Hyper-V in Windows Features must be disabled if you want to use WSL version 2.
```
dism.exe /Online /Disable-Feature:Microsoft-Hyper-V /All
```

Alternatively next command can be used.
```
bcdedit /set hypervisorlaunchtype off
```

Microsoft Windows Subsystem for Linux and Virtual Machine Platform must be enabled.
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
After that you will need to reboot your system.

You can check your Kernel with following command:
```
wsl cat /proc/version
```

Set your WSL to version 2.
```
wsl --set-default-version 2
```
If there are warnings related to virtualization, then you have not enabled CPU virtualization/disabled Hyper-V.

Next step is to install Ubuntu from Microsoft Store (20.04 is more preferable). Download and install it (choose user login/password).

You can then check subsystems.
```
wsl --list
```

## Setting up CUDA Toolkit

[Download](https://developer.nvidia.com/cuda/wsl/download) and install NVIDIA CUDA on WSL driver on Windows.

In Ubuntu:
```bash
sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo sh -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo apt-get update
```

Install cuda-toolkit last version.
```bash
sudo apt-get install -y cuda-toolkit-11-0
```

Maximum version of CUDA is in smi
```bash
nvidia-smi.exe
```

## Setting up cuDNN

Follow the [guide](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html).

Download [cuDNN](https://developer.nvidia.com/cudnn-download-survey) 8.0.3 for CUDA 11.0.

[Archive](https://developer.nvidia.com/rdp/cudnn-archive) is also available. You need to install 8.0.2+ for CUDA 11.0.

```bash
mv cudnn-11.0-linux-x64-v8.0.3.33.solitairetheme8 cudnn-11.0-linux-x64-v8.0.3.33.ga.tgz
tar -xzvf cudnn-11.0-linux-x64-v8.0.3.33.ga.tgz
sudo cp cuda/include/cudnn*.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```
Check if there is file *libcudnn.so.8.0.3* in /usr/local/cuda/lib64/.

Make CUDA example of Black-Scholes executing on GPU kernel 
```bash
cd /usr/local/cuda/samples/4_Finance/BlackScholes
sudo make -j12
sudo ./BlackScholes
```
Test must be passed if everything was installed properly.
If it was checked without cuDNN try to remake it.

## Setting up Nvidia drivers in classic Ubuntu 20.04

```bash
sudo apt-get remove --purge '^nvidia-.*'
sudo ubuntu-drivers devices
sudo ubuntu-drivers install
# sudo ubuntu-drivers autoinstall
```

## Setting up Miniconda3 on Ubuntu

Download and install Miniconda3.
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```
Agree with license terms and add it using conda init if you want.

In new tab create new environment and install numpy. It is required to build OpenCV properly.
```bash
conda create -n tfgpu python=3.8 # Y - agree to install
conda activate tfgpu
conda install numpy
```

## Setting up monitor forwarding

[Install VcXsrv](https://sourceforge.net/projects/vcxsrv/) on host machine (Windows).

Thereafter follow the [guide](https://dev.to/rescenic/comment/obea).

```bash
sudo apt update && sudo apt -y upgrade
sudo apt-get purge xrdp
sudo apt install xrdp
sudo apt install -y xfce4
sudo apt install -y xfce4-goodies

sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
sudo sed -i 's/3389/3390/g' /etc/xrdp/xrdp.ini
sudo sed -i 's/max_bpp=32/#max_bpp=32\nmax_bpp=128/g' /etc/xrdp/xrdp.ini
sudo sed -i 's/xserverbpp=24/#xserverbpp=24\nxserverbpp=128/g' /etc/xrdp/xrdp.ini
echo xfce4-session > ~/.xsession

sudo nano /etc/xrdp/startwm.sh
-comment these lines to:
#test -x /etc/X11/Xsession && exec /etc/X11/Xsession
#exec /bin/sh /etc/X11/Xsession
	
-add these lines:
# xfce
startxfce4

sudo /etc/init.d/xrdp start

export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0
```

## Setting up required language in WSL
```bash
locale -a

sudo locale-gen ru_RU
sudo locale-gen ru_RU.UTF-8

sudo update-locale 
```

In XLaunch session additional parameters VcXsrv:
```bash
-xkbmodel pc105 -xkblayout us,ru -xkboptions grp:alt_shift_toggle
```

## Adding new user
It is recommended to create user with /bin/bash shell. -G is for sudo.
```bash
sudo useradd -s /bin/bash -d /home/username -m -G username
sudo passwd username

sudo deluser --remove-home username
```

## Building OpenCV from sources

CUDA + cuDNN must be installed on your system before building up OpenCV.

[Example](https://medium.com/@sb.jaduniv/how-to-install-opencv-4-2-0-with-cuda-10-1-on-ubuntu-20-04-lts-focal-fossa-bdc034109df3) of installation guide.

```bash
mkdir opencv && cd opencv
git clone https://github.com/opencv/opencv
git clone https://github.com/opencv/opencv_contrib
```

Install all required packages and libraries.
```bash
sudo apt update
sudo apt upgrade

sudo apt install build-essential cmake pkg-config unzip yasm git checkinstall

sudo apt install libjpeg-dev libpng-dev libtiff-dev

sudo apt install libavcodec-dev libavformat-dev libswscale-dev libavresample-dev
sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt install libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev
sudo apt install libfaac-dev libmp3lame-dev libvorbis-dev

sudo apt install libopencore-amrnb-dev libopencore-amrwb-dev

# sudo apt-get install libdc1394-22 libdc1394-22-dev libxine2-dev libv4l-dev v4l-utils

sudo apt-get install libgtk-3-dev

sudo apt-get install python3-dev python3-pip
sudo -H pip3 install -U pip numpy
sudo apt install python3-testresources

sudo apt-get install libtbb-dev

sudo apt-get install libatlas-base-dev gfortran
```

```bash
# set up cmake flags with your own flags and paths
sudo sh ocvbuild.sh
```

Go to build and make OpenCV with your number of cores.
```bash
cd opencv/build
nproc
sudo make -j12
sudo make install 
```

Put libs in current enviroment.
```bash
sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
```

ImportError: /bin/../lib/libgio-2.0.so.0: undefined symbol: g_uri_join
Try to copy the same version of file from /usr/lib/x86_64-linux-gnu/ in conda lib.
```bash
cd ~/miniconda3/envs/tfgpu/lib
cp /usr/lib/x86_64-linux-gnu/libgio-2.0.so ./
```
Or "remove" libgio-2.0.so.0 file from conda lib.
```bash
mv libgio-2.0.so.0 _libgio-2.0.so.0
```

## Building dlib with GPU from sources

At first check [CUDA compatibility with gcc](https://stackoverflow.com/questions/6622454/cuda-incompatible-with-my-gcc-version).

Clone repository, build it using cmake and install in current Python environment.
```bash
git clone https://github.com/davisking/dlib.git
cd dlib
mkdir build; cd build; cmake ..; cmake --build .
cd ..
python3 setup.py install
```

Check in Python environment if dlib was installed using CUDA successfully.
```python
import dlib

# should be "True"
dlib.DLIB_USE_CUDA 

# should be "1"
print(dlib.cuda.get_num_devices()) 
```

If current version of CUDA is incompatible with gcc.
```bash
sudo apt -y install gcc-8 g++-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 8
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 8
```

Then reproduce previous steps (remake and build) again.

# SSH into WSL2
In WSL2 cmd:
```bash
sudo apt install openssh-server
sudo apt install net-tools
ifconfig
sudo ssh-keygen -A # if needed
sudpo nano /etc/ssh/sshd_config # change port to 2222
sudo service ssh --full-restart
```

In Powershell as Administrator:
```bash
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=172.23.129.80 connectport=2222
netsh advfirewall firewall add rule name=”Open Port 2222 for WSL2” dir=in action=allow protocol=TCP localport=2222
netsh interface portproxy show v4tov4 # list all rules
netsh int portproxy reset all # remove all rules
```

# TensorFlow installation guide

TensorFlow installation guide

## Beginning

At first decide on what device you will install TensorFlow.

The typical way is to install it with usage of the [cpu device].
The more modern way is to install it with usage of the [gpu device].

For gpu check if your version is [compatible with CUDA](https://developer.nvidia.com/cuda-gpus)

Also check [CUDA versions compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)

## Anaconda installation

In this guide Miniconda3 (basic version of Anaconda3 without packages) will be used.
It is better in case of easy installation and smaller size.

You can install Anaconda later using links on guides below in sections *`cpu`*, *`gpu`*.

Installation:
Delete all previous Miniconda3 (Anaconda3) versions

Install [Miniconda3](https://docs.conda.io/en/latest/miniconda.html)

Install for me

Do not add to path for [cpu, gpu w/ CUDA]

Add to path for [gpu w/out CUDA]

Set as default python enviroment [moreover if you don't use pip]

## [cpu]

You can follow [detailed instruction for tensorflow-cpu installation](https://github.com/jeffheaton/t81_558_deep_learning/blob/master/t81_558_class01_intro_python.ipynb)
by [Jeff Heaton](https://sites.wustl.edu/jeffheaton/) (Ph.D., Data scientist and adjunct instructor).
Also check included [videoinstruction](https://www.youtube.com/watch?v=59duINoc8GM)

Launch Anaconda promt (it will be installed in OS).

In (base) create new enviroment _tensorflowcpu_ with python3.6 version:
```bash
conda create --name tensorflowcpu python=3.6
```

Activate enviroment and check python version:
```bash
activate tensorflowcpu 
python --vsersion 
```

Install jupyter for better editing:
```bash
conda install jupyter
jupyter notebook
```

Install all necessary packages with specific versions:
```bash
conda install scipy
pip install --upgrade sklearn
pip install --upgrade pandas
pip install --upgrade pandas-datareader
pip install --upgrade matplotlib
pip install --upgrade pillow
pip install --upgrade requests
pip install --upgrade h5py
pip install --upgrade pyyaml
pip install --upgrade psutil
pip install --upgrade tensorflow==1.12.0
pip install --upgrade keras==2.2.4
```

Install VSC in Ubuntu:
```bash
wget https://code.visualstudio.com/docs/?dv=linux64_deb
sudo apt install ./code_1.55.2-1618307277_amd64.deb

wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg

sudo apt install apt-transport-https
sudo apt update
sudo apt install code

code
```

## [gpu without CUDA]

Compatibility with CUDA:
https://developer.nvidia.com/cuda-gpus
https://docs.nvidia.com/deploy/cuda-compatibility/index.html

Download [NVIDIA driver](https://www.nvidia.com/Download/index.aspx) (preferably Game Ready Driver).

Then you can follow [full guide of how to install TF without CUDA on Win10](https://www.pugetsystems.com/labs/hpc/How-to-Install-TensorFlow-with-GPU-Support-on-Windows-10-Without-Installing-CUDA-UPDATED-1419/) written by [Dr Donald Kinghorn](https://www.pugetsystems.com/bios.php?name=donkinghorn).

Short version of how to install:

At first install Anaconda (if you still didn't complete it) that was described in section ## Anaconda installation.

Then in Anaconda promt:

```bash
conda create --name tf-gpu
conda activate tf-gpu
conda install tensorflow-gpu
```

Install jupyter if you didn't and ipykernel (including with tf-gpu-version).
```bash
conda install ipykernel jupyter
python -m ipykernel install --user --name tf-gpu --display-name "TensorFlow-GPU-1.13"
jupyter notebook
```

Try to check the [gpu devices](https://stackoverflow.com/questions/38559755/how-to-get-current-available-gpus-in-tensorflow) using:

```python
from tensorflow.python.client import device_lib
local_device_protos = device_lib.list_local_devices()
print([x.name for x in local_device_protos if x.device_type == 'GPU'])
```

You must see something like:
> name: Gece GTX XXX major: X minor: X memoryClockRate(GHz): ...
>
> ['/device:GPU:0']

## [gpu with CUDA]

https://www.youtube.com/watch?v=KZFn0dvPZUQ

# Caffe installation guide

Caffe installation guide

## Beginning

For gpu check if your version is [compatible with CUDA](https://developer.nvidia.com/cuda-gpus).

Install last version of [gpu driver](https://www.nvidia.com/Download/Find.aspx).

Caffe requires CUDA packages before main installation. So it is better to [download CUDA](https://developer.nvidia.com/cuda-downloads)
and [cuDNN](https://developer.nvidia.com/cudnn). Also check the compatibility of:
1. NVIDIA GPU drivers
1. CUDA
1. cuDNN

Unpack cuDNN and copy files:
/cuda/
1. bin/
1. include/
1. lib/
1. txt file (not necessary)
in the directory of the CUDA (e.g C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v10.1/

## Installation in conda

Check open [issue on github](https://github.com/BVLC/caffe/issues/6569).

Be sure that CUDA (and its directories) are visible for your system (e.g check PATH variable on Win10).

In conda promt:
```bash
conda create --name caffe-gpu
conda activate caffe-gpu
conda config --add channels anaconda
conda install caffe -c willyd
```

In Python console try:
```bash
import caffe
```

## By hands

Follow [guide](https://www.youtube.com/watch?v=SeCE757egcE)
