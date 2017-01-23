# Using boost::python to build python extension in C/C++

In this document, we use `boost v1.63.0` and `Microsoft Visual Studio 2015`  or ```mingw-w64 gcc```on Windows platform.

## Preparing the Boost Libraries for Building Python Extension in C/C++

### 1. Get Boost

Get boost library ```v1.63.0``` from [Boost C++ Libraries - /boost/1.63.0 at SourceForge.net](https://sourceforge.net/projects/boost/files/boost/1.63.0/) and unpack it, assume the unpacked root directory is ```<boos_root>```.
### 2. Prepare the Boost.Build System Build From Source

Open a command prompt and change your current directory to the ```<boost_root>``` directory. Then, type the following commands:

```
C:\boost_1_63_0>boostrap
```

The `boostrap` command prepares the ```Boost.Build``` system for use. At that command is completed, you should see a `b2`  executables is built at the ```<boost_root>``` directory.

Boost.Build is the name of the complete build system. The executable that runs it is `b2` or `bjam`. 

### 3. Create a Jamfile To Configure Compiler and Python Installation

A file called `user-config.jam` in your home directory is used to configure your tools. In Windows, your home directory can be found by typing:    

```
ECHO %HOMEDRIVE%%HOMEPATH%
```

into a command prompt window. Your file should at least have the rules for your compiler and your python installation. A specific example of this on Windows would be:    

```
#  MSVC configuration, 14.0 means Visual Studio 2015
using msvc : 14.0 ;

#  Python configuration
using python : 3.5 : C:/python35 ;
```

### 4. Build From Source

### 4.1 List Allowed Options

Before you start to build the libraries, you can use following commands to list the allowed options:

```
C:\boost_1_63_0>b2 --help
Boost.Build 2015.07-git

Project-specific help:
  Project has jamfile at Jamroot

Usage:
  b2 [options] [properties] [install|stage]
  Builds and installs Boost.

Targets and Related Options:
  install                 Install headers and compiled library files to the
  =======                 configured locations (below).
  --prefix=<PREFIX>       Install architecture independent files here.
                          Default; C:\Boost on Win32
                          Default; /usr/local on Unix. Linux, etc.
...
Other Options:
  --build-type=<type>     Build the specified pre-defined set of variations of
                          the libraries. Note, that which variants get built
                          depends on what each library supports.

                              -- minimal -- (default) Builds a minimal set of
                              variants. On Windows, these are static
                              multithreaded libraries in debug and release
                              modes, using shared runtime. On Linux, these are
                              static and shared multithreaded libraries in
                              release mode.

                              -- complete -- Build all possible variations.
```

From the list options of ```build-type```, the default ```build-type``` is ```minimal```, its behavior depends on the platform you are using to build boost library:

1.  On Windows, these are ```static multithreaded libraries in debug and release modes```, using ```shared runtime```.
2.  On Linux, these are ```static and shared multithreaded libraries``` in ```release``` mode.

#### 4.2 Build Libraries

#### 4.2.1 Using VS2015

Type following command to invoke ```Boost.Build``` to build the separately-compiled Boost libraries using VS2015.

```
C:\boost_1_63_0>b2
```

The building process takes several minutes. After the build is completed, you can see the libraries are stored at ```<boost_root>\stage\lib```.  From that directory, you see following `boost::python` libraries:

```
libboost_python3-vc140-mt-1_63.lib
libboost_python3-vc140-mt-gd-1_63.lib
libboost_python-vc140-mt-1_63.lib
libboost_python-vc140-mt-gd-1_63.lib
```

You can refer here for the details of [boost library naming](http://www.boost.org/doc/libs/1_63_0/more/getting_started/windows.html#library-naming).

#### 4.2.2. Using mingw-w64 gcc 6.2.0

Type following command to invoke `Boost.Build` to build the separately-compiled Boost libraries using ```mingw-w64``` gcc v6.2.0.

```
c:\boost_1_63_0>b2 toolset=gcc define=_hypot=hypot
```

When we use mingw to compile ```boost::python``` library, we need to define CXX flags ```_hypot=hypto```, refer following links for the the details:

1. [c++ - "Error: '::hypot' has not been declared" in cmath while trying to embed Python - Stack Overflow](http://stackoverflow.com/questions/28683358/error-hypot-has-not-been-declared-in-cmath-while-trying-to-embed-python).
2. [error: '::hypot' has not been declared when compiling with MingGW64 · Issue #4926 ](https://github.com/Theano/Theano/issues/4926).

The building process takes several minutes. After the build is completed, you can see the libraries are stored at ```<boost_root>\stage\lib```.  From that directory, you see following `boost::python` libraries:

```
libboost_python-mgw62-mt-1_63.a
libboost_python-mgw62-mt-d-1_63.a
libboost_python3-mgw62-mt-1_63.a
libboost_python3-mgw62-mt-d-1_63.a
```



## Build Tutorial Example of Python Extension in C/C++

I will describe using CMake to build the tutorial example in boost library distribution instead of using `b2`, because I can not successfully using `Boost.Build`to build that example.

### 1. Prepare a CMakeLists.txt for the Tutorial Example

Change the working directory to `<boost_root>\libs\python\example\tutorial`, create a ```CMakeLists.txt``` with following content in that folder:

```
cmake_minimum_required( VERSION 2.8 )

project(hello_ext)

# 1. find python library and include directory.
find_package(PythonInterp 3)
find_package(PythonLibs 3 REQUIRED)
include_directories(${PYTHON_INCLUDE_DIRS} )

# 2. find boost library.
set(Boost_USE_STATIC_BOOST ON)
add_definitions(-DBOOST_ALL_NO_LIB) # may be not required for mingw-w64.
find_package(Boost COMPONENTS python REQUIRED )
include_directories(${Boost_INCLUDE_DIR})

if (MINGW)
    # solve hypot not defined issue.
    #
    # D:/lang/mingw-w64/i686-6.2.0-posix-dwarf-rt_v5-rev1/mingw32/lib/gcc/i686-w64-mingw32/6.2.0/include/c++/cmath:1133:11: error: '::hypot' has not been declared
    # using ::hypot;
    #
    # see https://github.com/Theano/Theano/issues/4926
    #     http://stackoverflow.com/questions/28683358/error-hypot-has-not-been-declared-in-cmath-while-trying-to-embed-python
    add_definitions(-D_hypot=hypot)

    # solve link error - undefined reference to `_imp___ZN5boost6python7objects15function_objectERKNS1_11py_functionE'
    # 
    # see http://stackoverflow.com/questions/14090683/compile-some-code-with-boost-python-by-mingw-in-win7-64bit
    add_definitions(-DBOOST_PYTHON_STATIC_LIB)
endif(MINGW)

# 3. build python extension (library).
add_library(hello_ext SHARED hello.cpp )
target_link_libraries(hello_ext ${Boost_LIBRARIES} ${PYTHON_LIBRARIES})

# 4. don't prepend wrapper library name with lib, and set the file extension as .pyd.
set_target_properties(hello_ext PROPERTIES PREFIX "" SUFFIX ".pyd" )
```



This is the descriptions of setting variables of ```Find Boost``` module:

```
Boost_USE_MULTITHREADED  - Set to OFF to use the non-multithreaded
                           libraries ('mt' tag).  Default is ON.
Boost_USE_STATIC_LIBS    - Set to ON to force the use of the static
                           libraries.  Default is OFF.
Boost_USE_STATIC_RUNTIME - Set to ON or OFF to specify whether to use
                           libraries linked statically to the C++ runtime
                           ('s' tag).  Default is platform dependent.
```



When we use Visual Studio to build the extension, we need to set ```BOOST_ALL_NO_LIB``` options, otherwise we will encounter link error. This the description from [Boost.Config - User settable options](http://www.boost.org/doc/libs/1_63_0/libs/config/doc/html/index.html#boost_config.configuring_boost_for_your_platform.user_settable_options):

```
Macro: BOOST_ALL_NO_LIB

Description:
Tells the config system not to automatically select which libraries to link against. Normally if a compiler supports #pragma lib, then the correct library build variant will be automatically selected and linked against, simply by the act of including one of that library's headers. This macro turns that feature off. 
```

Refer these links for the details:

1. [cmake - How do you add boost libraries in CMakeLists.txt - Stack Overflow](http://stackoverflow.com/questions/6646405/how-do-you-add-boost-libraries-in-cmakelists-txt).
2. [c++ - Linking boost library with Boost_USE_STATIC_LIB OFF on Windows - Stack Overflow](http://stackoverflow.com/questions/28887680/linking-boost-library-with-boost-use-static-lib-off-on-windows).

### 2. Add Boost Path Environment Variables

The [FindBoost](https://cmake.org/cmake/help/v3.7/module/FindBoost.html?highlight=findboost) module reads hints about search boost library locations from below variables:

```
BOOST_ROOT             - Preferred installation prefix
 (or BOOSTROOT)
BOOST_INCLUDEDIR       - Preferred include directory e.g. <prefix>/include
BOOST_LIBRARYDIR       - Preferred library directory e.g. <prefix>/lib
Boost_NO_SYSTEM_PATHS  - Set to ON to disable searching in locations not
                         specified by these hint variables. Default is OFF.
Boost_ADDITIONAL_VERSIONS
                       - List of Boost versions not known to this module
                         (Boost install locations may contain the version)
```

In order to allow ```Find Boost``` module, we add two environment variables:

```
BOOST_ROOT=<boost_root>
BOOST_LIBRARYDIR=<boost_root>\stage\lib
```

You need to replace ```<boost_root>``` with the root directory of your boost library source. 

### 3. Build the Python Extension

#### 3.1 Using VS2015

* From start menu - ```Visual Studio 2015```, click ```Developer Command Prompt for VS2015```.

* Change the working directory to the directory of tutorial example - *```<boost_root>\libs\python\example\tutorial>```.

* Create a directory ```build``` for out of source build, and change the working directory to it.

* Type ```cmake ..``` to generate VS2015 solution file - ```hello_ext.sln```

* Type ```msbuild hello_ext.sln``` to build the python extension. To clean the build, type ```msbuild hello_ext.sln /target:clean``` to clean the existing build - refer [How to: Clean a Build](https://msdn.microsoft.com/en-us/library/ms171480.aspx).

* The built extension ```hello_ext.pyd```should be placed at the ```build\Debug``` folder.

* Copy the ```build\Debug\hello_ext.pyd``` to the folder of ```hello.py```,  run ```python hello.py```, you should see this:

  ```
  C:\boost_1_63_0\libs\python\example\tutorial>python.exe hello.py
  hello, world
  ```

#### 3.2 Using mingw-w64 gcc 6.2.0

* Open a command prompt.

- Change the working directory to the directory of tutorial example - *```<boost_root>\libs\python\example\tutorial>```.
- Create a directory ```build``` for out of source build, and change the working directory to it.
- Type ```cmake -G "MinGW Makefiles" ..``` to generate makefile.
- Type ```make``` to build the python extension. 
- The built extension ```hello_ext.pyd```should be placed at the ```build\Debug``` folder.
- Copy the ```build\Debug\hello_ext.py``` to the folder of ```hello.py```,  run ```python hello.py```, you should see this:
  ```
  C:\boost_1_63_0\libs\python\example\tutorial>python.exe hello.py
  hello, world
  ```


