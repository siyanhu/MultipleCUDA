# How to apply multiple CUDA versions in Ubuntu system?

## Download CUDA files
 
 To pick the CUDA files you need to select the options based on specific machine and system. For me it is 
 Linux | x86_64 | Ubuntu 22.04 | runfile(local)
 and run the following commands

**11.8**

        wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
        sudo sh cuda_11.8.0_520.61.05_linux.run --silent --toolkit --toolkitpath=/usr/local/cuda-11.8

**12.1**
    
        wget https://developer.download.nvidia.com/compute/cuda/12.1.0/local_installers/cuda_12.1.0_530.30.02_linux.run
        sudo sh cuda_12.1.0_530.30.02_linux.run --silent --toolkit --toolkitpath=/usr/local/cuda-12.1
    
**12.5**

        wget https://developer.download.nvidia.com/compute/cuda/12.5.1/local_installers/cuda_12.5.1_555.42.06_linux.run
        sudo sh cuda_12.5.1_555.42.06_linux.run --silent --toolkit --toolkitpath=/usr/local/cuda-12.5


## Update env variables
Run the following commands:

    cd /usr/local
    sudo remove -rf cuda
    ln -s /usr/local/cuda-[target_version] cuda

and you will see a symbolink named "cuda" in /usr/local folder which directs to your target version of cudatoolkit. To verify you may type nvcc --version in your terminal. If it shows correct information, proceed to next step.

go to /home and find .bashrc.
Add the following scripts at the bottom of the file:

    export CUDA_HOME="/usr/local/cuda"
    export LD_LIBRARY_PATH=${CUDA_HOME}/lib64:$LD_LIBRARY_PATH
    export PATH="$PATH:$CUDA_HOME/bin"

and run 

    source .bashrc 
    nvcc --version

If it still doesn't show correct version, try to specify the version in .bashrc:
    
    export PATH=/usr/local/cuda-11.8/bin:$PATH
    export CUDA_HOME=/usr/local/cuda-11.8
    export LD_LIBRARY_PATH=/usr/local/cuda-11.8/

To verify cudatoolkit, try to run pytorch by downloading specific torch from [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/). For example CUDA 11.8 for conda env:

    conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

And in your conda activated terminal, type python and try the following script:

    import torch
    torch.cuda.is_available()

It should show "True".

## Update g++ and gcc to match CUDA rasterisation
Sometime system default g++/gcc version cannot match with CUDA. Here we should update by following instructions in this link:
    
    https://docs.nvidia.com/cuda/archive/[target_version]/cuda-installation-guide-linux/index.html

e.g.

**11.8** https://docs.nvidia.com/cuda/archive/11.8.0/cuda-installation-guide-linux/index.html

**12.1** https://docs.nvidia.com/cuda/archive/12.1.0/cuda-installation-guide-linux/index.html

**12.5** https://docs.nvidia.com/cuda/archive/12.5.0/cuda-installation-guide-linux/index.html

Taking 12.1 as an example, in [installation guides](https://docs.nvidia.com/cuda/archive/12.1.0/cuda-installation-guide-linux/index.html) we know that we need gcc/g++ 12.2 + for CUDA toolkit. So we check what we have on our system first:

    sudo dpkg -l | grep gcc
    sudo dpkg -l | grep g++

for my computer I see version 10, 11 and 13 on it. To be safe, we install gcc/g++ version 12 right now.
    
    sudo apt-get install gcc-12
    sudo apt-get install g++-12

and we'll see g++-12 is already the newest version (12.3.0-17ubuntu1). Same for gcc. And then we update alternatives:

    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-12 12

If we want to arrange other versions, just add the following code if 11 and 13 as example also on the disk:

    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 11
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 13
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-13 13

Then we check c/c++ compiler on system env and select target version:
    
    sudo update-alternatives --config gcc
    sudo update-alternatives --config g++

to verify the above steps, type:

    gcc --version
    g++ --version

