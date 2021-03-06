# Demonstrate linking of
# * C++ calling Fortran
# * Fortran calling C++
# * Fortran calling C
#
# https://stackoverflow.com/tags/fortran-iso-c-binding/info
#
# this also works between ABI compatible compilers
# e.g. Clang and GCC.
#
# This also works for MacOS with Apple's Clang and Homebrew GCC,
# by being sure you have in ~/.zshrc like the following:
# (check directory / versions on your Mac)
#
# export LIBRARY_PATH=$LIBRARY_PATH:/Library/Developer/CommandLineTools/SDKs/MacOSX11.1.sdk/usr/lib
#
# export CPLUS_INCLUDE_PATH=/Library/Developer/CommandLineTools/SDKs/MacOSX11.1.sdk/usr/include
# export CXXFLAGS=-I$CPLUS_INCLUDE_PATH
# export CFLAGS=$CXXFLAGS
#
# in general, CMake >= 3.14 have better link resolution than CMake 3.13.
cmake_minimum_required(VERSION 3.14...3.21)

project(Fortran_C_CXX_interface
LANGUAGES C Fortran CXX
VERSION 1.1.1)

include(CTest)

include (cmake/compilers.cmake)

add_library(c_lib call_c.c)
target_compile_features(c_lib PRIVATE c_std_99)

add_executable(f_call_c f_call_c.f90 $<TARGET_OBJECTS:c_lib>)
add_test(NAME FortranCallC COMMAND f_call_c)

# -- C++ calling Fortran
add_library(call_f OBJECT c_call_f.f90)

add_executable(cxx_call_f cxx_call_f.cxx $<TARGET_OBJECTS:call_f>)
target_compile_features(cxx_call_f PRIVATE cxx_std_11)
add_test(NAME CXXCallFortran COMMAND cxx_call_f)

# -- Fortran calling C++
add_library(cxx_lib OBJECT call_cxx.cxx)
target_compile_features(cxx_lib PRIVATE cxx_std_11)

add_executable(f_call_cxx f_call_c.f90 $<TARGET_OBJECTS:cxx_lib>)
# LINKER_LANGUAGE option is necessary for ifort at least
set_target_properties(f_call_cxx PROPERTIES LINKER_LANGUAGE Fortran)
add_test(NAME FortranCallC++ COMMAND f_call_cxx)
