Overview
========

Continuum's conda toolset provides cross-platform binary package management. Originally built to furnish Python distributions, the tools are in active development and being applied in many use cases. This tutorial explores what goes into building conda packages.

For a typical python-native binary package built in a Linux environment with `$ python setup.py build; python setup.py install`, conda packaging can be as simple as issuing a one-line command. Linked libraries are located relative to the binary (rather than referenced with absolute paths) and the software is bundled and ready to ship.

As we'll explore, other package builds are not trivial, especially if complex dependencies are involved or the package was built in an ad-hoc way that does not conform to the usual placement of files relative to one another. This tutorial will move through the gradations of difficulty to illustrate both the potential and challenges of applying conda to package and distribute your software.

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

A fundamental design philosophy of conda is that users should have a fully functioning programming environment in their home or working directory without requiring administrative privileges or disrupting system- or root-level software installations. Therefore you will not need administrative access to run conda or manage conda environments. Building conda packages may, however, require administrative privileges in certain cases.

Provide the script with your choices about where to install conda and whether or not the path will be added to your environment. I will assume it is added to the path; otherwise you will have to know the explicit path to your conda executable.

Once conda is installed, relaunch a terminal window or issue

`$ source ~/.bashrc`

and confirm that

`$ which conda`

finds the executable where you have just installed it. If you did not prepend the conda path (the default option), the `which` command will not find the conda executable. You'll have to supply its path explicitly or create a soft link, etc.

It's a useful habit to do

`$ conda update conda`

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
Trivial
-------
The simplest examples are very simple. With a correct meta.yaml file and a properly bundled binary distribution hosted on binstar, this can be a one-liner (e.g. Trent's ppt demo with pyfaker):

`$ conda build .`

What are other approaches that people might try, with roughly increasing complexity?

Using conda skeleton to build from a PyPi package
-------------------------------------------------
Confirm that the package is hosted by PyPi. Here I use the music21 package, motivated by a [recent request](https://groups.google.com/a/continuum.io/forum/#!searchin/anaconda/conda$20package/anaconda/yu2ZKPI3ixU/VSWejiDoXlQJ) on the [Anaconda support list](https://groups.google.com/a/continuum.io/forum/#!forum/anaconda). It turns out this has already been packaged for conda, but it serves its purpose as an example here.

`$ pip install music21 -E ~/miniconda_local`

Here miniconda_local is the top_level directory where I've installed my conda distribution. Explicitly, the conda and conda-managed python executables are located in miniconda_local/bin. The -E flag specifies the path extension of the python distribution into which the package should be installed (it will default to /usr otherwise, but you want it to be part of the conda-managed environment).

If this goes well, you now have an installed music21 package from which to build a conda package:

`$ conda skeleton pypi music21 --no-download`

The --no-download flag simply prevents the tarball from being downloaded again, to save a couple minutes, since we just did that. You should verify the existence of the meta.yaml, build.sh, and bld.bat files in a newly created directory called music21.

Now, it should be straightforward to use the conda-build tool. Let's try it:

`$ cd music21`

`$ conda build .`

The workflow and file management can be a nuisance here. Specifically my build effort failed because a music21 tarball (**due to the skeleton command or a partially successful conda build command??**) was already sitting in miniconda_local/conda-bld/src_cache. Apparently then conda build could not write this file and bailed with a messy and unclear error. Moving that music21*tar.gz file out of there resolved the issue. **This tutorial should be sequenced in such a way that the workflow is better and that conflict is avoided.**

That worked.

Although if I now `$ conda install pkgs/music21-1.8.1-py27_0.tar.bz2` using the package I just built it fails with

    shutil.Error: `pkgs/music21-1.8.1-py27_0.tar.bz2` and `/home/gergely/code/miniconda/pkgs/music21-1.8.1-py27_0.tar.bz2` are the same file`

**Why is that not a sensible thing to do at this point?**

Writing meta.yaml by hand
-------------------------
Suppose we stick with the same package, music21, but don't start from the pip installation. We can use common sense values for the meta.yaml fields, based on other conda recipes and information about where to download the tarball. To furnish a detailed failure mode, I'll take the meta.yaml file from the pyfaker package:

    package:
      name: pyfaker
      version: 0.3.2
    
    source:
      git_tag: 0.3.2
      git_url: https://github.com/tpn/faker.git
    
    requirements:
      build:
        - python
        - distribute
    
      run:
        - python
    
    test:
      imports:
        - faker
    
    about:
      home: http://www.joke2k.net/faker
      license: MIT

With a search on github and some sensible choices for substitutions, I get a makeshift .yaml for music21:

    package:
    
      name: music21
    
      version: 1.8.1
    
    source:
    
      git_tag: 1.8.1
    
      git_url: https://github.com/cuthbertLab/music21/releases/download/v1.8.1/music21-1.8.1.tar.gz
    
    requirements:
    
      build:
    
        - python
    
        - distribute
    
      run:
    
        - python
    
    test:
    
      imports:
    
        - music21
    
    about:
    
      home: https://github.com/cuthbertLab/music21
    
      license: LGPL
    
This seems reasonable. Being sure to supply build.sh and bld.bat files in the same directory, I try

`$ conda build .`

and get a 403 error trying to access the repository. Now, with the benefit of comparison with the skeleton-generated file, I observe that the key difference is in the keywords that specify the git repository:

      fn: music21-1.8.1.tar.gz
    
      url: https://github.com/cuthbertLab/music21/releases/download/v1.8.1/music21-1.8.1.tar.gz`

versus

      git_tag: 1.8.1
    
      git_url: https://github.com/cuthbertLab/music21/releases/download/v1.8.1/music21-1.8.1.tar.gz`

**What is the significance of this difference, and how should I know which set of keywords to use?** But with this substitution, it works, and I have a conda package as desired.

Building from locally compiled source distribution
--------------------------------------------------
**The next step will be building from source without the training wheels of skeleton or pip. Perhaps this is the place to slot in the continuum library?**

Issues/ Weird Stuff/ Needs Attention
===========
* conda-build splashes error asking for `conda install jinja2` to enable jinja template support. Build proceeds to completion without, but fails if it's installed with an error `unable to load pkg_resources`.

* Difference/ relationship between `conda build .` and `conda-build .`?

* How should a user intelligently search for existing packages without advance knowledge of all the channels they ought to search. Is there a utility that crawls binstar or a protocol for package builders to register their packages in a central place that can be accessed by conda search (if particular channels have not already been added)?

References
==========
[Using PyPi packages for conda](http://www.linkedin.com/today/post/article/20140107182855-25278008-using-pypi-packages-with-conda)
[music21 inquiry on support list](https://groups.google.com/a/continuum.io/forum/#!searchin/anaconda/conda$20package/anaconda/yu2ZKPI3ixU/VSWejiDoXlQJ)