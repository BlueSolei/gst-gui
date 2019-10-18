cmake_minimum_required(VERSION 3.2)
project(gst)

if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Build type is default defined as Release. change it by define -DCMAKE_BUILD_TYPE=Debug as param to cmake")
    set(CMAKE_BUILD_TYPE Release)
endif()

# Download conan.cmake, download taged release - v0.14
if(NOT EXISTS "${CMAKE_SOURCE_DIR}/env/conan.cmake")
   message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
   file(DOWNLOAD "https://github.com/conan-io/cmake-conan/raw/v0.14/conan.cmake"
                 "${CMAKE_SOURCE_DIR}/env/conan.cmake")
endif()
include(env/conan.cmake)

conan_cmake_run(CONANFILE conanfile.txt
                BASIC_SETUP
                BUILD missing)

set(CMAKE_CXX_STANDARD 17)

set(CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})
set(CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})

# CONFIG option is important so that CMake doesnt search for modules into the default modules directory
find_package(imgui CONFIG)
find_package(glfw CONFIG)
find_package(glew CONFIG)

set(GUISrc
		src/Main.cpp
		src/OpenGLShader.cpp
		src/FileManager.cpp
		src/OpenGLShader.h
		src/FileManager.h
		src/bindings/imgui_impl_glfw.cpp
		src/bindings/imgui_impl_opengl3.cpp
		src/bindings/imgui_impl_glfw.h
		src/bindings/imgui_impl_opengl3.h
		src/assets/simple-shader.vs
		src/assets/simple-shader.fs
		)

set(AppSrc
		)

add_executable(gst
			${GUISrc}
			${AppSrc}
		)

add_custom_command(TARGET gst
		POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_SOURCE_DIR}/src/assets/simple-shader.vs ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
		COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_SOURCE_DIR}/src/assets/simple-shader.fs ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
		)

target_compile_definitions(gst PUBLIC IMGUI_IMPL_OPENGL_LOADER_GLEW)

target_link_libraries(gst imgui::imgui glfw::glfw glew::glew)

# Tests
set(TEST_FILES
  	src/gst.cpp
  	src/gst.h
	tests/gstTest.cpp
	tests/Main.cpp
	)
set(TEST_NAME ${CMAKE_PROJECT_NAME}Test)

add_executable(${TEST_NAME} ${TEST_FILES})
target_link_libraries(${TEST_NAME} ${CONAN_LIBS})


install(DIRECTORY ${CMAKE_SOURCE_DIR}/env/
DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
FILES_MATCHING
PATTERN "*.*"
PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)
#FILES ${CMAKE_SOURCE_DIR}/env DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
