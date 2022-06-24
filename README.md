# cmake-issue-recreation

Clone the repo:

```
git clone git@github.com:ugGit/cmake-issue-recreation.git
```

## Requirements
The issue has been encountered in an environment using the following modules:

* nvhpc/22.5
* nvhpc/22.3
* cmake/3.22.2
* cmake/3.23.2

## What is this about?
This project contains a simple CMakeFile that compiles a CUDA file. 
The goal is to specify nvc++ as compiler for CUDA files instead of nvcc. 
The overall objective is the establishment of similar conditions for a performance analysis between a CUDA based code vs. a std::par implementation. 

Despite a [nvidia forum post from 2020](https://forums.developer.nvidia.com/t/nvc-doesnt-recognise-cu-file-type/145666/4), we assume this is possible since CUDA files can be compiled on the CLI using nvc++.
However, there seems to be a problem when using CMake, where the error message is slightly different depending on the CMake and nvhpc version used.

## Observations
The flag `CMAKE_CUDA_ARCHITECTURES` is required and the value `"52"` has been chosen based on the traccc project. 
It works fine, when specifying nvcc from the nvhpc package as CUDA compiler. 
Thus, we assume it is a valid value in general.
Using `all` as alternative value results in the same error.

### Using nvhpc/22.3 and cmake/3.22.2
```
$ cmake -S . -B build -DCMAKE_CUDA_COMPILER=nvc++
-- The CUDA compiler identification is unknown
-- Detecting CUDA compiler ABI info
-- Detecting CUDA compiler ABI info - failed
-- Check for working CUDA compiler: /opt/nvidia/hpc_sdk/Linux_x86_64/22.3/compilers/bin/nvc++
-- Check for working CUDA compiler: /opt/nvidia/hpc_sdk/Linux_x86_64/22.3/compilers/bin/nvc++ - broken
CMake Error at /bld4/opt/cmake/3.22.2/share/cmake-3.22/Modules/CMakeTestCUDACompiler.cmake:56 (message):
  The CUDA compiler

    "/opt/nvidia/hpc_sdk/Linux_x86_64/22.3/compilers/bin/nvc++"

  is not able to compile a simple test program.

  It fails with the following output:

    Change Dir: /bld6/users/nwachuch/nvcpp-tests/cmake-issue-recreation/build/CMakeFiles/CMakeTmp
    
    Run Build Command(s):/usr/bin/gmake -f Makefile cmTC_890d8/fast && /usr/bin/gmake  -f CMakeFiles/cmTC_890d8.dir/build.make CMakeFiles/cmTC_890d8.dir/build
    gmake[1]: Entering directory `/bld6/users/nwachuch/nvcpp-tests/cmake-issue-recreation/build/CMakeFiles/CMakeTmp'
    Building CUDA object CMakeFiles/cmTC_890d8.dir/main.cu.o
    /opt/nvidia/hpc_sdk/Linux_x86_64/22.3/compilers/bin/nvc++      -c /bld6/users/nwachuch/nvcpp-tests/cmake-issue-recreation/build/CMakeFiles/CMakeTmp/main.cu -o CMakeFiles/cmTC_890d8.dir/main.cu.o
    Linking CUDA executable cmTC_890d8
    /bld4/opt/cmake/3.22.2/bin/cmake -E cmake_link_script CMakeFiles/cmTC_890d8.dir/link.txt --verbose=1
    "" CMakeFiles/cmTC_890d8.dir/main.cu.o -o cmTC_890d8 
    Error running link command: No such file or directory
    gmake[1]: *** [cmTC_890d8] Error 2
    gmake[1]: Leaving directory `/bld6/users/nwachuch/nvcpp-tests/cmake-issue-recreation/build/CMakeFiles/CMakeTmp'
    gmake: *** [cmTC_890d8/fast] Error 2
    
    

  

  CMake will not be able to correctly generate this project.
Call Stack (most recent call first):
  CMakeLists.txt:7 (project)


-- Configuring incomplete, errors occurred!
```

### Using nvhpc/22.5 and cmake/3.22.2
This combination fails with almost the same error as the previous one, but the file executable that is being linked changes from `cmTC_890d8` to `cmTC_63803`
```
    Run Build Command(s):/usr/bin/gmake -f Makefile cmTC_63803/fast && /usr/bin/gmake  -f CMakeFiles/cmTC_63803.dir/build.make CMakeFiles/cmTC_63803.dir/build
    gmake[1]: Entering directory `/bld6/users/nwachuch/nvcpp-tests/cmake-issue-recreation/build/CMakeFiles/CMakeTmp'
    Building CUDA object CMakeFiles/cmTC_63803.dir/main.cu.o
    /opt/nvidia/hpc_sdk/Linux_x86_64/22.5/compilers/bin/nvc++      -c /bld6/users/nwachuch/nvcpp-tests/cmake-issue-recreation/build/CMakeFiles/CMakeTmp/main.cu -o CMakeFiles/cmTC_63803.dir/main.cu.o
    Linking CUDA executable cmTC_63803
    /bld4/opt/cmake/3.22.2/bin/cmake -E cmake_link_script CMakeFiles/cmTC_63803.dir/link.txt --verbose=1
    "" CMakeFiles/cmTC_63803.dir/main.cu.o -o cmTC_63803 
    Error running link command: No such file or directory
    gmake[1]: *** [cmTC_63803] Error 2
    gmake[1]: Leaving directory `/bld6/users/nwachuch/nvcpp-tests/cmake-issue-recreation/build/CMakeFiles/CMakeTmp'
    gmake: *** [cmTC_63803/fast] Error 2
```

### Using (nvhpc/22.3 or nvhpc/22.5) and cmake/3.23.2
```
$ cmake -S . -B build -DCMAKE_CUDA_COMPILER=nvc++
-- The CUDA compiler identification is unknown
CMake Error at /bld4/opt/cmake/3.23.2/share/cmake-3.23/Modules/CMakeDetermineCUDACompiler.cmake:654 (message):
  The CMAKE_CUDA_ARCHITECTURES:

    52

  do not all work with this compiler.  Try:

    

  instead.
Call Stack (most recent call first):
  CMakeLists.txt:7 (project)


-- Configuring incomplete, errors occurred!
```

I think, we all agree that it would be nice to actually get a suggestion from the error as indicated by the void after "... Try:"
