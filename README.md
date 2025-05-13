# gsplat 3DGUT Tutorial

## About this project fork - READ FIRST

I created this fork from the oiginal gsplat project to guide users through installing gsplat on Windows PCs and running 3DGUT. I ommitted basic information on running other gsplat training models. Please refer to the original project for the latest developments as this fork does not contain future developments and will not be maintained long term.

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

<br>

### Creating your environment
```
conda create --name gsplat -y python=3.10
conda activate gsplat
```
<br>

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

### Install Pytorch

For CUDA Toolkit 12.6 
```
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
```

For CUDA Toolkit 11.8
```
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

**Note:** Refer to [Pytorch.org](https://pytorch.org/get-started/locally/) for additional CUDA Toolkit distributions
<br><br>

### Clone the `gsplat` repository:
```bash
git clone --recursive https://github.com/nerfstudio-project/gsplat.git
cd gsplat
```

<br><br>

### Install `gsplat` and requirements using pip:
```bash
pip install -e .
pip install -r examples/requirements.txt
```

<br><br>

### Replace scene_manager.py file
The `scene_manger.py` file that comes packaged with the project is incompatible with Windows. Replace the file the version in the `assets` folder. [You can download it directly here](https://github.com/jonstephens85/gsplat_3dgut/blob/main/assets/scene_manager.py).

Replace the existing `scene_manager.py` file with the downloaded python script in your: C:\Users\<username>\anaconda3\envs\gsplat\Lib\site-packages\pycolmap **Note: replace the <username> with your username.**

_Congratulations, gsplat is installed and ready to train scenes with 3DGUT!_

<br><br>

## 2. Prepare your data

This section covers how to prepare your data for training. As more image types are supported, I will expand on this section.

Required software:
- COLMAP - [Download the latest Windows binary](https://github.com/colmap/colmap/releases) release and install it.
- Imagemagick (for image downsampling) - [download the installer here](https://imagemagick.org/script/download.php#windows).
<br><br>


The data format is expected to be in the same format as the original 3DGS project, however, cameras, images, and points3d need to be text format:
```
<location>
|---images
|   |---<image 0>
|   |---<image 1>
|   |---...
|---sparse
    |---0
        |---cameras.bin
        |---images.bin
        |---points3D.bin
```
<br><br>

### Downsampling Images
You may run out of memory if you have low VRAM and/or are using too many high resolution images. Downsampling will reduce the amount of VRAM needed and speed up training. I suggest keeping images around 2k-4k pixels on the longest dimension.

1. Install Imagemagick if not already installed
2. Make a downsample directory and downsample your images. This example is for half scale. For quarter, eighth scale, etc. call your folder images_4, images_8, etc. and change the resize value to 25%, 12.5%, etc.:
```
mkdir images_2
magick mogrify -path images_2 -resize 50%% images\*.jpg
```

Your resulting folder should look like this:
```
|---images
|   |---<image 0>
|   |---<image 1>
|---images_2
|   |---<image 0>
|   |---<image 1>
```
<br><br>

### Running COLMAP
COLMAP is a specialized software for running structure from motion. This software will determine where each photo was taken in 3D space as well as build a 3D point cloud for you.

If you alread have it installed and are comfortable using it in terminal, follow these commands:
```
mkdir sparse

# Use images_2 if you want downsampled images
# Change to OPENCV_FISHEYE for fisheye lenses
# Remove --ImageReader.single_camera if using more than one camera
colmap feature_extractor --database_path database.db --image_path images --ImageReader.camera_model SIMPLE_PINHOLE --ImageReader.single_camera 1

colmap exhaustive_matcher --database_path database.db

# Use images_2 if you want downsampled images
colmap mapper --database_path database.db --image_path images --output_path sparse
```

For Newbies, the GUI path is much easier to follow.

Step 1: Launch the GUI and set up a new project
- Launch COLMAP using `colmap gui` in terminal. A GUI will launch. From there, select `File > New Project`
- Select `New` for the Database and create a new database file called `database.db` in your project's root folder (not the images folder)
- For images, select the `images` folder or `images_2` folder if it exist. Using the downsampled images will run faster and often yields better results!

Step 2: Run Feature Extraction
- Select `Processing > Feature Extraction`
- For Camera model select `PINHOLE`, `SIMPLE_PINHOLE`, or `OPENCV_FISHEYE` depending on your camera. I suggest unless you are using a fisheye camera, you should choose `SIMPLE_PINHOLE`
- Checkmark `Shared for all images` if you used one camera for the entire scene.
- Click Extract - this should only take a minute or two to run.

Step 3: Feature Matching
- Select `Processing > Feature Matching`
- For randomly captured images, use the `Exhaustive` tab, for images taken in sequence, use the `Sequential` tab.
- Leave parameters at default and click `Run` for large datasets this could take a while...and yes, it runs primarily on CPU.

Step 4: Reconstruction
- Select `Reconstruction > Start Reconstruction` - this can take a while...and again, it runs on CPU.
- You will see the scene incrementally build before your eyes. Grab popcorn and enjoy!

Step 5: Export
- Select `File > Export Model`
- In the export folder dialog box, create a new folder in your project folder called `sparse`. Within the newly created sparse folder, create another folder called `0`. Save your files into the 0 folder. Refer to my file structure reference above.

BOOM! You are ready to create your very own 3DGUT scene!!!

<br><br>


## Training

Simply passing in `--with_ut --with_eval3d` to the `simple_trainer.py` arg list will enable training with 3DGUT! And note in gsplat they only support MCMC densification strategy for 3DGUT.


```
# With fisheye camera
python examples/simple_trainer.py mcmc --data_dir data/<dataset> --data_factor 2 --result_dir results/<results name> --camera_model fisheye --save_ply --with_ut --with_eval3d 

# With pinhole camera (regular camera)
python examples/simple_trainer.py mcmc --data_dir data/<dataset> --data_factor 2 --result_dir results/<results name> --camera_model pinhole --save_ply --with_ut --with_eval3d 
```

**Note on the live viewer:** the path to view it in browser did not work on my PC. Try using [http://localhost:8080/](http://localhost:8080/) to launch the viewer.

<br><br>


## Rendering

Once trained, you could view the 3DGS and play with the distortion effect supported through 3DGUT via our viewer:

```bash
python examples/simple_viewer_3dgut.py --ckpt results/<training result file>.pt 
```

Or a more comprehensive nerfstudio-style viewer to export videos. (note changing distortion is not yet supported in this comprehensive viewer!)
```bash
python examples/simple_viewer.py --with_ut --with_eval3d --ckpt results/<training result file>.pt 
```

<br><br>

## For users using gsplat' API:
To use the 3DGUT technique The relavant arguments in `rasterization()` function are:
- Setting `with_ut=True` and `with_eval3d=True` to enable 3DGUT (which is consist of two parts: using unscented transform to estimate the camera projection and evaluate Gaussian response in 3D space.)
- To train/render pinhole camera with distortion, setting the distortion parameters to `radial_coeffs`, `tangential_coeffs`, `thin_prism_coeffs`.
- To train/render fisheye camera with distortion, 
setting the distortion parameters to `radial_coeffs` and set `camera_model="pinhole"`
- To enable rolling shutter effects, checks out `rolling_shutter` and `viewmats_rs` on the type of rolling shutters we supported.

<br><br>

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
