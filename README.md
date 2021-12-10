# PyCBC_on_ARM
Instructions for how to [hopefully] install PyCBC on ARM

## Using conda (recommended)

### Step 1

Run
```
brew install --cask miniconda
```

### Step 2

Initialize conda, **this step is invasive**, and will make changes in your local .bash_XXX files. It's one thing I'm not too happy about with this.

```
conda init "$(basename "${SHELL}")"
```

### Step 3

Create a conda env and activate it

```
conda create -n pycbc_test python=3.9
conda activate pycbc_test
```

## Step 4

Install lalsuite

```
conda install lalsuite
```

(and if you've tried the from source build, breathe a big sigh of relief at the simplicity of this!)

## Step 5

Install pycbc

```
pip install git+https://github.com/gwastro/pycbc.git@master
```

(It seems that conda is avoiding some of the weird OSX packing problems that are encountered if not using conda).


## Step 6

(Optional) Install PyCBC's complete list of optional dependencies. From a checkout of the PyCBC source code do:

```
pip install -r requirements.txt
pip install -r companion.txt)
```



## Using homebrew and venv without conda (not recommended)

It is possible to install without using conda. We do this using a virtualenv here, though that isn't a requirement. A number of dependencies have to be installed from source as they are only available in conda. It also causes issues in lalsuite, as some codes that require lalsuite (pycbc, ligo.skymap, ...) struggle to find lalsuite installed from source when I do this. However, I did manage to create a functional env by doing this.

### Step 1

Create a basic homebrew setup. This will require XCode and it’s command line tools. Install python with
`brew install python`
(https://docs.python-guide.org/starting/install3/osx/ <- See here for some pointers on how to do this if new to homebrew. Do *not* try and mix homebrew and macports!)


### Step2

We’ll be installing lalsuite from source, so need a bunch of dependencies installed, so do:

```
brew install numpy openblas hdf5 pkg-config git-lfs automake gsl fftw swig brew install cmake zlib
export PKG_CONFIG_PATH="/opt/homebrew/opt/zlib/lib/pkgconfig":${PKG_CONFIG_PATH} # For some reason zlib doesn't get picked up automatically without this.
```

### Step3:

Need to make sure OPENBLAS is used for some compilation:

```
export OPENBLAS="$(brew --prefix openblas)"
```

### Step 4

Create, and activate, a blank venv

```
DIRECTORY_OF_YOUR_CHOICE_FOR_THE_VENV=SET_THIS_WISELY
VENV_NAME=ALSO_SET_THIS_WISELY
cd ${DIRECTORY_OF_YOUR_CHOICE_FOR_THE_VENV}
virtualenv ${VENV_NAME} -p /opt/homebrew/bin/python3
source ${DIRECTORY_OF_YOUR_CHOICE_FOR_THE_VENV}/${VENV_NAME}/bin/activate
pip install --upgrade cython setuptools pip wheel
```


### Step 5

Install libframe from source (Needed for lalsuite)

```
mkdir SRC
cd SRC
curl http://software.igwn.org/lscsoft/source/framel-8.40.1.tar.xz > framel-8.40.1.tar.xz
cd framel-8.40.1
IDIR=${DIRECTORY_OF_YOUR_CHOICE_FOR_THE_VENV}/${VENV_NAME}
cmake . --install-prefix=${IDIR}
make install -j
export PKG_CONFIG_PATH=${IDIR}/lib/pkgconfig:${PKG_CONFIG_PATH} # Again, I would expect this to work without setting this, but it doesn't.
```

### Step 6

Install libmetaio from source (Needed for lalsuite) 

```
curl http://software.igwn.org/lscsoft/source/metaio-8.5.1.tar.gz > metaio-8.5.1.tar.gz
cd metaio-8.5.1
./configure --prefix=/Users/iwharry/venvs/test --with-zlib=/opt/homebrew/Cellar/zlib/1.2.11
make install -j
```

### Step 7

Install lalsuite

```
git clone https://git.ligo.org/lscsoft/lalsuite.git
export CFLAGS="-Wno-error" # Framel raises warnings
cd lalsuite
./00boot
./configure --prefix=/Users/iwharry/venvs/test
make install -j
export DYLD_LIBRARY_PATH=/Users/iwharry/venvs/test/lib
```

## Step 8

Install pycbc. Ask Ian for the notes on why this isn’t trivial. Two env variables are needed, and we do need to install from the latest PyCBC dev version. The env variables should be updated to your version of MacOSX.

```
 _PYTHON_HOST_PLATFORM=macosx-12.0-arm64 MACOSX_DEPLOYMENT_TARGET=12.0 pip install git+https://github.com/gwastro/pycbc.git@master
 ```
