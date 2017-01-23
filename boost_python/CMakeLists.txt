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
