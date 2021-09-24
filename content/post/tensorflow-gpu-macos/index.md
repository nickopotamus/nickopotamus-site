---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Keras and TensorFlow with GPU support on macOS?"
subtitle: "Say what?"
summary: "Say what?"
authors: []
tags: []
categories: []
date: 2021-09-24T18:31:39+01:00
lastmod: 2021-09-24T18:31:39+01:00
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
After my raging success at [getting a GPU-accelerated deep learning stack installed and running on Ubuntu](/post/tensorflow-gpu-ubuntu-20-04), I wondered - could it be done on my MacBook? Third generation 15" MacBook Pros should be capable of running deep-learning on their NVIDIA GPU's, the fourth generation 16" have [relatively beefy AMD cards that run OpenCL](https://www.quora.com/Is-it-possible-to-run-a-TensorFlow-GPU-version-on-an-AMD-graphics-card), and much was made of using the new M1 graphics cores and "[neural engine](https://github.com/hollance/neural-engine)" for rapid deep-learning, so surely it should be doable?

The official party line [from Tensorflow](https://www.tensorflow.org/install/pip) and [from NVIDIA](https://www.provideocoalition.com/officially-official-nvidia-drops-cuda-support-for-macos/) is no. But that hasn't stopped Apple releasing [a Mac-optimised TensorFlow](https://github.com/apple/tensorflow_macos/releases/), and more excitingly, a [platform-optimised graphics API called Metal](https://developer.apple.com/metal/) with a specific [TensorFlow plugin](https://developer.apple.com/metal/tensorflow-plugin/).

This is how I got things working on my Intel-based 16" 2019 MBP, but it *should* also work on the new M1 MBPs.

(You can [also apparently get things working with PlaidML](https://towardsdatascience.com/deep-learning-using-gpu-on-your-macbook-c9becba7c43), but I've not tried that)

## Step 0: Check pre-requesits

You must have:
* A 15/16" Intel MBP running the latest [macOS 11 Big Sur](https://www.apple.com/uk/macos/big-sur/), or a M1 MBP (which comes with Big Sur as default)
* Installed the [Xcode](https://developer.apple.com/xcode/features/) command line tools by running `xcode-select --install` in the terminal
* Installed either [Anaconda](https://www.anaconda.com/), or [Miniforge](https://github.com/conda-forge/miniforge#miniforge3) if on M1 silicon

## Step 1: Set up a virtual environment

I've said before that I adore [Conda virtual environments](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) so I'm going to use one (there are instructions out there for other virtual environments). However, if you do use Conda, it installs [a version of Python that for some reason pretends to be the non-existent macOS 10.16](https://github.com/conda-forge/python-feedstock/issues/445), not macOS 11, which then breaks all the compatibility looking specifically for macOS 11. The easiest way around this is to edit your `~/.zshrc` (or `~/.bash_profile` if using Bash, which you'd know about it you were) to include the following line before starting anything to do with Python:

```
export SYSTEM_VERSION_COMPAT=0
```

#### Intel based Macs:

Now you can set up a virtual environment with the right version of Python, switch into it, and update pip:
```
conda create -n tensorflow-metal python=3.8
conda activate tensorflow-metal
python -m pip install -U pip
```
_Don't_ use the `environment.yml` file [suggested by Apple](https://github.com/apple/tensorflow_macos/issues/153) to set up the environment, because it installs the wrong version of many of the dependencies which you'll have to then roll back one by one manually...

#### Apple silicon:

[Download the Conda environment](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh), then install it, switch into it, and install the required dependencies for Tensorflow using the below code:

```
chmod +x ~/Downloads/Miniforge3-MacOSX-arm64.sh
sh ~/Downloads/Miniforge3-MacOSX-arm64.sh
source ~/miniforge3/bin/activate
conda install -c apple tensorflow-deps
```

## Step 2: Install TensorFlow and the Metal plugin

This _should_ go seamlessly, and install all the other packages you need with it:
```
python -m pip install tensorflow-macos
python -m pip install tensorflow-metal
```

## Step 3: Install Keras in RStudio

Similar to [on Ubuntu](/post/tensorflow-gpu-ubuntu-20-04) we now need to install the [{Keras} package for R](https://keras.rstudio.com/), point it towards our new Conda environment, and install the required packages to make it go:

``` r
install.packages("keras")
library(keras)
use_condaenv("tensorflow-metal")
install_keras()
```

This will take some time, but should pick up your optimised Tensorflow installation, and you'll be away. It installs Python 3.6 into the environment as part of this, but that doesn't seem to compromise the GPU-accelration from the Metal plugin. Try it out by [building a convolutional neural network](https://tensorflow.rstudio.com/tutorials/advanced/images/cnn/) - if the whole thing seemingly grinds to a halt, it hasn't worked!
