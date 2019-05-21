# Tutorial: Multi-Lingual Case study with VersionClimber: Phenomenal

## Overview
In this tutorial, you will learn how to use VersionClimber when packages comes from multiple languages (e.g.C++ and Python). In our case, this repository aims at testing Phenomenal, a package built to process biological image data and extract 3D structural information to infer genotype to phenotype relationship.

A Python library, named Phenomenal, has been developed and depends on a set of widely used libraries for scientific visualisation and image processing such as VTK and OpenCV. While these libraries have Python interfaces, they are implemented in C++ for efficiency. However, they are know to be difficult to install due to binary conflicts between lots of dependencies.

Moreover, in this example, we rely on some code that is not available on any public conda channel. For each configuration, we will then rebuild the local packages. Then, to test the status of each configuraion, rather than executing a notebook, we will run the full test suite.

Phenomenal depends on Python packages such as Python, NumPy, SciPy, MatPlotLib, NetworkX, Pandas, Scikit-Learn and Scikit-Image.
It depends also on utilities such as nose, coverage or openalea.deploy.
As noted above, it also depends on more complex scientific packages such as OpenCV and VTK.

## Before you start

You should already have installed [Miniconda](https://conda.io) or
[Anaconda](https://docs.continuum.io/anaconda/install).

Create an new conda environment:

```bash
conda create -n vclimber -c versionclimber versionclimber -y
```
and then activate this environment:

```bash
conda activate vclimber
```

Now, to run VersionClimber, just type ::

```bash
vclimb -a
```

To obtain the documentation, just type:

```bash
vclimb -h
```

If you want to show the list of available versions for each package, run:

```bash
vclimb -va
```

## Classical layout of a project

To reproduce an execution by using VersionClimber, you will create a directory containing a file and a directory.
This current directory contains:
- **config.yaml:** the VersionClimber configuration file
- **recipes: ** a directory that contains one recipe per dependency package.


## Definition of a simple configuration file

VersionClimber uses the declarative configuration file to indicate which packages have to be tested and how.

In this section you are going to define a configuration file [config.yaml](config.yaml) that uses 16 conda packages with different levels of complexity.
Some are pure Python, while others are C/C++ packages with potential binary compatibility problems.


It is divided into four sections, namely **pre**, **packages**, **run**, and **post**:
- **pre** (Optional): a set of commands to execute before running VersionClimber. Here, we install the conda build package to be able to build new conda packages for each configuration.
- **packages:** list the different packages, their location (e.g. git repository), how to build them and which git commit or tags will be considered (in hierarchy, as explained below).
- **run:** indicate how to test the different packages together to know if one combination is valid. Typically (as in this example), this will be the name of a driver file.
- **post** (Optional): a command run at the end of the process, when VersionClimber has found the up-to-date package combination. Here, we export the conda environment into a file to be able to recreate it on another computer.

### Packages

The *packages* section list the different packages that will be tested by the run command:
- **name** is the name of the package
- **vcs** define which type of version control system the package use (i.e. git or svn).
- **url** is the address where the package will be cloned or checkout
- **cmd** is the command to build the package
- **conda** is an optional argument to indicate if the package is managed by conda (`True`) or pip (`False`)
- **recipe** is the local path where the recipe is defined
- **hierarchy** is the strategy use to select the different versions of the package from the *vcs*.
If *hierarchy* is `major`, `minor`, or `patch`, the versions of the tags will be selected for that indentation level and higher. Otherwise, (`commit`) all the commits of the origin or master branch will be tested by VersionClimber. In this example, because minor packages are of the  form x.y, VersionClimber will take the most recent patch associated with each x.y. So, if a package is identified as 5.4.3 and there is no higher patch number among the patches that begin with 5.4, then VersionClimber will select 5.4.3.


### Run command in config.yaml

This is the script (usually) after run: in that file. In our example,
`python test_function.py`

## Adaptation of conda recipes for Version Climber

Conda recipes of major packages can be found on different repository, such as [conda-recipes](https://github.com/conda/conda-recipes).
In this example, we depends on two packages: [boost](http://www.boost.org) and [protobuf](https://developers.google.com/protocol-buffers).

VersionClimber needs modified conda recipes to build all the packages together locally from different versions of git source code.
All the modified recipes are located in the recipes directory.

### openalea.deploy recipe

First, we get the [openalea.deploy recipe](https://github.com/openalea/deploy) from its git repository.

The *meta.yaml* file is copied into a template file (named [*meta.yaml.tpl*](./recipes/openalea.deploy/meta.yaml.tpl)) and is modified as follow:
```yaml

...
```

The **package** and the **source** sections are modified:
- **package**: the **version** variable must be equal to **"$version"**. VersionClimber will replaced the **"$version"** variable by the git tag or the git commit value.
- **source**: the **url** value is replaced by a local location where VersionClimber will clone the boost package (i.e. **../../.vclimb/openalea.deploy**).



### openalea.phenomenal recipe

The [Phenomenal recipe](https://github.com/openalea/phenomenal) is retrieved from its github repository.

We simplify it by removing the header and the tests that can be done in the driver file. The [meta.yaml.tpl](./recipes/protobuf/meta.yaml.tpl) looks like :

```yaml
# REMOVE THESE LINES
# {% set name = "protobuf" %}
# {% set version = "3.2.0" %}
# {% set sha256 = "2a25c2b71c707c5552ec9afdfb22532a93a339e1ca5d38f163fe4107af08c54c" %}

package:
  name: "protobuf"
  # REPLACE
  version: {{ version }}
  # BY VERSIONCLIMBER template flag
  version : "$version"

source:
  # REPLACE
  fn: {{ name }}-{{ version }}.tar.gz
  url: https://github.com/google/protobuf/archive/v{{ version }}/{{ name }}-v{{ version }}.tar.gz
  sha256: {{ sha256 }}
  # BY new VERSIONCLIMBER protobuf location
  path : ../../.vclimb/protobuf

build:
  features:

...
# REMOVE ALL THE TEST  SECTION

#test:
#  commands:
#    - protoc --help
#    - test -f ${PREFIX}/lib/libprotobuf.a                           # [unix]
#    - test -f ${PREFIX}/lib/libprotobuf.dylib                       # [osx]
#    - test -f ${PREFIX}/lib/libprotobuf.so                          # [linux]
#    - if not exist %PREFIX%\\Library\\lib\\libprotobuf.lib exit 1   # [win]
#  imports:
#    - google
#    - google.protobuf
#    - google.protobuf.internal
#    - google.protobuf.pyext

about:
    home: https://developers.google.com/protocol-buffers/
    license: New BSD License
    summary: Protocol Buffers - Google's data interchange format

...
```



## Invocation of VersionClimber

vclimb -a -- will fetch the packages from git, retrieve all the versions, install each configuration (set of package-version pairs) suggested by the Version Climber software, then invoke the run part of the config.yaml on that installed configuration. The output is configuration that works sorted based on the priorities in config.yaml
