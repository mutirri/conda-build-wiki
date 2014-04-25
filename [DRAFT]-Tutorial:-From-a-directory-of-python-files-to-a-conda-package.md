This episode starts with the most basic scenario. I have a directory containing python code. It does not have dependencies. I want to bundle it and make it available as a conda package.

The basic steps are as follows (for a Linux environment):
1. Format files to be compatible with a setup.py build process.
2. Add and edit setup.py.
3. Create a tarball of the distribution.
4. Create and edit conda build files: meta.yaml, build.sh, bld.bat.
5. Perform the package build with conda.
6. Place the package in a repository on binstar.
7. Add the channel and conda install!

**Using tpn's continuum library of python utilities, here's what this looks like, step by step.**

1. Format files to be compatible with a setup.py build process
--------------------------------------------------------------
Our initial setup is a collection of .py files in a directory:
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

This is what we want to bundle and distribute as a conda package.

We'll create a new directory and assemble everything we want:
    $ mkdir continuum-python
    $ cp -r python-libcontinuum/lib continuum-python/
    $ ls -R continuum-python
    .:
    continuum
    
    ./continuum:
    cli.py               command.pyc  debug.py      invariant.py  path.py        util.pyc
    commandinvariant.py  commands.py  __init__.py   logic.py      pipe_win32.py  _version.py
    command.py           config.py    __init__.pyc  logic.pyc     util.py

2. Add and edit setup.py, and test it
-------------------------------------
Now we need to place a setup.py file in the top-level directory of the package. I took one from a similar distribution and copied it, then edited by hand.
    $ cd continuum-python
    $ cp ../some_other_package/setup.py .

Basic edits involve substituting your package name and details about the author, version, license, etc. Note that setup.py makes use of a file called _version.py in the source directory. This file should be edited or created with this result:

    $ more continuum/_version.py
    __version_info__ = (0, 1, 0)
    __version__ = '.'.join(str(x) for x in __version_info__)

where the tuple __version_info__ contains the correct version for your software.

Verify that the build and install commands function correctly:
    $ python setup.py build
    $ python setup.py install

Make any edits if errors occur.

3. Create a tarball of the distribution
---------------------------------------
You now need to bundle this package and place it in a repository where it can be downloaded and distributed. First, create a tarball of the files you've just assembled:

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


