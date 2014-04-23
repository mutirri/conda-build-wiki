Overview
========

Continuum's conda and conda-build toolset provides cross-platform binary package management. Originally built to furnish Python distributions, the tools are in active development and being applied in many use cases. This tutorial explores what goes into building conda packages.

For a typical binary package built in a Linux environment with './configure; make; make install', conda packaging can be as simple as issuing a one-line command. Linked libraries are located relative to the binary (rather than referenced with absolute paths) and the software is bundled and ready to ship.

As we'll explore, other package builds are not trivial, especially if complex dependencies are involved or the package was built in an ad-hoc way that does not conform to the usual placement of files relative to one another. This tutorial will move through the gradations of difficulty to illustrate both the potential and challenges of applying conda and conda-build to your software.

Presently, we're focused on bundling packages for Linux. Mac OS X and Windows packages are supported, and future tutorials will cover the additional considerations entailed therein.

Basic Concepts
==============

The following assumes comfort navigating the *NIX command line and doing routine shell script editing and file manipulation. We'll touch on some important concepts and resources you may want to read up on if you encounter something unfamiliar.

Dynamic Linking in One Paragraph
--------------------------------

Why does conda exist? To address a problem that comes about because executables are dynamically linked, meaning they depend on already-compiled libraries that are invoked at runtime. These libraries may be system-level utilities, math packages, or ancillary self-contained tools (like a visualization package). Many applications depend on software like this, which can itself take up a lot of space and involve other dependencies which make it complicated to install. Therefore, it's a desirable model 

Preliminaries
=============

If any of the following are not installed in your Linux environment, you will want to install them:

`sudo apt-get install chrpath`

`sudo apt-get install git`

Install Conda
-------------

conda is installed as part of Continuum's Anaconda distribution and used to manage changes thereto. A lightweight [python + conda standalone distribution](http://conda.pydata.org/miniconda.html) is also available, which is what we'll assume here. Upon downloading the installer script, run it in a working directory:
`>./Miniconda-3.3.0-Linux-x86.sh`

A fundamental design philosophy of conda is that users should have a fully functioning developer programming environment in their 'home' or working directory without requiring administrative privileges or disrupting system- or root-level software installations).