---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Installing Keras and Tensorflow with GPU support on Ubuntu 20.04 LTS"
subtitle: "Just follow the instructions and it'll all be fine"
summary: "Just follow the instructions and it'll all be fine"
authors: []
tags: []
categories: []
date: 2021-09-07T04:02:05+01:00
lastmod: 2021-09-07T04:02:05+01:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
Everyone who's dabbled in deep-learning knows [TensorFlow](https://www.tensorflow.org/) and its wrapper [Keras](https://keras.io/), but few realise it [runs beautifully in R](https://tensorflow.rstudio.com/tutorials/). But to really experience the benefits of the library, [you need to run it using a GPU](https://tensorflow.rstudio.com/installation/gpu/local_gpu/), which can be a really pain in the backside anyway, and especially a pain to get workin in R. 

This is a quick walk-through of how I eventually got it working with a RTX 2080 Super on [Ubuntu 20.04](https://releases.ubuntu.com/20.04/), but it should work on most Linux distributions with most GPUs. In essence, you have three options:
1. Install drivers, CUDA, cuDDN, and TensorFlow manually
2. Use Lamba Lab's [Lamba Stack](https://lambdalabs.com/lambda-stack-deep-learning-software) to install a whole heap of packages together with drivers
3. Use the [NVIDIA Data Science Stack](https://github.com/NVIDIA/data-science-stack) to handle everything NVIDIA-related

Having used all three, the **NVIDIA Data Science Stack** seems to automate everything the most painlessly, ensuring all the multiple disprate packages install the right ones that are interoperable (which is not the most recent of anything, be warned) and plays nicest with R's implementation of Keras (which was my issue with the Lambda Stack). Instructions for that approach are what I recommend, but in case you want to try and do things manually, I've also provided a walkthrough for that too...

# tl;dr: Use the NVIDIA Data Science Stack

## Step 1: Clone and run the NVIDIA Data Science Stack

Fetch the script from the NVIDIA GitHub repository, and run it:

```
git clone https://github.com/NVIDIA/data-science-stack
cd data-science-stack
./data-science-stack setup-system
```

This will ask for your root password when it needs it (_do not_ run using sudo) and do things like sort your NVIDIA drivers out properly and get all the required packages installed. It'll then invite you to reboot the system (which you have to do manually).

## Step 2: Set up a data-science-stack virtual environment

Now you've got two choices - install the stack into a container, or into a [virtual environment]((https://realpython.com/python-virtual-environments-a-primer/)). I'm a big fan of [using virtual environemtns for development](https://stackoverflow.com/questions/50974960/whats-the-difference-between-docker-and-python-virtualenv), which is most of the player we're going to be doing, before using [Docker containers](https://www.docker.com/resources/what-container) to deploy any finished products (which is unlikely). As such, I recommend that you install the stack into a [Conda virtual environment](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) using:

```
./data-science-stack build-conda-env
```

Check that everything has installed properly using `./data-science-stack run-jupyter` and try running one of [the included demo notebooks](https://github.com/rapidsai/notebooks).

## Step 3: Make RStudio play nicely

Finally, we need to install Keras and Tensorflow - or rather the corresponding R packages, because Tensorfloww has already been installed in the stack. This is as simple as opening RStudio, and running:

``` r
install.packages("keras")
```

This will also install the R package for Tensorflow, and a few other dependencies, however [despite what the RStudio instructions say](https://tensorflow.rstudio.com/installation/) you _do not_ need to run `install_keras()`, and instead just point R at the Conda environment you set up previously:

``` r
library(keras)
use_condaenv("data-science-stack-2.9.0")
```

Test that it is all working by creating a TensorFlow object and get cracking.

``` r
tf$constant("Hello Tensorflow")
```

# The masochist's approach: Install everything manaully

If you want to be a Linux purist, you can install everything by hand. If you're _desperate_ to do so, this is the approach you need to take:

## Step 0: Start with a clean installation

Remove all NVIDIA related drivers and software with the following, then reboot.

```
# Remove CUDA Toolkit and associated packages:
sudo apt-get --purge remove "*cublas*" "*cufft*" "*curand*" "*cusolver*" "*cusparse*" "*npp*" "*nvjpeg*" "cuda*" "nsight*" 
# Remove NVIDIA Drivers:
sudo apt-get --purge remove "*nvidia*"
# Clean up the uninstall:
sudo apt-get autoremove
```

## Step 1: Use NVIDIA drivers

This is the most straightforward step - open ```Software & Updates```, select "Additional Drivers", and chose the most recent proprietary Nvidea driver set (470 at the time of writing). Hit "Apply". Job done. 

## Step 2: Install the CUDA toolkit

CUDA (Compute Unified Device Architecture) is a specific set of drivers for the NVIDIA GPU allowing it to run a low-level programming language for parallel computing that TensorFlow can hook into. Fortunately, NVIDIA provide a (several Gb) package that installs it all for us; unfortunately, that package updates regularly and so we need to find one that fits our version of Linux, graphics card, and drivers.

To do so, visit the [CUDA toolkit download page](https://developer.nvidia.com/cuda-downloads) and work through the tick boxes to select out version (of follow [this shortcut](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_network), but I can't guarantee how long it'll work for):
![CUDA options](/post/tensorflow-gpu-ubuntu-20-04/cuda_selection.png)

This should result in you being offered something similar to the below, which you should follow, **except for the final line which I've changed**. Using ```sudo apt-get -y install cuda``` as suggested by NVIDIA will install the latest and greatest CUDA, 11.4. Unfortunately (at the time of writing) [Tensorflow 2.6 is only compatible with CUDA 11.2](https://www.tensorflow.org/install/source#gpu), so instead you need to specify the earlier version to be installed:

```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda-11-2
```

Now is a great opportunity for a cup of coffee while it downloads. You may be tempted to download the local ```.deb``` installer instead; if you do, just remember to remove the local apt repository that it creates, otherwise it breaks all future NVIDIA related updates. I mention this from bitter experience!

## Step 3: Install cuDNN

CUDA Deep Neural Network (cuDNN) is a library used for further optimizing neural network computations, using the CUDA API. Again, this involves installing the right version, but also [registering for a (free) NVIDIA developers account to get hold of it](https://developer.nvidia.com/cudnn). Once signed up, and agreeing to practice only ethical AI, you need to select the version that works with our CUDA (currently CUDA 11.**2**, see above). This means looking through the Archived cuDNN releases for the most recently supported version [cuDNN SDK 8.1.0 for CUDA 11.2](https://www.tensorflow.org/install/gpu) for our version of Linux (cuDDN Library for Linux (x86_64)), which is [currently downloadable from this link](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/8.1.0.77/11.2_20210127/cudnn-11.2-linux-x64-v8.1.0.77.tgz).

Sit tight, or go for another coffee. Once you've got it, head to the Downloads folder, and extract the tarball before copying files into our CUDA installation (replace ```cudnn-11.2-linux-x64-v8.1.0.77.tgz``` with whatever you've just downloaded):

```
tar xvf cudnn-11.2-linux-x64-v8.1.0.77.tgz
sudo cp -P cuda/lib64/* /usr/local/cuda/lib64/
sudo cp cuda/include/* /usr/local/cuda/include/
```

## Step 4: Update paths

Finally, we have to configure some enviroment variables to help the system locate these libraries, then reload bash:

```
echo 'export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"' >> ~/.bashrc
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
echo 'export PATH="/usr/local/cuda/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## Step 5: Install Keras and Tensorflow with GPU support

Moving back to RStudio, we need to install the Keras and TensorFlow packages with:

``` r
install.packages("keras")
library(keras)
install_keras(tensorflow = "gpu")
```

If you haven't already got [Python virtual environments](https://realpython.com/python-virtual-environments-a-primer/) set up, it'll invite you to install [miniconda](https://docs.conda.io/en/latest/miniconda.html) (if you have, and want to manage them yourself, see [the instructions on the RStudio website](https://tensorflow.rstudio.com/installation/custom/)). Otherwise, just let RStudio do its thing.

This _should_ just work, but the main pitfall here is the new virtual environment not finding CUDA library files, even after adding links to `PATH`. It initially worked for me, but after a reboot couldn't find them at all, even after manually updating the path to point directly at them. A solution is - [apparently](https://github.com/tensorflow/tensorflow/issues/45930) - to install Conda's [cudatoolkit](https://anaconda.org/anaconda/cudatoolkit) into the virtual environment, but this now involves once again selecting the right version for your CUDA and cuDDN, and putting it in the right place, which is the point where I gave up and resorted to the NVIDIA DSS...

# Where do we go from here?

I heartily recommend working through François Chollet's [Deep Learning with R](https://www.manning.com/books/deep-learning-with-r). Given [he created Keras](https://en.wikipedia.org/wiki/Fran%C3%A7ois_Chollet), it's a pretty nifty insight into how to use these powerful tools in R.








