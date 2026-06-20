# Install cuda 12.4

The version is depends on which version the gpu driver is supporting, in this case I will use CUDA 12.4

To check nvidia gpu driver version run

| nvidia-smi |
| :---- |

1. To get the binary file, Run in terminal

| wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local\_installers/cuda\_12.4.0\_550.54.14\_linux.run |
| :---- |

2. Run this to install the toolkit without installing the new driver (we used the existing tested driver from ubuntu team)

| sudo sh cuda\_12.4.0\_550.54.14\_linux.run \-toolkit \-silent \-override |
| :---- |

3. Set up the environment by copying the following line into the bashrc file  
   

| export PATH=/usr/local/cuda-12.4/bin${PATH:+:${PATH}}export LD\_LIBRARY\_PATH=/usr/local/cuda-12.4/lib64\\                         ${LD\_LIBRARY\_PATH:+:${LD\_LIBRARY\_PATH}} |
| :---- |

4. Close and open new terminal then run the command below to check whether the toolkit is installed successfully

| nvcc \--version |
| :---- |

## Test CUDA 12.4

Download files from  [https://github.com/nvidia/cuda-samples](https://github.com/nvidia/cuda-samples), download zip files with the same tags as the CUDA version

1. Go to appropriate workdir and compile the deviceQuery file by running this command in the terminal

| nvcc cuda-samples-12.4/Samples/1\_Utilities/deviceQuery/deviceQuery.cpp \-I ./cuda-samples-12.4/Common/ |
| :---- |

2. Run the deviceQuery

| ./deviceQuery |
| :---- |

3. If successful, you will see status PASS on the printed output.