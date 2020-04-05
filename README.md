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

