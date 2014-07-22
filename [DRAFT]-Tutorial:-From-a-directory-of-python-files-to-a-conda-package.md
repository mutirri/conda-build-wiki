This episode starts with the most basic scenario. I have a directory containing
python code. It does not have dependencies. I want to bundle it and make it
available as a conda package.

The basic steps are as follows (for a Linux environment):
1. Format files to be compatible with a setup.py build process.
2. Add and edit setup.py.
3. Create a tarball of the distribution.
4. Create and edit conda build files: `meta.yaml`, `build.sh`, `bld.bat`.
5. Perform the package build with conda.
6. Place the package in a repository on (binstar.org)[https://binstar.org/].
7. Add the channel and conda install!

**Using tpn's continuum library of python utilities, here's what this looks like, step by step.**

1. Format files to be compatible with a setup.py build process
--------------------------------------------------------------

Our initial setup is a collection of .py files in a directory:

```
$ cd python-libcontinuum/
$ ls -R
.:
lib  LICENSE  README.md

./lib:
continuum

./lib/continuum:
cli.py               command.pyc  debug.py      invariant.py  path.py        util.pyc
commandinvariant.py  commands.py  __init__.py   logic.py      pipe_win32.py
command.py           config.py    __init__.pyc  logic.pyc     util.py
```

This is what we want to bundle and distribute as a conda package.
We'll create a new directory and assemble everything we want:

```
$ mkdir continuum-python
$ cp -r python-libcontinuum/lib continuum-python/
$ ls -R continuum-python
.:
continuum

./continuum:
cli.py               command.pyc  debug.py      invariant.py  path.py        util.pyc
commandinvariant.py  commands.py  __init__.py   logic.py      pipe_win32.py  _version.py
command.py           config.py    __init__.pyc  logic.pyc     util.py
```

2. Add and edit setup.py, and test it
-------------------------------------

Now we need to place a setup.py file in the top-level directory of the package.
I took one from a similar distribution and copied it, then edited by hand.

```
$ cd continuum-python
$ cp ../some_other_package/setup.py .
```

Basic edits involve substituting your package name and details about the
author, version, license, etc. Note that setup.py makes use of a file called
_version.py in the source directory. This file should be edited or created with
this result:

```
$ more continuum/_version.py
__version_info__ = (0, 1, 0)
__version__ = '.'.join(str(x) for x in __version_info__)
```

where the tuple __version_info__ contains the correct version for your software.

Verify that the build and install commands function correctly:

```
$ python setup.py build
$ python setup.py install
```

Make any edits if errors occur.

3. Create a tarball of the distribution
---------------------------------------

You now need to bundle this package and place it in a repository where it can
be downloaded and distributed. First, create a tarball of the files you've just
assembled:

```
$ pwd
~/sandbox/continuum-python
$ tar -czvf continuum.tar.gz ./*
./*
./continuum/
./continuum/cli.py
./continuum/command.py
./continuum/command.pyc
./continuum/commandinvariant.py
./continuum/commands.py
./continuum/config.py
./continuum/debug.py
./continuum/invariant.py
./continuum/logic.py
./continuum/logic.pyc
./continuum/path.py
./continuum/pipe_win32.py
./continuum/util.py
./continuum/util.pyc
./continuum/_version.py
./continuum/__init__.py
./continuum/__init__.pyc
./setup.py
```

4. Create and edit conda build files: meta.yaml, build.sh, bld.bat
------------------------------------------------------------------

Create a build directory and include in it the conda build files (steal 'em
from somewhere).

```
$ mkdir continuum-build-trial
$ cd continuum-build-trial/
$ cp ../continuum-conda-pkg/meta.yaml .
$ cp ../continuum-conda-pkg/build.sh .
$ cp ../continuum-conda-pkg/bld.bat .
```

Special Case - Building locally
-------------------------------

This can be built from a local tarball. In that case, it needs to also be
placed in the build directory...

```
$ cp ../cotinuum-python/continuum.tar.gz .
```

... and the conda build files look like this:


```
$ more meta.yaml

package:
  name: continuum
  version: 0.1.0

#source:
#  fn: 0.1.0
#  #url:

requirements:
  build:
    - python
    - distribute

  run:
    - python

test:
  imports:
    - continuum

about:
  home: https://github.com/tpn/python-libcontinuum
  license: LGPL
```

Note that no source keynames are specified. This requires modifying build.sh to
locate and situate the tarball correctly, along these lines:

```
$ more build.sh
#!/bin/sh

cp -r $RECIPE_DIR/* .
tar -zxvf continuum.tar.gz
$PYTHON setup.py install
```

If you have to hack at this, it's crucial to understand that $RECIPE_DIR (and
other environment variables) are set by conda when the conda build command is
executed, so you should expect different behavior running

```
$ ./build.sh
```

by itself versus when it is invoked by conda build. This means debugging and
hacking any necessary modifications should be done by calling `$ conda build .`
and peppering build.sh with `ls` and `pwd` statements so you can track what
it's doing.

5. Perform the package build with conda
---------------------------------------

If everything is set up right it is, as they say, as easy as

```
$ conda build .
```

6. Place the package in a repository on binstar
-----------------------------------------------

An account on binstar is required; users can request an account at binstar.org.
Once approved and set up, use the binstar command line utility:

```
$ conda install binstar
$ binstar login
$ binstar upload /home/gergely/code/miniconda/conda-bld/linux-32/continuum-0.1.0-py27_0.tar.bz2
```

7. Add the channel and conda install
------------------------------------

```
$ conda config --add channels johngergely
$ conda install continuum
```

It seems to work... but I think it's not right. This package seems to install
but it's kind a hack. The way I got to this point requires an awkward kind of
bootstrap.

Initially all the source files are local, so once I've got setup.py tuned, I
make a tarball. Then I set up the meta.yaml in a new directory without
specifying a source (because it's all local, not in a repository yet) and copy
the tarball there. I hack up the build.sh to unpack the local tarball and build
from that.

This is the thing that's packaged and hosted on binstar at the moment:
https://binstar.org/johngergely/continuum/0.1.0/download/linux-32/continuum-0.1.0-py27_0.tar.bz2
and it seems to work. BUT it doesn't conform to the steps laid out in the docs.
The problem is, how do you specify a url in the meta.yaml for the initial conda
build **before** the package is uploaded yet?

