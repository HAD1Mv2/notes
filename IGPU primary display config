- [Configure IGPU as primary display driver](#configure-igpu-as-primary-display-driver)
  - [Setting steps](#setting-steps)
    - [1. Hardware \& BIOS Configuration](#1-hardware--bios-configuration)
    - [2. Driver \& System Setup](#2-driver--system-setup)
      - [Installing Drivers](#installing-drivers)
      - [Verification](#verification)
    - [3. Configuring Neural Network Frameworks](#3-configuring-neural-network-frameworks)
      - [PyTorch](#pytorch)
      - [TensorFlow](#tensorflow)
    - [4. (Optional) Explicit Environment Variables](#4-optional-explicit-environment-variables)
    - [Quick Summary Checklist](#quick-summary-checklist)
  - [Checking which GPU used as primary display](#checking-which-gpu-used-as-primary-display)
    - [1. Check Which GPU is Rendering Your Display (Monitors)](#1-check-which-gpu-is-rendering-your-display-monitors)
    - [2. Check Hardware \& Driver Status for All GPUs](#2-check-hardware--driver-status-for-all-gpus)
    - [3. Monitor Real-Time GPU Usage](#3-monitor-real-time-gpu-usage)
      - [Option A: Vendor-Specific Tools](#option-a-vendor-specific-tools)
      - [Option B: Unified All-in-One Monitor (`nvtop`)](#option-b-unified-all-in-one-monitor-nvtop)

# Configure IGPU as primary display driver

## Setting steps
Setting up your Linux machine to use the integrated GPU (iGPU) for your display monitors and keeping the discrete GPU (dGPU) dedicated exclusively to neural network training (CUDA / ROCm) is a great approach. It frees up **100% of your dGPU's VRAM** for machine learning tasks and prevents desktop lag while training heavy models.

Here is the step-by-step guide to set this up:

---

### 1. Hardware & BIOS Configuration

1. **Connect Display Cables:** Plug all display cables (HDMI / DisplayPort) directly into the **motherboard ports** (iGPU), not the dedicated graphics card.
2. **BIOS / UEFI Settings:**
* Reboot into your BIOS/UEFI (usually by pressing `F2`, `Del`, or `F12` during boot).
* Search for graphics settings (often under *Advanced*, *System Agent*, or *PCI Configuration*).
* Set **Primary Display / Initial Display Output** to **IGFX / iGPU / Processor Graphics**.
* Enable options like **iGPU Multi-Monitor** or **Enable iGPU** if available.
* Save and reboot into Linux.



---

### 2. Driver & System Setup

#### Installing Drivers

* **NVIDIA dGPU:** Install the proprietary NVIDIA driver package via your distribution's package manager (e.g., `sudo apt install nvidia-driver-550` on Ubuntu or via `additional-drivers`).
* **AMD dGPU:** Install `amdgpu` drivers along with the [ROCm stack](https://rocm.docs.amd.com/).

#### Verification

Open a terminal and verify that both GPUs are recognized by the system:

```bash
# Verify both GPUs are visible to the OS
lspci -nn | grep -Ei 'VGA|3D|DISPLAY'

```

Check that your dGPU compute stack is active:

* **NVIDIA:** Run `nvidia-smi`. You should see your card listed with `0 MiB / XX MiB` VRAM in use (since no display desktop processes are running on it).
* **AMD:** Run `rocm-smi` or `clinfo`.

---

### 3. Configuring Neural Network Frameworks

Modern Linux display managers (Wayland or X11) automatically default to rendering the desktop on whatever GPU connected to your monitors (iGPU). Machine learning frameworks detect available compute devices via PCIe regardless of display connections.

#### PyTorch

By default, PyTorch assigns `cuda:0` to your primary CUDA device.

```python
import torch

# Check if CUDA is available
print("CUDA Available:", torch.cuda.is_available())
print("Device Name:", torch.cuda.get_device_name(0))

# Assign tensor/model to dGPU
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
model.to(device)

```

#### TensorFlow

TensorFlow will automatically allocate compute tasks to the available dGPU.

```python
import tensorflow as tf

# List available GPUs
gpus = tf.config.list_physical_devices('GPU')
print("GPUs Available:", gpus)

```

---

### 4. (Optional) Explicit Environment Variables

If you have multiple GPUs or want to explicitly enforce which GPU your Python script uses, set the `CUDA_VISIBLE_DEVICES` or `HIP_VISIBLE_DEVICES` environment variable:

* **In Terminal:**
```bash
CUDA_VISIBLE_DEVICES=0 python train.py

```


* **In Python Code:**
```python
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "0"  # Index of your dGPU

```



---

### Quick Summary Checklist

* [x] Display cable plugged into motherboard (iGPU).
* [x] Primary display set to iGPU in BIOS.
* [x] Proprietary dGPU drivers / CUDA / ROCm installed.
* [x] Verified `nvidia-smi` shows `0 MiB` display memory usage at idle.

## Checking which GPU used as primary display

To check which GPU is handling display tasks versus which GPU is being utilized for background compute (like machine learning), you can use a few quick terminal commands in Linux:

---

### 1. Check Which GPU is Rendering Your Display (Monitors)

To confirm that your iGPU is actively rendering your graphical desktop interface (X11 or Wayland), check the **OpenGL renderer**:

```bash
glxinfo | grep -E "OpenGL vendor|OpenGL renderer"

```

*(If `glxinfo` is missing, install it via `sudo apt install mesa-utils` on Debian/Ubuntu).*

* **Expected Output:** You should see **Intel** or **AMD** listed as the renderer string (your iGPU), confirming your desktop GUI is not running on the dGPU.

---

### 2. Check Hardware & Driver Status for All GPUs

To list all detected GPUs and see which kernel drivers are actively bound to them:

```bash
lspci -k | grep -A 3 -i 'vga\|3d\|display'

```

* Look for the `Kernel driver in use:` line under each card.
* Your iGPU should be using `i915` or `xe` (Intel) or `amdgpu` (AMD).
* Your dGPU should show `nvidia` (or `amdgpu` for AMD dGPUs).

---

### 3. Monitor Real-Time GPU Usage

#### Option A: Vendor-Specific Tools

* **NVIDIA dGPU:** Run `nvidia-smi` to see active processes and VRAM usage.
```bash
nvidia-smi

```


If configured correctly, display processes (like `gnome-shell`, `Xorg`, or `kwin`) will **not** appear here, and VRAM usage will stay near `0 MiB` until your Python ML training script starts.
* **AMD GPUs:** Use `rocm-smi` or `radeontop`.
```bash
rocm-smi

```



---

#### Option B: Unified All-in-One Monitor (`nvtop`)

The easiest way to view both iGPU and dGPU usage side-by-side in real time is using `nvtop`:

```bash
# Install nvtop
sudo apt install nvtop     # Debian/Ubuntu
sudo dnf install nvtop     # Fedora/RHEL
sudo pacman -S nvtop       # Arch Linux

# Run monitor
nvtop

```

`nvtop` opens an `htop`-like interactive graph showing load %, VRAM usage, and running process IDs for **all** detected GPUs simultaneously.