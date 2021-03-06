cmake_minimum_required(VERSION 2.6)
enable_testing()
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)
project(marray)
include_directories(include include/marray)
file(GLOB headers include/marray/*)

if(CMAKE_COMPILER_IS_GNUCXX)
    # add_definitions(-Wall -Werror)
    add_definitions(-Wall)
endif()

if(MSVC)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE -D_HDF5USEDLL_)
endif()

find_package(HDF5 QUIET)
if(HDF5_FOUND)
    message(STATUS "HDF5 found.")
    add_definitions(-DHasHDF5)
    include_directories(${HDF5_INCLUDE_DIR})
else()
    message("HDF5 not found.")
endif()

find_package(Doxygen QUIET)
if(DOXYGEN_FOUND)
    message(STATUS "Doxygen found.")
else()
    message("doxygen not found.")
endif()

set(WITH_CPP0X CACHE BOOL "Compile with C++0x features.")
if(WITH_CPP0X)
    if(CMAKE_COMPILER_IS_GNUCXX)
        message(STATUS "C++0x support enabled.")
        add_definitions(-std=gnu++0x)
        # add_definitions(-std=c++0x)
        add_definitions(-fno-deduce-init-list)
        add_definitions(-DHAVE_CPP0X_INITIALIZER_LISTS)
        add_definitions(-DHAVE_CPP0X_VARIADIC_TEMPLATES)
        # add_definitions(-DHAVE_CPP0X_TEMPLATE_ALIASES)
        # not supported in GNU G++ 4.4
    else()
        message("C++0x support disabled. Add flags for your compiler in CMakeLists.txt to compile with C++0x.")
    endif()
else()
    message(STATUS "C++0x support disabled.")
endif()

add_executable(tutorial-marray src/tutorial/tutorial.cxx ${headers})

add_executable(test-marray src/unittest/marray.cxx ${headers})
add_test(test-marray test-marray)

if(HDF5_FOUND)
    add_executable(test-marray-hdf5 src/unittest/marray_hdf5.cxx ${headers})
    	target_link_libraries(test-marray-hdf5 ${HDF5_CORE_LIBRARY})
    add_test(test-marray-hdf5 test-marray-hdf5)
else()
    message("Not building test-marray-hdf5.")
endif()

if(DOXYGEN_FOUND)
    configure_file("${marray_SOURCE_DIR}/Doxyfile.in" "${marray_BINARY_DIR}/Doxyfile" @ONLY IMMEDIATE)
    add_custom_target(doc-marray ALL COMMAND ${DOXYGEN} "${marray_BINARY_DIR}/Doxyfile")
endif()

install(DIRECTORY "include/marray" DESTINATION include PATTERN ".hxx")

