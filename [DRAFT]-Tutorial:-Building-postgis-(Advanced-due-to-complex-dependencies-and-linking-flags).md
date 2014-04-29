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

For an individual doing this for their own purposes (and maintaining their own computing environments), the most efficient solution may be to install whatever pre-existing binaries are available. For example, I am able to find a binary of PostgreSQL for my Ubuntu OS and do

`$sudo apt-get install postgresql`

BUT if your goal is to bundle a built package and distribute it worry-free to your friends and other users, this is just punting on the real solution. If there aren't reliable conda packages available on binstar to satisfy the needed dependencies, you need to build those as well.

Build postgresql Package
------------------------
    $ mkdir postgresql-9.3.4
    $ ls
    build.sh  meta.yaml
    $ more build.sh

    #!/bin/bash
    
    ./configure --prefix=$PREFIX
    
    make
    make install

    $ more meta.yaml
package:
  name: postgresql
  version: 9.3.4

source:
  fn: postgresql-9.3.4.tar.gz
  url: http://ftp.postgresql.org/pub/source/v9.3.4/postgresql-9.3.4.tar.gz

build:
  number: 0

about:
  home: http://www.postgresql.org
  license: GPL2

Build command history - to be formatted later
=============================================
Build postgresql
----------------
2034  mkdir postgresql-9.3.4
 2035  cd postgresql-9.3.4/
 2036  pd
 2037  pd ../../src/
 2038  ls
 2039  ls src_postgresql-9.3.4/
 2040  ll src_postgresql-9.3.4/
 2041  popd
 2042  mv ../../src/src_postgresql-9.3.4/meta.yaml .
 2043  mv ../../src/src_postgresql-9.3.4/build.sh .
 2044  ls
 2045  conda build .
 2046  conda update conda
 2047  vim meta.yaml 
 2048  conda build .
 2049  vim meta.yaml 
 2050  ls
 2051  ls ..
 2052  ls ../..
 2053  conda build .
 2054  binstar upload /home/gergely/code/miniconda/conda-bld/linux-32/postgresql-9.3.4-0.tar.bz2
 2055  conda info
 2056  conda search postgresql
 2057  conda install postgresql

Return to build of postgis
--------------------------
 2058  cd ..
 2059  ls
 2060  mkdir postgis-2.1.2
 2061  ls ..
 2062  ls ../postgis_2.1.2/
 2063  cp -r ../postgis_2.1.2/ .
 2064  ls
 2065  ls postgis_2.1.2/
 2066  ls postgis-2.1.2/
 2067  rm -rf postgis-2.1.2/
 2068  ls
 2069  cd postgis_2.1.2/
 2070  ls
 2071  vim build.sh 
 2072  vim meta.yaml 
 2073  conda build .

ERROR - libxml2
-----
 2074  conda install libxml2
 2075  conda build .

ERROR - geos-config

Install geos
------------

 2024  mkdir geos-3.4.2
 2025  cd geos-3.4.2/
 2026  cp ../postgresql-9.3.4/meta.yaml .
 2027  cp ../postgresql-9.3.4/build.sh .
 2028  vim build.sh 
 2029  vim meta.yaml 
 2030  conda build .
 2031  binstar upload /home/gergely/code/miniconda/conda-bld/linux-32/geos-3.4.2-0.tar.bz2

ERROR
-----
configure: error: could not find libgeos_c - you may need to specify the directory of a geos-config file using --with-geosconfig

Need to update meta.yaml with requirements as they become clear. conda will help resolve these paths as dependencies are specified.

ERROR
-----
configure: error: could not find proj_api.h - you may need to specify the directory of a PROJ.4 installation using --with-projdir

Build proj
----------

