project(xreal)

cmake_minimum_required(VERSION 2.8)

set(CMAKE_MODULE_PATH ${CMAKE_HOME_DIRECTORY}/cmake)

# You can override this var with commandline option -DXREAL_CLIENT_NAME=name
if (NOT XREAL_CLIENT_NAME)
    set(XREAL_CLIENT_NAME XreaL)
endif()

# Windows specific settings
if (${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    set(CMAKE_INCLUDE_PATH
        "./libs/libogg"
        "./libs/libvorbis"
        "./libs/libtheora"
        "./libs/libcurl/include"
        "./libs/zlib"
        "./libs/libfreetype"
        "./libs/libpng"
        "./libs/libjpeg"
    )

    add_definitions(-D_CRT_SECURE_NO_WARNINGS -DCURL_STATICLIB)

    set(CMAKE_LIBRARY_PATH "sys/win/x86/lib/debug")
endif()

# OS X specific settings
if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	set(CMAKE_OSX_ARCHITECTURES "$(ARCHS_STANDARD_64_BIT)")    # Compile only 64-bit version
	set(CMAKE_OSX_SYSROOT "macosx")                            # Compile with latest available OS X sdk
	set(CMAKE_XCODE_ATTRIBUTE_MACOSX_DEPLOYMENT_TARGET "10.8") # Use 10.8 as deployment target
    set(CMAKE_FRAMEWORK_PATH "sys/mac/Frameworks")
    set(CMAKE_MACOSX_RPATH 1)
    set(JPEG_NAMES libjpeg) # libjpeg.framework should be renamed to jpeg.framework to remove this hack
endif()

find_package(Ogg REQUIRED)
find_package(Vorbis REQUIRED)
find_package(Theora REQUIRED)
find_package(CURL REQUIRED)
find_package(ZLIB REQUIRED)
find_package(Freetype REQUIRED)
find_package(JPEG REQUIRED)
find_package(PNG REQUIRED)
find_package(SDL REQUIRED)

macro(find_windows_release_libs libs)
    foreach (lib_name ${${libs}})
        string(REPLACE "lib/debug" "lib/release" release_lib_name ${lib_name})
        set(${libs}_DEBUG ${${libs}_DEBUG} debug ${lib_name})
        set(${libs}_RELEASE ${${libs}_RELEASE} optimized ${release_lib_name})
    endforeach()

    set(${libs} ${${libs}_RELEASE} ${${libs}_DEBUG})
endmacro()

if (${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MTd")

    set(CMAKE_EXECUTABLE_SUFFIX "_x86")
    set(CMAKE_SHARED_LIBRARY_SUFFIX "_x86")

    find_windows_release_libs(ZLIB_LIBRARY)
    find_windows_release_libs(JPEG_LIBRARIES)
    find_windows_release_libs(PNG_LIBRARIES)
    find_windows_release_libs(FREETYPE_LIBRARIES)
    find_windows_release_libs(OGG_LIBRARY)
    find_windows_release_libs(VORBIS_LIBRARIES)
    find_windows_release_libs(THEORA_LIBRARY)
    find_windows_release_libs(CURL_LIBRARY)
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-unused-function -fvisibility=hidden -Winvalid-pch")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -O0 -D_DEBUG")
    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -fno-strict-aliasing -ffast-math -funroll-loops -msse2")

    set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG}")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}")
endif()

macro(qf_set_output_dir name dir)
    foreach (OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES})
        string(TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIGUPPERCASE)
        set_target_properties(${name} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIGUPPERCASE} ${CMAKE_HOME_DIRECTORY}/build/${OUTPUTCONFIG}/${dir})
        set_target_properties(${name} PROPERTIES LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIGUPPERCASE} ${CMAKE_HOME_DIRECTORY}/build/${OUTPUTCONFIG}/${dir})
        set_target_properties(${name} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIGUPPERCASE} ${CMAKE_HOME_DIRECTORY}/build/${OUTPUTCONFIG}/${dir})
    endforeach()

    set_target_properties(${name} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_HOME_DIRECTORY}/build/${dir})
    set_target_properties(${name} PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_HOME_DIRECTORY}/build/${dir})
    set_target_properties(${name} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_HOME_DIRECTORY}/build/${dir})
endmacro()

macro(qf_add_library name)
    add_library(${name} ${ARGN})

    if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
        set_target_properties(${name} PROPERTIES PREFIX "")
        set_target_properties(${name} PROPERTIES SUFFIX "_x86_64${CMAKE_SHARED_LIBRARY_SUFFIX}")
    endif()

    if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
        set_target_properties(${name} PROPERTIES PREFIX "")
        set_target_properties(${name} PROPERTIES SUFFIX "_mac${CMAKE_SHARED_LIBRARY_SUFFIX}")
    endif()
endmacro()

macro(qf_add_executable name)
    add_executable(${name} ${ARGN})

    if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
        set_target_properties(${name} PROPERTIES PREFIX "")
        set_target_properties(${name} PROPERTIES SUFFIX ".x86_64${CMAKE_EXECUTABLE_SUFFIX}")
    endif()
endmacro()

add_subdirectory(cgame)
add_subdirectory(game)
add_subdirectory(q3_ui)
add_subdirectory(client)
add_subdirectory(server)
add_subdirectory(tools)
