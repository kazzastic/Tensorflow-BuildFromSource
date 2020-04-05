# Tensorflow-BuildFromSource
How to built tensorflow from source, on Xeon processor, ubuntu 19.x, tf 2.x with GPU support.

Even though all the required information that one might need when building tensorflow from source can  be obtained from
https://www.tensorflow.org/install/source but I just want to highlight the current situation of tensorflow on ubuntu and xeon processor. 

## Configuration 
These are the configuration which will be able to re produce results most accurately 
- Ubuntu 19.10
- CPU: Intel Xeon E5620 (8) @ 2.395GHz 
- GPU: NVIDIA GeForce GTX 1050 Ti
- NVIDIA-SMI 440.64.00
- CUDA 10.1
- nvcc cuDNN 7.6

## GPU softwares

### CUDA Toolkit 10.1 Download
[Here](https://developer.nvidia.com/cuda-10.1-download-archive-update2?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal) you can see how to install the CUDA 10.1, even though this is outdated but you would want to install this once since TF 2.X supports this version.

```
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin

$ sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600

$ wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda-repo-ubuntu1804-10-1-local-10.1.243-418.87.00_1.0-1_amd64.deb

$ sudo dpkg -i cuda-repo-ubuntu1804-10-1-local-10.1.243-418.87.00_1.0-1_amd64.deb

$ sudo apt-key add /var/cuda-repo-10-1-local-10.1.243-418.87.00/7fa2af80.pub

$ sudo apt-get update

$ sudo apt-get -y install cuda
```
now, if you had already tried installing an nvidia toolkit before then you might run into this minor problem which might give you following error.

```
$ sudo apt-get install cuda
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies.
 cuda : Depends: cuda-10-0 (>= 10.0.130) but it is not going to be installed
E: Unable to correct problems, you have held broken packages
```
this is normal and can be solved by the following commands, 

```
$ sudo add-apt-repository -r ppa:graphics-drivers/ppa

$ sudo apt remove nvidia-*

$ sudo apt autoremove
```
this should solved the problem by removing previouly installed toolkits of any version, and now proceed where you left from meaning, 

```
$ sudo apt install cuda
```
once done installing, modify the `~/.profile` file and add the following lines, 
first gedit, nano or vim open the file using following command, 
```
$ nano ~/.profile
```
then add these lines, 
```
# set PATH for cuda 10.1 installation
if [ -d "/usr/local/cuda-10.1/bin/" ]; then
    export PATH=/usr/local/cuda-10.1/bin${PATH:+:${PATH}}
    export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
fi
```
then compile this once, just to be sure!
```
$ source ~/.profile
```
now restart the OS once or use this,
```
$ sudo reboot now
```
Now check the installation using these commmands, 
```
$ nvidia-smi
Sun Apr  5 18:34:20 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.64.00    Driver Version: 440.64.00    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 105...  On   | 00000000:0F:00.0  On |                  N/A |
|  0%   46C    P0    N/A /  72W |    341MiB /  4038MiB |     15%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1113      G   /usr/lib/xorg/Xorg                            14MiB |
|    0      1705      G   /usr/lib/xorg/Xorg                            85MiB |
|    0      1912      G   /usr/bin/gnome-shell                         101MiB |
|    0      7262      G   ...AAAAAAAAAAAAAAgAAAAAAAAA --shared-files    88MiB |
+-----------------------------------------------------------------------------+

$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Sun_Jul_28_19:07:16_PDT_2019
Cuda compilation tools, release 10.1, V10.1.243
```
if you get any error here reach out to github/stackoverflow community. 

If you run into any gcc or gnu version error till now, or in any further installation then install these versions of gcc and gnu, 
```
$ sudo apt-get install -y software-properties-common

$ sudo add-apt-repository ppa:ubuntu-toolchain-r/test

$ sudo apt update

$ sudo apt install g++-7 -y
```
Set it up so the symbolic links gcc, g++ point to the newer version:
```
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 60 \
                         --slave /usr/bin/g++ g++ /usr/bin/g++-7 

$ sudo update-alternatives --config gcc

$ gcc --version

$ g++ --version
```

### cuDNN v7.6.5 for CUDA 10.1

You can find the file [here](https://developer.nvidia.com/rdp/cudnn-download), you might have to sign up to download it. 
Download the given packages 
- cuDNN Runtime Library for Ubuntu18.04 (Deb)
- cuDNN Developer Library for Ubuntu18.04 (Deb)
- cuDNN Code Samples and User Guide for Ubuntu18.04 (Deb)
These are for Ubuntu 18.04 but they work like a charm even on Ubuntu 19.10

```
# see files we downloaded
$ ll | grep cudnn

$ sudo dpkg -i libcudnn7-doc_7.6.5.32-1+cuda10.1_amd64.deb
 
$ sudo dpkg -i libcudnn7-dev_7.6.5.32-1+cuda10.1_amd64.deb
 
$ sudo dpkg -i libcudnn7_7.6.5.32-1+cuda10.1_amd64.deb

$ apt search libcudnn7
```
navigate the packages and install each as above.
We then test the CUDNN installation with:
```
$ cp -r /usr/src/cudnn_samples_v7/ ~/test/
$ cd ~/test/mnistCUDNN/
$ make clean && make
$ $ ./mnistCUDNN
# ...
# Test passed!
```
We need to know where CUDDN7 is installed for tensorflow build configuration
```
$ whereis cudnn

cudnn: /usr/include/cudnn.h

$ ll /usr/include/ | grep cudnn

lrwxrwxrwx  1 root root     26 Sep 29 22:06 cudnn.h -> /etc/alternatives/libcudnn

$ ll /etc/alternatives/libcudnn

libcudnn        libcudnn_so     libcudnn_stlib
  
$ ll /etc/alternatives/ | grep cudnn
lrwxrwxrwx   1 root root    40 Sep 29 22:06 libcudnn -> /usr/include/x86_64-linux-gnu/cudnn_v7.h
lrwxrwxrwx   1 root root    39 Sep 29 22:06 libcudnn_so -> /usr/lib/x86_64-linux-gnu/libcudnn.so.7
lrwxrwxrwx   1 root root    46 Sep 29 22:06 libcudnn_stlib -> /usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a

$ dpkg-query -L libcudnn7
/.
/usr
/usr/lib
/usr/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu/libcudnn.so.7.3.1
/usr/share
/usr/share/doc
/usr/share/doc/libcudnn7
/usr/share/doc/libcudnn7/changelog.Debian.gz
/usr/share/doc/libcudnn7/copyright
/usr/share/lintian
/usr/share/lintian/overrides
/usr/share/lintian/overrides/libcudnn7
/usr/lib/x86_64-linux-gnu/libcudnn.so.7

$ dpkg-query -L libcudnn7-dev 
/.
/usr
/usr/include
/usr/include/x86_64-linux-gnu
/usr/include/x86_64-linux-gnu/cudnn_v7.h
/usr/lib
/usr/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a
/usr/share
/usr/share/doc
/usr/share/doc/libcudnn7-dev
/usr/share/doc/libcudnn7-dev/changelog.Debian.gz
/usr/share/doc/libcudnn7-dev/copyright
/usr/share/lintian
/usr/share/lintian/overrides
/usr/share/lintian/overrides/libcudnn7-dev
```
We decide to use `/etc/alternatives/` for the tensorflow configuration.

## Install Bazel 

### Release Version of Bazel 
The most suitable for our build can be checked [here](https://www.tensorflow.org/install/source#tested_build_configurations) further more navigate to correct release version of bazel from [here](https://github.com/bazelbuild/bazel/releases) I used bazel version 2.0.0 
download the `bazel-2.0.0-installer-linux-x86_64.sh` 

```
$ cd where-bazel-is-downloaded

$ chmod +x bazel-2.0.0-installer-linux-x86_64.sh

$ ./bazel-2.0.0-installer-linux-x86_64.sh --user
```
this would install the bazel. Now setup the enviroment, 
```
export PATH="$PATH:$HOME/bin"
```
add these lines to your ./bashrc file, you can open this file with these commands, 
```
$ nano ~/.bashrc
```

### NVIDIA Collective Communications Library (NCCL)
From [here](https://developer.nvidia.com/nccl/nccl-download) download `Download NCCL v2.6.4, for CUDA 10.1, March 26,2020`. When enabling CUDA tensorflow asks us to specify the nccl library location. We go at this nvidia webpage to install it. We download the network installer, so that it is more straightforward to get updates.

```
$ sudo dpkg-i nccl-repo-ubuntu1804-2.6.4-ga-cuda10.2_1-1_amd64.deb 

$ sudo apt update

$ apt search libnccl
Full Text Search... Done
libnccl-dev/unknown 2.3.5-2+cuda10.0 amd64
  NVIDIA Collectives Communication Library (NCCL) Development Files

libnccl2/unknown 2.3.5-2+cuda10.0 amd64
  NVIDIA Collectives Communication Library (NCCL) Runtime

$ sudo apt install libnccl2 libnccl-dev

# See where the libraries got installed
$ dpkg-query -L libnccl-dev libnccl2
/.
/usr
/usr/include
/usr/include/nccl.h
/usr/lib
/usr/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu/libnccl_static.a
/usr/share
/usr/share/doc
/usr/share/doc/libnccl-dev
/usr/share/doc/libnccl-dev/changelog.Debian.gz
/usr/share/doc/libnccl-dev/copyright
/usr/lib/x86_64-linux-gnu/libnccl.so

/.
/usr
/usr/lib
/usr/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu/libnccl.so.2.3.5
/usr/share
/usr/share/doc
/usr/share/doc/libnccl2
/usr/share/doc/libnccl2/changelog.Debian.gz
/usr/share/doc/libnccl2/copyright
/usr/lib/x86_64-linux-gnu/libnccl.so.2
```
We decide to create symbolic links from within the cuda installation to where they are:
```
$ cd /usr/local/cuda/

$ sudo mkdir lib

$ cd lib

$ sudo ln -s /usr/lib/x86_64-linux-gnu/libnccl.so.2 libnccl.so.2

$ cd ../include

$ sudo ln -s /usr/include/nccl.h nccl.h
```
## Building Tensorflow




