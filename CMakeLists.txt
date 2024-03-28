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
set(TARGET_NAME "skeleton")

# Set target specific properties based on platform
if(platform STREQUAL "unix")
    set(TARGET "${TARGET_NAME}_libretro.${EXT}")
    set(SHARED "-shared -Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/link.T -Wl,--no-undefined")
elseif(platform STREQUAL "osx")
    set(TARGET "${TARGET_NAME}_libretro.dylib")
    set(SHARED "-dynamiclib")
elseif(platform STREQUAL "win")
    set(TARGET "${TARGET_NAME}_libretro.dll")
    set(SHARED "-shared -static-libgcc -static-libstdc++ -s -Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/link.T -Wl,--no-undefined")
endif()

# Include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Add source files
file(GLOB SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/*.c" "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp")

# Build the target
add_library(${TARGET_NAME} SHARED ${SOURCES})

# Link libraries
target_link_libraries(${TARGET_NAME} ${LIBM})

# Set target properties
set_target_properties(${TARGET_NAME} PROPERTIES OUTPUT_NAME ${TARGET})

# Installation
install(TARGETS ${TARGET_NAME} DESTINATION lib)