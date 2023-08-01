---
layout: post
title:  "Installing GSL on Windows"
date:   2023-07-30 12:25:44 +0100
categories: quantatitive, learning
use_math: true
tags: C++, Data Science
published: True
---

A post on installing the [GNU Scientific Library](https://www.gnu.org/software/gsl/doc/html/index.html) on Windows 11 for use with VS Code.

GSL is a C Library for numerical computations. You can use GSL to solve linear system of equations, to dit a curvem numerical integration, statistical calculations etc. My motivation for doing this is part of my journey to learn C++ for its use in  Quantatitive Finance. These are notes so that I can replcate the process later if required.

## Installing a package manager

To provide easy installation of packages and a way to keep them updated I will install [MSYS2](https://www.msys2.org/), which features a package management system called Pacman.

[MSYS2](https://www.msys2.org/) is a collection of tools and libraries providing you with an easy-to-use environment for building, installing and running native Windows software.

If you have already followed the ['Using GCC with MinGW'](https://code.visualstudio.com/docs/cpp/config-mingw#_prerequisites) guide from microsoft to configure th GCC C++, MSYS2 will already be installed and added to `PATH`. Therefore skip to the next step.

If not follow the ['Getting Started Guide'](https://www.msys2.org/)

Add the path to your MinGW-w64 bin folder to the Windows PATH environment variable by using the following steps:

1. In the Windows search bar, type Settings to open your Windows Settings.
2. Search for Edit environment variables for your account.
3. In your User variables, select the Path variable and then select Edit.
4. Select New and add the MinGW-w64 destination folder you recorded during the installation process to the list. If you used the default settings above, then this will be the path: C:\msys64\mingw64\bin.
5. Select OK to save the updated PATH. You will need to reopen any console windows for the new PATH location to be available.

## Installing the GSL libraries

Now MSYS2 is ready, launch the terminal. It should be located in the Start Menu if the defulat installation was followed.

To install GSL use the following commands in the terminal:

```bash
pacman -S mingw-w64-x86_64-gsl
```

More information on the package can be found [here](https://packages.msys2.org/package/mingw-w64-x86_64-gsl).

The libraries should now be installed under the `..\myss64\mingw64\include\` folder.

## Updating the includePath

Add `..\\msys64\\mingw64\\include"` to the `"includePath"` in the `.vscode/c_ccp_properties.json` file in your workspace.
