---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Fixing the ModuleNotFoundError in Jupyter"
subtitle: "tl;dr: python -m ipykernel install --user --name=your_environment_here"
summary: "tl;dr: python -m ipykernel install --user --name=your_environment_here"
authors: []
tags: []
categories: []
date: 2021-01-10T22:25:23Z
lastmod: 2021-01-10T22:25:23Z
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
[Python](https://www.python.org/) is great. [It's no R](https://www.reddit.com/r/statistics/comments/8de54s/is_r_better_than_python_at_anything_i_started/), obviously, but an increasing number of machine learning courses are using Python, including [the one I'm currently taking](https://www.inf.ed.ac.uk/teaching/courses/iaml/), so it's worth getting to grips with.

The big problem with Python is that it's so intertwined with most operating systems that doing anything to it - upgrading it, installing packages, etc - runs the very real risk of breaking other pieces of software which depend on it, if not your whole computer. There's also so many different versions out there, both of base Python and the packages it uses, that if you don't have the right ones it might not work as you expect - or at all - when following along with someone else's code without the precise versions that they used.

As such everyone recommends you use [virtual environments](https://realpython.com/python-virtual-environments-a-primer/) which package up a specific version of Python, and allow you to do whatever you want to it without breaking anything outside of this protected environment. Anything you install stays within the environment, which is great because it can't leak out and affect software outside of it, but has the downside that if you then try to use something on the outside (such as the excellent [Jupyter notebook](https://jupyter.org/)), it can't always access the right version of Python and you we're back the unexpected/not working at all problem, often resulting in you tearing your hair out at the infamous ```ModuleNotFoundError: No module named...``` error. 

But there's a quick fix, don't worry.

There's [a fair few different ways of setting up a virtual environment](https://stackoverflow.com/questions/38217545/what-is-the-difference-between-pyenv-virtualenv-anaconda), but I'm going to focus on using [Conda](https://docs.conda.io/en/latest/) here, which comes with the scientist-friendly distribution [Anaconda](https://www.anaconda.com/). Assuming you've installed Anaconda, open the terminal and create yourself a new environment - forcing Python 3.7 (for example) - to play in with:

```
conda update conda
conda create -n py3Env python=3.7
source activate py3Env
```

Once this installs, the terminal which until now started with ```(base) username@machine $``` should instead read ```(py3Env) username@machine $```, signifying that anything Pythonic you now do will occur to py3Env, rather than the base Python installed on your machine.

Let's install some packages then! [Pandas](https://pandas.pydata.org/) is a very useful data handling library, so go install that using the following commands, which will install it, then list all the installed packages in ```py3Env``` which should include it:

```
conda install pandas
conda list
```
 
Great, it's in and working. Lets open up Jupyter with ```jupyter notebook``` and open a new notebook. Set your first command as ```import pandas as pd``` and run it... Oh no!

```
---------------------------------------------------------------------------
ModuleNotFoundError                       Traceback (most recent call last)
<ipython-input-1-5f3260843b53> in <module>
----> 2 import pandas as pd

ModuleNotFoundError: No module named 'pandas'
```

It's definitely installed. You definitely opened Jupyter from ```py3Env```. So why is this happening?

The issue is exactly as described above - Jupyter is looking into the environment from outside, and starts by looking for ```(base)``` Python (and its installed libraries) before using your environment's version. Unless you've fannied around with Python before learning the reason you should be using virtual environments, it won't have Pandas installed, so will error. 

The reason this happens is to do with the [kernel](https://jupyter.readthedocs.io/en/latest/projects/kernels.html), or instance of Python, that Jupyter is using to run. We can run a couple of commands within Jupyter to look at this:

```
!jupyter kernelspec list --json
!which -a python
```

The first gives you details about which version of Python we're running, the second shows a list of where Jupyter looks when you try and run ```python```, and probably looks something like:

```
/Users/username/.pyenv/versions/3.7.7/bin/python
/Users/username/.pyenv/shims/python
/Users/username/opt/anaconda3/envs/py3Env/bin/python
/usr/bin/python
```

We need to force Jupyter to just use the ```/Users/username/opt/anaconda3/envs/py3Env/bin/python``` version of Python, not any of the others (and it's currently defaulting to ```/usr/bin/python```). The best way round this is back to the terminal, and to install the [IPython Kernel for Jupyter](https://github.com/ipython/ipykernel) using:

```
python -m ipykernel install --user --name=py3Env
```

Now check Jupyter, and you should have under ```Kernel > Change Kernel``` (or listed under ```New``` without an open notebook) the option to use the ```py3Env``` kernel. Switch to that, and run your import of pandas again, which should work perfectly... Any bugs from here are on you!





