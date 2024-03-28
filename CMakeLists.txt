cmake_minimum_required(VERSION 3.0)
project(skeleton_libretro)

# Set the default platform
if(NOT platform)
    if(UNIX)
        set(platform "unix")
    elseif(WIN32)
        set(platform "win")
    elseif(APPLE)
        set(platform "osx")
    endif()
endif()

# Set system platform
if(UNIX)
    set(system_platform "unix")
elseif(APPLE)
    set(system_platform "osx")
elseif(WIN32)
    set(system_platform "win")
endif()

# Set architecture flags
if(NOT ARCHFLAGS)
    if(archs STREQUAL "ppc")
        set(ARCHFLAGS "-arch ppc -arch ppc64")
    else()
        set(ARCHFLAGS "-arch i386 -arch x86_64")
    endif()
endif()

# Set the target extension based on static linking
if(STATIC_LINKING)
    set(EXT "a")
else()
    set(EXT "so")
endif()

# Set compiler flags
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -D__LIBRETRO__")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -D__LIBRETRO__")

# Set additional flags based on platform
if(platform STREQUAL "osx")
    if(NOT NOUNIVERSAL)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${ARCHFLAGS}")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ARCHFLAGS}")
    endif()
endif()

# Set additional flags based on debug mode
if(DEBUG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -g -DDEBUG")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -g -DDEBUG")
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
endif()

# Set the target name
set(LIBRARY_TARGET_NAME "skeleton")
set(LINKER_SCRIPT "${CMAKE_SOURCE_DIR}/link.T")

# Set target specific properties based on platform
if(platform STREQUAL "unix")
    add_compile_options(-Wall --no-undefined)
elseif(platform STREQUAL "win")
    add_compile_options(-static-libgcc -static-libstdc++ -s -W4 --no-undefined)
endif()

# Include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Add source files
file(GLOB SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/*.c" "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp")

# Build the target
add_library(${LIBRARY_TARGET_NAME} SHARED ${SOURCES})

#Linker Script
set(CMAKE_EXE_LINKER_FLAGS "-T ${LINKER_SCRIPT}")
set_target_properties(${LIBRARY_TARGET_NAME} PROPERTIES PREFIX "")
set_source_files_properties(${CMAKE_SOURCE_DIR}libretro.c PROPERTIES OBJECT_DEPENDS ${LINKER_SCRIPT})

# Link libraries
target_link_libraries(${LIBRARY_TARGET_NAME} ${LIBM})

# Installation
install(TARGETS ${LIBRARY_TARGET_NAME} DESTINATION lib)