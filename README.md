# python bindings to rai

This repo builds python bindings to
[rai](https://github.com/MarcToussaint/rai), including a Pypi wheel.

## Documentation

Although very incomplete, the best intro to the code is found as part
of the
[robotics-course documentation](https://marctoussaint.github.io/robotics-course/). Jupyter
notebooks that demonstrate the use are found in the
[rai tests](https://github.com/MarcToussaint/rai/tree/master/test/ry)
and in [tutorials/](tutorials/).

## Installation via pip

* The pip package was compiled for python3.6 .. 3.10, and most of the dependencies statically linked. A few are still loaded dynamically, which requires installing on Ubuntu:
```
sudo apt install liblapack3 freeglut3 libglew-dev python3 python3-pip
```
* pip-install robotic and dependencies (numpy, scipy)
```
python3 -m pip install --user robotic numpy scipy
```
* Test:
```
python3 -c 'from robotic import ry; print("ry version:", ry.__version__, ry.compiled());'
python3 -c 'from robotic import ry; ry.test.RndScene()'
```
If the `rai-robotModels` path fails, find rai-robotModels and try something like
```
python3 -c 'from robotic import ry; ry.setRaiPath("/usr/local/rai-robotModels"); ry.test.RndScene()'
```
When rai-robotModels is still messed up, try cloning it completely:
```
cd ~/.local; rm -Rf rai-robotModels;
git clone https://github.com/MarcToussaint/rai-robotModels.git
```
<!--
* You can download other examples and test:
```
wget https://github.com/MarcToussaint/rai-python/raw/master/examples/skeleton-solving-example.py
python3 skeleton-solving-example.py
```
-->

## tested within a ubuntu:latest docker:
```
sudo apt install --yes liblapack3 freeglut3 libglew-dev python3 python3-pip
python3 -m pip install --user robotic numpy scipy
python3 -c 'from robotic import ry; ry.test.RndScene()'
```


## Installation from source

This assumes a standard Ubuntu 20.04 (or 18.04) machine.

* Clone the repo:
```
git clone --recursive https://github.com/MarcToussaint/rai-python.git
cd rai-python
```

* Install Ubuntu packages. The following should do this automatically; if you don't like this, call `make -j1 printUbuntuAll` to see which code components depend on which Ubuntu packages, and install by hand.
```
sudo apt-get update
make -C rai -j1 installUbuntuAll  # calls sudo apt-get install; you can always interrupt
```

* Install python packages, including pybind:
```
echo 'export PATH="${PATH}:$HOME/.local/bin"' >> ~/.bashrc   #add this to your .bashrc, if not done already
sudo apt-get install python3 python3-dev python3-numpy python3-pip python3-distutils
pip3 install --user --upgrade pip
pip3 install --user jupyter nbconvert matplotlib pybind11
```
(If tab-autocomplete for jupyter does not work, try `pip3 install --user jedi==0.17.2` )

* Compile using cmake:
```
ln -s build_utils/CMakeLists-ubuntu.txt CMakeLists.txt
make -C rai cleanAll
make -C rai unityAll
mkdir build
cd build
cmake ..
make -j $(command nproc)
```

* Test a first notebook, then checkout all notebooks in `notebooks/` and `rai/test/ry`
```
jupyter-notebook tutorials/1-basics.ipynb
```

* Other tests
```
cd examples
python3 skeleton-solving-example.py
```

## Building a wheel within a manylinux docker

* Build the docker
```
cd build_utils
./build-docker.sh
```

* Run docker and compile wheels inside
```
./run-docker.sh
## inside docker:
cd local #this mounts rai-python/
make -C rai -j1 unityAll
build_utils/build-wheels.sh
exit
```

* Outside of docker, install locally with pip or push wheels to pypi
```
# e.g.
python3.8 -m pip install --user dist/robotic-*cp38*.whl --force-reinstall
python3.10 -m pip install --user dist/robotic-*cp310*.whl --force-reinstall
# or
twine upload dist/*.whl
```


## Use of the wheel binary in C++

* Get the binary lib by installing the pip package:
```
python3 -m pip install --user robotic
```
* Get the sources by cloning this repo recursively:
```
cd $HOME/git; git clone --recursive https://github.com/MarcToussaint/rai-python.git
```
* Copy things into an include and link folder (like 'make install') CHANGE PYTHON VERSION:
```
mkdir -p $HOME/opt/include/rai $HOME/opt/lib
cp $HOME/.local/lib/python3.6/site-packages/robotic/libry.so -f $HOME/opt/lib/libry.cpython-36m-x86_64-linux-gnu.so
cp $HOME/git/rai-python/rai/rai/* -Rf $HOME/opt/include/rai
cp $HOME/git/rai-python/botop/src/* -Rf $HOME/opt/include/rai
```
* Compile your main
```
gcc script2-IK.cpp -I$HOME/opt/include/rai -L$HOME/opt/lib -lry.cpython-36m-x86_64-linux-gnu -lstdc++ `python3-config --ldflags`
```

