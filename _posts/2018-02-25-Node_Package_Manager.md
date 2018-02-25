---
layout: post
post: true
title: Node Package Manager
date: 2018-02-25
category: Angular
tag: NPM
author: Feng Yuan
---

* content
{: toc}




**This tutorial tells you how to use the Angular framework to establish new projects from the ground up.**

## Prerequisites

1. NPM package manager uses a very different way in managing dependencies. It allows a lot of redenduncy. Sharing one unique module with all other dependent modules can always be a nasty problem. Although there are some methods to enable sharing to some degree, such as the `npm dedupe` command, yet it is not suggested. Thus, it is difficult to manage the packages as a common framework(not impossible). Node packages usually stick closely to your project folder.

2. It is possible to symlink a common framework somewhere in your computer to your project folder, such as using the `npm-link-shared` or `npm link` tools. Yet, these commands always first create symlinks in your global *node_modules* folder and then create second symlinks in your project's *node_modules* folder. This can be messy sometimes.

3. If you re-install every node module in each of your project *node_modules* folder, it is not only time-comsuming when creating a new project, but also space-comsuming on your hard drives.

4. Taking all the above into consideration, I decide the best way is to store a global *package.json* file in a safe place as your common framework. When you need to create new projects, use this file to install all the required modules in the common *node_modules* folder in your project folder.

## Directory Strcuture

We organise the projects directory as follows:

{%highlight zsh%}
+Projects
++node_modules
++package.json
++ProjectA
+++node_modules
+++src
++ProjectB
+++node_modules
+++src
++ProjectC
+++node_modules
+++src
{%endhighlight%}

## Linux Code to Create a New Project

{%highlight zsh%}
mkdir Projects
cd Projects
cp /path/to/the/common/package.json .
npm install
mv package.json package.json.bak                    //This is important, otherwise the ng command will not create a new project
ng new ProjectA --skip-install
ng new ProjectB --skip-install
ng new ProjectC --skip-install
mkdir ProjectA/node_modules                         //This is also important, since ng will always look at the local project node_modules folder first even if it is empty
mkdir ProjectB/node_modules
mkdir ProjectC/node_modules
cd ProjectA
ng serve
{%endhighlight%}

## Final Remarks

The common packages used by all A,B,and C projects are stored in the project root *node_modules* folder. In each project folder, local modules can be installed in the local *node_modules* directory. ng will first look for modules in the local *node_modules* directory, then look for modules in the root *node_modules* folder. Thus, we can allow some kind of sharing of node modules among projects in this way.

If you like, you can install all required node packages globally, and then symlink them into your project folder. That can be annoying sometimes because you have to store a lot of packages of different versions in your global folder, most of which you do not need for a specific project. Node follows a very different package management philosophy, so that sharing some unique library across the whole system disobeys this philosophy.

Therefore, keep everything as locally as possible and only share modules locally across a few similar projects. You can store different *package.json* files as different development environments. When you need to create an environment, simply copy the related .json file and `npm install` all the packages. Avoid using symbolic links. NPM is project-oriented.
