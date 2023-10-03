---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Building a website in RStudio using Hugo and Netlify"
subtitle: ""
summary: ""
authors: []
tags: ["R", "Web"]
categories: []
date: 2020-11-03T13:39:54Z
lastmod: 2020-11-03T13:39:54Z
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
After running on various versions of home-brew software and Wordpress, I got fed up trying to force PHP and MySQL into behaving and transitioned my website over to [Jekyll](https://jekyllrb.com/) running on [Netlify](https://www.netlify.com/). However Jekyll isn't the most userfriendly of software, and trying to make Ruby run properly every time I updated macOS was putting me off writing because I'd then spend a day trying to publish it! 

Recently I've been inspired by fellow anaesthetist [Chris Tomlinson's](https://ctomlinson.net) use of [Hugo](https://gohugo.io/) and [Github pages](https://pages.github.com/) to run a static website, and as an R-zealot the fact I can run it entirely from within the One True IDE [RStudio](https://rstudio.com) had me sold. So with a few modifications and updates to [his installation instructions](https://ctomlinson.net/post/building-a-website-with-hugo-academic/) here is how I got this site running on Netlify using [Wowchemy](https://wowchemy.com/)'s [Academic template](https://academic-demo.netlify.app/)...

## Step 1: Install Hugo

First step is to install Hugo and its dependencies. Specific [instructions by OS can be found here](https://gohugo.io/getting-started/installing/), but on macOS you'll need to start with [Homebrew](https://brew.sh/) (you mean you haven't already got it?), updating it, and installing Hugo and required dependencies:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
brew update && brew upgrade
brew install git golang hugo
```

Next step is to update  ```.zshrc``` (or ```.bashrc```) to point Hugo at Go by running ```nano ~/.zshrc``` and adding the line:

```
export PATH=$PATH:/usr/local/go/bin
```

## Step 2: GitHub

If you don't already have one, register for a [GitHub account](https://github.com/join?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home). You don't need to know Git, but if you want a quick exploratory tutorial [the Software Carpentary tutorial](https://swcarpentry.github.io/git-novice/) is a good start, and the [GitHub desktop app](https://desktop.github.com/) is a good cheat to avoid terminal errors.

When you're ready fork the [Hugo Academic template](https://github.com/wowchemy/starter-academic) into your own repository ```https://github.com/YOURUSERNAME/starter-academic```.

## Step 3: RStudio

Create a new project in [RStudio](https://rstudio.com/) from your forked template by selecting File > New Project... > Version Control > Git. Use your fork of the starter project URL and select where you want it saved (a good place to save it is the ```~/Documents/GitHub``` folder that GitHub desktop uses). Then hit Create Project, and wait while it downloads all the files.

This will open a new project based on all the included ```academic.Rproj``` file, and open the file structure of the template in the RStudio Files pane.

You also need to install [Blogdown](https://bookdown.org/yihui/blogdown/) to allow R to run Hugo, and include it in the environment, by running in the console:

```
install.packages("blogdown")
library(blogdown)
```

To run the website, use ```blogdown:::serve_site()```. It will appear in the Viewer pane, but renders more accurately if you visit the URL that blogdown outputs to the Console (eg ```Serving the directory . at http://127.0.0.1:4321```) using Chrome/Safari/etc.

## Step 4: Netlify

[Login to Netlify](https://app.netlify.com/) using your GitHub credentials. Fill in the registration details and you'll be taken to your "Team Overview" (yes, you're a team of one). Under the "YOURNAME's team" box is "Sites", so select the green "New site from Git" button, which allows you to automatically process your GitHub repository and publish it as a website:

1. Connect to GitHub for continuous deployment (it may reauthenticate)
2. Select the repository ```YOURNAME/starter-academic```. 
3. You'll get a number of build options. Owner should remain you, branch to deploy should be master, and under "Basic build settings" the build command should be ```hugo --gc --minify -b $URL``` and the publish directory set to ```public```.

Hit deploy site, wait for it to build, and you should be rewarded with a lovely website (with a big green top telling you to edit it). It'll probably have a stupid name like ```lemon-upside-fish.netlify.app``` but it's trvially easy to change this to something meaningful, or even link it to a domain name you own.

## Step 5: Edit and update your site

Hugo uses [fairly human readable Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) so writing new content and building an elegant website is relatively straightforward. You can open the files directly from the Files pane, edit them in RStudio, and when you hit save it should automagically update in your browser. Have a play using [the Wowchemy documentation](https://wowchemy.com/docs/get-started/) and make it lovely and pretty.

Now for the magic. Netlify will watch for updates to the Git repository and run Hugo automatically to publish the site. All you have to do is push the updates to GitHub, either by cheating and using the Desktop app, or by switching to the Terminal within RStudio and executing:

```
git add .
git commit -m "Write a comment about it here"
git push -u origin master
```

Enter your username and password, and Netlify will perform witchcraft in the background and publish your site. 

![alt text](https://media3.giphy.com/media/3o84U6421OOWegpQhq/giphy.gif?cid=ecf05e47om608ounov53ld7p5itn7mddv1qhkesi6yteaecc&rid=giphy.gif "Magic")
