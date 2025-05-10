# gsplat 3DGUT Tutorial

## About this project fork - READ FIRST

I created this fork from the oiginal gsplat project to guide users through installing gsplat on Windows PCs. Please refer to the original project for the latest developments as this fork does not contain future developments and may not be maintaine in perpetuity.

### Reference to the official gsplat project
[http://www.gsplat.studio/](http://www.gsplat.studio/)

### About gsplat
gsplat is an open-source library for CUDA accelerated rasterization of gaussians with python bindings. It is inspired by the SIGGRAPH paper [3D Gaussian Splatting for Real-Time Rendering of Radiance Fields](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/), but we’ve made gsplat even faster, more memory efficient, and with a growing list of new features! 

<div align="center">
  <video src="https://github.com/nerfstudio-project/gsplat/assets/10151885/64c2e9ca-a9a6-4c7e-8d6f-47eeacd15159" width="100%" />
</div>

## News

[April 2025] [NVIDIA 3DGUT](https://research.nvidia.com/labs/toronto-ai/3DGUT/) is now integrated in gsplat! Checkout [here](docs/3dgut.md) for more details.

## Installation

### Dependencies:

- Install [Git](https://git-scm.com/downloads).
- Install Anaconda or Miniconda - I suggest [Miniconda](https://www.anaconda.com/download?utm_source=anacondadocs&utm_medium=documentation&utm_campaign=download&utm_content=installwindows)
- Install Visual Studio 2022. This must be done before installing CUDA Toolkit. The necessary components are included in the `Desktop Development with C++` workflow (also called `C++ Build Tools` in the BuildTools edition). Also, see below for setup.
- Cuda Toolkit (I tested with 12.6 and 11.8) Note that if you use 11.8, you will need Visual Studio 2019. You can download VS2019 Professional edition for free. You do not need to activate a license for building gsplat.
- Pytorch (see below in install notes)
  v

<br><br>

### Visual studio setup

Setting up and activating Visual Studio can be done through these steps:

1. Install Visual Studio Build Tools. If MSVC 143 does not work, you may also need to install MSVC 142 for Visual Studio 2019. And your CUDA environment should be set up properly.


2. Activate your conda environment:
    ```bash
    conda activate <your_conda_environment>
    ```
    Replace `<your_conda_environment>` with the name of your conda environment. For example:
    ```bash
    conda activate gsplat
    ```

3. Activate your Visual C++ environment:
    Navigate to the directory where `vcvars64.bat` is located. This path might vary depending on your installation. A common path is:
    ```
    C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build
    ```

4. Run the following command:
    ```bash
    ./vcvars64.bat
    ```

    If the above command does not work, try activating an older version of VC:
    ```bash
    ./vcvarsall.bat x64 -vcvars_ver=<your_VC++_compiler_toolset_version>
    ```
    Replace `<your_VC++_compiler_toolset_version>` with the version of your VC++ compiler toolset. The version number should appear in the same folder.
    
    For example:
    ```bash
    ./vcvarsall.bat x64 -vcvars_ver=14.29
    ```
<br><br>


### Install CUDA Toolkit
Find your prefered distrbution here: [https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive)
I suggest 12.6, but 11.8 is also compatible with many other 3DGS projects.
<br><br>

### Create a Conda Environment and Install Pytorch

For CUDA Toolkit 12.6 
```bash
conda create --name gsplat -y python=3.10
conda activate gsplat
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
```

For CUDA Toolkit 11.8
```bash
conda create --name gsplat -y python=3.10
conda activate gsplat
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

**Note:** Refer to [Pytorch.org](https://pytorch.org/get-started/locally/) for additional CUDA Toolkit distributions

<br><br>

### Clone the `gsplat` repository:
    ```bash
    git clone --recursive https://github.com/nerfstudio-project/gsplat.git
    cd gsplat
    ```

### Install `gsplat` using pip:
    ```bash
    pip install -e .
    ```

To build gsplat from source on Windows, please check [this instruction](docs/INSTALL_WIN.md).

## Evaluation

This repo comes with a standalone script that reproduces the official Gaussian Splatting with exactly the same performance on PSNR, SSIM, LPIPS, and converged number of Gaussians. Powered by gsplat’s efficient CUDA implementation, the training takes up to **4x less GPU memory** with up to **15% less time** to finish than the official implementation. Full report can be found [here](https://docs.gsplat.studio/main/tests/eval.html).

```bash
cd examples
pip install -r requirements.txt
# download mipnerf_360 benchmark data
python datasets/download_dataset.py
# run batch evaluation
bash benchmarks/basic.sh
```

## Examples

We provide a set of examples to get you started! Below you can find the details about
the examples (requires to install some exta dependencies via `pip install -r examples/requirements.txt`)

- [Train a 3D Gaussian splatting model on a COLMAP capture.](https://docs.gsplat.studio/main/examples/colmap.html)
- [Fit a 2D image with 3D Gaussians.](https://docs.gsplat.studio/main/examples/image.html)
- [Render a large scene in real-time.](https://docs.gsplat.studio/main/examples/large_scale.html)


## Development and Contribution

This repository was born from the curiosity of people on the Nerfstudio team trying to understand a new rendering technique. We welcome contributions of any kind and are open to feedback, bug-reports, and improvements to help expand the capabilities of this software.

This project is developed by the following wonderful contributors (unordered):

- [Angjoo Kanazawa](https://people.eecs.berkeley.edu/~kanazawa/) (UC Berkeley): Mentor of the project.
- [Matthew Tancik](https://www.matthewtancik.com/about-me) (Luma AI): Mentor of the project.
- [Vickie Ye](https://people.eecs.berkeley.edu/~vye/) (UC Berkeley): Project lead. v0.1 lead.
- [Matias Turkulainen](https://maturk.github.io/) (Aalto University): Core developer.
- [Ruilong Li](https://www.liruilong.cn/) (UC Berkeley): Core developer. v1.0 lead.
- [Justin Kerr](https://kerrj.github.io/) (UC Berkeley): Core developer.
- [Brent Yi](https://github.com/brentyi) (UC Berkeley): Core developer.
- [Zhuoyang Pan](https://panzhy.com/) (ShanghaiTech University): Core developer.
- [Jianbo Ye](http://www.jianboye.org/) (Amazon): Core developer.

We also have a white paper with about the project with benchmarking and mathematical supplement with conventions and derivations, available [here](https://arxiv.org/abs/2409.06765). If you find this library useful in your projects or papers, please consider citing:

```
@article{ye2025gsplat,
  title={gsplat: An open-source library for Gaussian splatting},
  author={Ye, Vickie and Li, Ruilong and Kerr, Justin and Turkulainen, Matias and Yi, Brent and Pan, Zhuoyang and Seiskari, Otto and Ye, Jianbo and Hu, Jeffrey and Tancik, Matthew and Angjoo Kanazawa},
  journal={Journal of Machine Learning Research},
  volume={26},
  number={34},
  pages={1--17},
  year={2025}
}
```

We welcome contributions of any kind and are open to feedback, bug-reports, and improvements to help expand the capabilities of this software. Please check [docs/DEV.md](docs/DEV.md) for more info about development.
