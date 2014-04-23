Overview
========

Continuum's conda and conda-build toolset provides cross-platform binary package management. Originally built to furnish Python distributions, the tools are in active development and being applied in many use cases. This tutorial explores what goes into building conda packages.

For a typical binary package built in a Linux environment with `$ ./configure; make; make install`, conda packaging can be as simple as issuing a one-line command. Linked libraries are located relative to the binary (rather than referenced with absolute paths) and the software is bundled and ready to ship.

As we'll explore, other package builds are not trivial, especially if complex dependencies are involved or the package was built in an ad-hoc way that does not conform to the usual placement of files relative to one another. This tutorial will move through the gradations of difficulty to illustrate both the potential and challenges of applying conda and conda-build to your software.

Presently, we're focused on bundling packages for Linux. Mac OS X and Windows packages are supported, and future tutorials will cover the additional considerations entailed therein.

Basic Concepts
==============

The following assumes comfort navigating the UNIX command line and doing routine shell script editing and file/ directory manipulation. We'll touch on some important concepts and resources you may want to read up on if you encounter something unfamiliar.

Linking - Why are we doing this?
--------------------------------

Why does conda exist? To address a problem that comes about because executables depend on already-compiled libraries that are linked against at runtime. These libraries may be system-level utilities, math packages, or ancillary self-contained tools (like a visualization package). Many applications depend on software like this, which can itself take up a lot of space and involve other dependencies which make it complicated to install. Therefore, it's a desirable model to build software that makes use of already-existing software, rather than recompiling every time.

The challenge contained here is that your software must be able to locate compatible builds to link against. If a link target can't be located or is an outdated version, the software fails. conda provides a solution to manage controlled, reproducible environments in which to run software with complex dependencies.

Preliminaries
=============

If any of the following are not installed in your Linux environment, you will want to install them:

`$ sudo apt-get install chrpath`

`$ sudo apt-get install git`

Install conda and conda-build
-----------------------------

conda is installed as part of Continuum's Anaconda distribution and used to manage changes thereto. A lightweight [python + conda standalone distribution](http://conda.pydata.org/miniconda.html) is also available, which is what we'll assume here. Upon downloading the installer script, run it in a working directory:

`$ ./Miniconda-3.3.0-Linux-x86.sh`

A fundamental design philosophy of conda is that users should have a fully functioning programming environment in their home or working directory without requiring administrative privileges or disrupting system- or root-level software installations. Therefore you will not need administrative access to run conda or manage conda environments. Building conda packages may, however require administrative privileges in certain cases.

Provide the script with your choices about where to install conda and whether or not the path will be added to your environment. I will assume it is added to the path; otherwise you will have to know the explicit path to your conda executable.

Once conda is installed, relaunch a terminal window or issue

`$ source ~/.bashrc`

and confirm that

`$ which conda`

finds the executable where you have just installed it. If you did not prepend the conda path (which is the default), the `which` command will not find the conda executable. You'll have to supply its path explicitly or create a soft link, etc.

It's a useful habit to do

` $conda update conda`

with a fresh install. You can issue to update command for any installed package, including conda itself. This intrinsic bootstrapping capacity makes conda very powerful. In fact, if you started with the miniconda installation, you can expand it to the full Anaconda distribution with

`$ conda install anaconda`

but that's not the focus here.

What you do need to install is conda-build:

`$ conda install conda-build`

Clone conda-recipes from github
-------------------------------

This is not a necessary step to build your own packages, but it's a very useful resource to investigate already-built packages as a guide for your task ahead.

`$ git clone https://github.com/conda/conda-recipes`

will establish a copy of the conda-recipes repository on your local disk.

Elementary conda Package Building
=================================
The simplest examples are very simple. With a correct meta.yaml file, this can be a one-liner:

`$ conda build .`

What are other approaches that people might try, with roughly increasing complexity?

Using conda skeleton to build from pypi
---------------------------------------

Building from locally installed software
----------------------------------------

Weird Stuff
===========
conda-build splashes error asking for `conda install jinja2` to enable jinja template support. Build proceeds to completion without, but fails if it's installed with an error `unable to load pkg_resources`.

Difference/ relationship between `conda build .` and `conda-build .`?

