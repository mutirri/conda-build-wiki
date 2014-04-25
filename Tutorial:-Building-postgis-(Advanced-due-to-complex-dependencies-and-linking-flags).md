Let's set out to build postgis. This package is installable with conda, so we are not flying blind. But we'll start out naively and try to learn some things along the way.

In fact, to make this interesting, let's build postgis 2.1.2 since 2.1.1 has already been done. We'll see how good of an idea that is.

I found a [tarball](http://download.osgeo.org/postgis/source/postgis-2.1.2.tar.gz
) with some internet searching. Let's just start at zero and see how far we get. I'll hack a meta.yaml file like so:

    package:
      name: postgis
      version: 2.1.2

    source:
      fn: postgis-2.1.2.tar.gz
      url: http://download.osgeo.org/postgis/source/postgis-2.1.2.tar.gz

    build:
      number: 0

    about:
      home: http://postgis.net
      license: GPL2

and start with the trivial build.sh:

    #!/bin/sh

    ./configure
    make
    make install

I will make sure my Anaconda distribution is updated first.
    $ cd postgis_2.1.2
    $ ls
    build.sh  meta.yaml
    $ source activate anaconda
    $ conda update anaconda
    $ conda info
    Current conda install:

                 platform : linux-32
            conda version : 3.4.2
           python version : 2.7.6.final.0
         root environment : /home/gergely/code/miniconda  (writable)
      default environment : /home/gergely/code/miniconda/envs/anaconda
         envs directories : /home/gergely/code/miniconda/envs
            package cache : /home/gergely/code/miniconda/pkgs
             channel URLs : http://repo.continuum.io/pkgs/free/linux-32/
                            http://repo.continuum.io/pkgs/pro/linux-32/
              config file : None
        is foreign system : False

Ok now

    $ conda build .

It rolled along until I got this error:
    configure: error: could not find pg_config within the current path. You may need to try re-running configure with a --with-pg_config parameter.
    Command failed: /bin/bash -x -e /media/data/code/sandbox/postgis_2.1.2/build.sh

I'm being asked to specify a path to the utility pg_config. I do `$ which pg_config` and find that it's not installed. The philosophy of conda packaging is that you bundle what you need, so this utility has to be included in the package. I do `$conda search pg_config` and turn up nothing. Therefore I have to assume this package also needs to be built.

Some searching indicates pg_config is distributed with postgresql, so let me check that out. A search on binstar.org for conda packages with the name `postgresql` yields some results. I'll try to install from one of the binstar channels. First the channel must be added with:

    $ conda config --add channels https://conda.binstar.org/trent

which I can verify by inspecting

    $ conda info
    Current conda install:
    
                 platform : linux-32
            conda version : 3.4.2
           python version : 2.7.6.final.0
         root environment : /home/gergely/code/miniconda  (writable)
      default environment : /home/gergely/code/miniconda/envs/anaconda
         envs directories : /home/gergely/code/miniconda/envs
            package cache : /home/gergely/code/miniconda/pkgs
             channel URLs : https://conda.binstar.org/trent/linux-32/
                            http://repo.continuum.io/pkgs/free/linux-32/
                            http://repo.continuum.io/pkgs/pro/linux-32/
              config file : /home/gergely/.condarc
        is foreign system : False

Now:

    $ conda install -c https://conda.binstar.org/trent postgresql
    Fetching package metadata: ....
    Error: No packages found matching: postgresql

Huh. This is a little confounding given what I read [here](https://binstar.org/trent/postgresql).



