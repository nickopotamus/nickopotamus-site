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
Everyone who's dabbled in deep-learning knows [TensorFlow](https://www.tensorflow.org/) and its wrapper [Keras](https://keras.io/), but few realise it [runs beautifully in R](https://tensorflow.rstudio.com/tutorials/). But to really experience the benefits of the library, [you need to run it using a GPU](https://tensorflow.rstudio.com/installation/gpu/local_gpu/). This is a quick walk-through of how I eventually got it working with a RTX 2080 Super on [Ubuntu 20.04](https://releases.ubuntu.com/20.04/).

## Step 1: Use Nvidia drivers

This is the most straightforward step - open ```Software & Updates```, select "Additional Drivers", and chose the most recent proprietary Nvidea driver set (470 at the time of writing). Hit "Apply". Job done. 

## Step 2: Install the CUDA toolkit

CUDA (Compute Unified Device Architecture) is a specific set of drivers for the Nvidia GPU allowing it to run a low-level programming language for parallel computing that TensorFlow can hook into. Fortunately, Nvidia provide a (several Gb) package that installs it all for us; unfortunately, that package updates regularly and so we need to find one that fits our version of Linux, graphics card, and drivers.

To do so, visit the [CUDA toolkit download page](https://developer.nvidia.com/cuda-downloads) and work through the tick boxes to select out version (of follow [this shortcut](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_network), but I can't guarantee how long it'll work for):
![CUDA options](/post/tensorflow-gpu-ubuntu-20-04/cuda_selection.png)

This should result in you being offered something similar to the below, which you should follow, **except for the final line which I've changed**. Using ```sudo apt-get -y install cuda``` as suggested by Nvidia will install the latest and greatest CUDA, 11.4. Unfortunately (at the time of writing) [Tensorflow 2.6 is only compatible with CUDA 11.2](https://www.tensorflow.org/install/source#gpu), so instead you need to specify the earlier version to be installed:

```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda-11-2
```

Now is a great opportunity for a cup of coffee while it downloads. You may be tempted to download the local ```.deb``` installer instead; if you do, just remember to remove the local apt repository that it creates, otherwise it breaks all future Nvidia related updates. I mention this from bitter experience!

## Step 3: Install cuDNN

CUDA Deep Neural Network (cuDNN) is a library used for further optimizing neural network computations, using the CUDA API. Again, this involves installing the right version, but also [registering for a (free) Nvidia developers account to get hold of it](https://developer.nvidia.com/cudnn). Once signed up, and agreeing to practice only ethical AI, you need to select the version that works with our CUDA (currently CUDA 11.**2**, see above). This means looking through the Archived cuDNN releases for the most recently supported version [cuDNN SDK 8.1.0 for CUDA 11.2](https://www.tensorflow.org/install/gpu) for our version of Linux (cuDDN Library for Linux (x86_64)), which is [currently downloadable from this link](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/8.1.0.77/11.2_20210127/cudnn-11.2-linux-x64-v8.1.0.77.tgz).

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

Finally, once everything has got itself sorted, you can test that it is all working by creating a TensorFlow object:

``` r
library(tensorflow)
tf$constant("Hellow Tensorflow")
```

## Where do we go from here?

I heartily recommend working through François Chollet's [Deep Learning with R](https://www.manning.com/books/deep-learning-with-r). Given [he created Keras](https://en.wikipedia.org/wiki/Fran%C3%A7ois_Chollet), it's a pretty nifty insight into how to use these powerful tools in R.