4. Create and edit conda build files: meta.yaml, build.sh, bld.bat
------------------------------------------------------------------
Create a build directory and include in it the conda build files (steal 'em from somewhere).
    $ mkdir continuum-build-trial
    $ cd continuum-build-trial/
    $ cp ../continuum-conda-pkg/meta.yaml .
    $ cp ../continuum-conda-pkg/build.sh .
    $ cp ../continuum-conda-pkg/bld.bat .

Special Case - Building locally
-------------------------------
This can be built from a local tarball. In that case, it needs to also be placed in the build directory...

    $ cp ../cotinuum-python/continuum.tar.gz .

... and the conda build files look like this.

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

Note that no source keynames are specified. This requires modifying build.sh to locate and situate the tarball correctly, along these lines:

    $ more build.sh 
    #!/bin/sh
    
    cp -r $RECIPE_DIR/* .
    tar -zxvf continuum.tar.gz
    $PYTHON setup.py install

If you have to hack at this, it's crucial to understand that $RECIPE_DIR (and other environment variables) are set by conda when the conda build command is executed, so you should expect different behavior running
    $ ./build.sh
by itself versus when it is invoked by conda build. This means debugging and hacking any necessary modifications should be done by calling `$ conda build .` and peppering build.sh with `ls` and `pwd` statements so you can track what it's doing.

5. Perform the package build with conda
---------------------------------------
If everything is set up right it is, as they say, as easy as
    $ conda build .


** DISREGARD CONTENT BEYOND THIS POINT **
6. Place the package in a repository on binstar
-----------------------------------------------
An account on binstar is required; users can request an account at binstar.org. Once approved and set up, use the binstar command line utility:

    $ conda install binstar
    $ binstar login
    $ binstar upload /home/gergely/code/miniconda/conda-bld/linux-32/continuum-0.1.0-py27_0.tar.bz2

7. Add the channel and conda install
------------------------------------
$ conda config --add channels johngergely
$ conda install continuum

It seems to work... but I think it's not right. This package seems to install but it's kind a hack. The way I got to this point requires an awkward kind of bootstrap.

Initially all the source files are local, so once I've got setup.py tuned, I make a tarball. Then I set up the meta.yaml in a new directory without specifying a source (because it's all local, not in a repository yet) and copy the tarball there. I hack up the build.sh to unpack the local tarball and build from that.

This is the thing that's packaged and hosted on binstar at the moment:
https://binstar.org/johngergely/continuum/0.1.0/download/linux-32/continuum-0.1.0-py27_0.tar.bz2

and it seems to work. BUT it doesn't conform to the steps laid out in the docs. The problem is, how do you specify a url in the meta.yaml for the initial conda build **before** the package is uploaded yet?

Not sure what I'm mixing up here.


**DISREGARD THE FOLLOWING**

 1859  ls
 1860  ls python-libcontinuum/
 1861  ls python-libcontinuum/lib/
 1862  ls python-libcontinuum/lib/continuum/
 1863  ll
 1864  tar -crvf continuum-python/
 1865  tar -crvf continuum-python/ continuum.tar
 1866  tar -crvf continuum.tar continuum-python/
 1867  tar -cvf continuum-python/ continuum.tar
 1868  ll
 1869  tar -cvf continuu.tar continuum-python/
 1870  ll
 1871  mv continuu.tar continuum.tar
 1872  rm continuum.tar 
 1873  tar -czvf continuum.tar continuum-python/
 1874  ll
 1875  ls
 1876  cd continuum-build-trial/
 1877  ls
 1878  mv ../continuum.tar .
 1879  conda build . continuum.tar 
 1880  vim meta.yaml 
 1881  conda build . continuum.tar 
 1882  vim meta.yaml 
 1883  conda build . continuum.tar 
 1884  conda build .
 1885  vim build.sh 
 1886  vim meta.yaml 
 1887  conda build.sh .
 1888  conda build .
 1889  ls
 1890  vim build.sh 
 1891  ./build.sh 
 1892  vim build.sh 
 1893  ls
 1894  ls continuum-python/
 1895  rm -rf continuum-python/
 1896  ls
 1897  rm continuum.tar 
 1898  ls
 1899  cd ..
 1900  ls
 1901  ll
 1902  cd continuum-python/
 1903  ls
 1904  tar -czvf continuum.tar ./*
 1905  ll
 1906  mv continuum.tar ../continuum-build-trial/
 1907  cd ../continuum-build-trial/
 1908  l
 1909  ll
 1910  conda build  .
 1911  conda build .
 1912  ll
 1913  tar -zxvf continuum.tar 
 1914  ll
 1915  rm -rf continuum
 1916  rm setup.py 
 1917  rm installer.py 
 1918  ls
 1919  vim build.sh 
 1920  ./build.sh 
 1921  ll
 1922  rm -rf continuum
 1923  rm installer.py 
 1924  rm setup.py 
 1925  ls
 1926  vim meta.yaml 
 1927  conda build .
 1928  ls
 1929  /bin/bash -x -e /media/data/code/sandbox/continuum-build-trial/build.sh
 1930  ls
 1931  /bin/bash -x -e /media/data/code/sandbox/continuum-build-trial/build.sh
 1932  vim build.sh 
 1933  conda install setuptools
 1934  ls
 1935  rm -rf continuum
 1936  rm setup.py 
 1937  rm installer.py 
 1938  ls
 1939  conda build .
 1940  ls
 1941  vim build.sh 
 1942  ./build.sh 
 1943  vim build.sh 
 1944  ./build.sh 
 1945  pwd
 1946  ls
 1947  vim build.sh 
 1948  ./build.sh 
 1949  conda build .
 1950  vim build.sh 
 1951  conda build .
 1952  vim build.sh 
 1953  conda build .
 1954  ls
 1955  vim build.sh 
 1956  conda build .
 1957  vim build.sh 
 1958  conda build .
 1959  conda info
 1960  cd
 1961  which python
 1962  conda info
 1963  python
 1964  conda create -n test_continuum
 1965  conda create -n test_continuum python=2.7
 1966  source activate test_continuum
