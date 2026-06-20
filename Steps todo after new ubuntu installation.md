- [Install build essential](#install-build-essential)
- [Install curl](#install-curl)
- [Install git](#install-git)
- [Install CUDA in Ubuntu 26.04](#install-cuda-in-ubuntu-2604)
  - [Test CUDA](#test-cuda)
- [Install miniconda](#install-miniconda)
  - [Managing conda](#managing-conda)
    - [Create new venv](#create-new-venv)
    - [Activate and deactivate new venv](#activate-and-deactivate-new-venv)
    - [View list of active venvs](#view-list-of-active-venvs)
- [Install pytorch](#install-pytorch)
- [Install transformers](#install-transformers)
  - [Optional: Install huggingface\_hub](#optional-install-huggingface_hub)
- [Install Jupyter lab](#install-jupyter-lab)
  - [Connect jupyter lab with specific venv](#connect-jupyter-lab-with-specific-venv)
- [Install Docker](#install-docker)
- [Optionals](#optionals)


# Install build essential

1. Run in terminal 

``` bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential 
``` 


# Install curl

1. Run in terminal

``` bash
sudo apt install curl
```

# Install git

1. Run in terminal

``` bash
sudo apt install git 
```

# Install CUDA in Ubuntu 26.04

This Ubuntu version supports installing cuda directly from the repository. It is easier, no hassle like the previous installing process. Just run codes below. The CUDA version is 13.1.

1. 
```nbash
Sudo apt install cuda-toolkit -y 
```

2. Set up the environment by copying the following lines into the bashrc file  
   
``` bash
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}} 
```

## Test CUDA

Download files from  [https://github.com/nvidia/cuda-samples](https://github.com/nvidia/cuda-samples), download zip files with the same tags as the CUDA version

1. Go to appropriate workdir and compile the deviceQuery file by running this command in the terminal

``` bash
nvcc cuda-samples-13.1/Samples/1_Utilities/deviceQuery/deviceQuery.cpp -I ./cuda-samples-13.1/Common/
```

   

2. There will be file a.out in the same folder, Then you need to run the file

```
./a.out 
```

   

3. If successful, you will see status `PASS` on the printed output.

# Install miniconda

1. Run sequentially in the terminal the following command

``` bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86\_64.sh
bash ~/Miniconda3-latest-Linux-x86_64.sh 
```

2. Type `yes` for agreeing with **TOS**  
3. Choose `yes` for initialization option  
4. Close and reopen terminal to be able to use `conda` command

## Managing conda

### Create new venv

When working with python or any project, it is better to work in virtual environment, to keep the main env from unintended change and keep the overall system safe.

1. To create an environment (named myenv) with a specific version of Python, run 

```
conda create -n myenv python=3.11
```

### Activate and deactivate new venv

1. Run this to activate
```
conda activate myenv
```

2. Run this to deactivate

```
conda deactivate 
```

3. To remove env

```
conda env remove --name myenv
```

### View list of active venvs

```
conda info --envs
```



# Install pytorch

The last pytorch supported compute capability 6.1 (included gtx 1060 3g) is version 2.9.0+cu126, to install run in terminal

Pip

``` bash
pip install torch==2.9.0 torchvision==0.24.0 torchaudio==2.9.0 --index-url https://download.pytorch.org/whl/cu130
```

To check whether pytorch using cuda or not, run in python

``` python
import torch
torch.cuda.is_available()
```

# Install transformers

Pip

```
pip install transformers
```


Conda
```
conda install conda-forge::transformers
```

## Optional: Install huggingface\_hub

Pip

```
pip install huggingface_hub
```

Conda

```
conda install -c conda-forge huggingface_hub 
```

To be able to use ipywidget, install ipywidgets,  run in the virtual env

Conda

```
conda install -c conda-forge ipywidgets
```

# Install Jupyter lab

Note: install jupyter lab in base env

1. Using pip

```
pip install jupyterlab
```

2. Or if using conda

```
conda install conda-forge::jupyterlab
```

3. Then to launch jupyter lab, run command below in the terminal

```
jupyter lab
```

## Connect jupyter lab with specific venv

In terminal, go to virtual env  
Run these

```
conda install ipykernel
python -m ipykernel install -user -name=myenv
```

Change myenv with the venv name

# Install Docker

Goto [https://docs.docker.com/engine/install/ubuntu/\#install-using-the-repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)  
Follow the steps there

To run docker without sudo

Run

```
sudo groupadd docker sudo usermod -aG docker $USER
```

Logout then re-login

Try

```
docker run hello-world
```

# Optionals

Install node js and npm

1. Run in terminal

```
sudo apt update
sudo apt install nodejs
node -v
sudo apt install npm
```
