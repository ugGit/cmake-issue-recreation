
cmake_minimum_required(VERSION 3.22)

set( CMAKE_CUDA_ARCHITECTURES "all" CACHE STRING
   "CUDA architectures to build device code for" )

project(TestNvcppCmake LANGUAGES CUDA)

# add the executable
add_executable(TestNvcppCmake matrix_sums.cu)
