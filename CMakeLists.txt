project(HelloOpenGL)
cmake_minimum_required(VERSION 2.8)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(ucm)

ucm_add_dirs(src TO SRC_LIST)

add_executable(${PROJECT_NAME} ${SRC_LIST})

include_directories(${PROJECT_SOURCE_DIR}/include)

################################################################################
if (${CMAKE_SYSTEM_NAME} STREQUAL Linux)
    OPTION(USE_System_GLFW "Use GLFW externally (recommended for platforms which support pkgconfig like Linux" ON)
    OPTION(USE_System_GLM "Use GLM externally (recommended for platforms like Linux" ON)
	find_library(dl_lib dl)
	set(externalLibs ${externalLibs} ${dl_lib})
else (${CMAKE_SYSTEM_NAME} STREQUAL Linux)
    OPTION(USE_System_GLFW "Use GLFW externally (recommended for platforms which support pkgconfig like Linux" OFF)
    OPTION(USE_System_GLM "Use GLM externally (recommended for platforms like Linux" OFF)
endif (${CMAKE_SYSTEM_NAME} STREQUAL Linux)    
################################################################################

# Adding header files recursively. aux_source_directory doesn't do that very well
#file(GLOB_RECURSE headerFilesList "*.h" "*.hpp")
#add_custom_target(headerFiles SOURCES ${headerFilesList})

################################################################################
# Enabling C++11. Add these two lines after add_executable. Notice you must 
# Enable C++11 for every target in all your CMake scripts.
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 11)
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD_REQUIRED ON)

################################################################################

#find_package(OpenGL REQUIRED)

if (${USE_System_GLFW})
    find_package(PkgConfig REQUIRED)
    pkg_search_module(GLFW REQUIRED glfw3)
    set(externalLibs ${externalLibs} ${GLFW_LIBRARIES})
else (${USE_System_GLFW})
	set(GLFW_INSTALL_LOCATION "${PROJECT_BINARY_DIR}/external/glfw")
	include(ExternalProject)
	ExternalProject_Add(external_glfw
		URL https://github.com/glfw/glfw/releases/download/3.2.1/glfw-3.2.1.zip
		CMAKE_ARGS "${CMAKE_ARGS};"
			"-DCMAKE_INSTALL_PREFIX=${GLFW_INSTALL_LOCATION};"
			"-DGLFW_BUILD_DOCS=OFF;"
			"-DGLFW_BUILD_TESTS=OFF;"
			"-DGLFW_BUILD_EXAMPLES=OFF;"
		)
	add_library(glfw STATIC IMPORTED)
	add_dependencies(glfw external_glfw)
	add_dependencies(${PROJECT_NAME} glfw)
	include_directories(${GLFW_INSTALL_LOCATION}/include)
	find_library(glfw_lib glfw3 ${GLFW_INSTALL_LOCATION}/lib)
	set(externalLibs ${externalLibs} ${glfw_lib})
endif (${USE_System_GLFW})

if (NOT ${USE_System_GLM})
#    include_directories(${PROJECT_SOURCE_DIR}/external/glm)
endif (NOT ${USE_System_GLM})

target_link_libraries(${PROJECT_NAME} ${externalLibs})

file(COPY ${PROJECT_SOURCE_DIR}/shaders/ DESTINATION ${PROJECT_BINARY_DIR}/shaders
     FILES_MATCHING PATTERN "*")

