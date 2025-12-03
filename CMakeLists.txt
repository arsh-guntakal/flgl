cmake_minimum_required(VERSION 3.10)
project(FLGL)

set(CMAKE_CXX_STANDARD 20)

find_package(OpenGL REQUIRED)

add_subdirectory(lib/glfw)

add_library(GLAD lib/glad/src/glad.c)
target_include_directories(GLAD PUBLIC lib/glad/include)

include_directories(inc lib/glfw/include lib/glad/include ${OPENGL_INCLUDE_DIR})

file(GLOB_RECURSE FLGL_SOURCES "src/*.cpp")

if(MSVC)
    # /W4       - Enable high warning level (closest to -Wall -Wextra)
    # /permissive- - Enforce standards conformance (closest to -pedantic)
    # /we4715   - Error: Not all control paths return a value (return-type)
    # /we4189   - Error: Local variable is initialized but not referenced (unused-variable)
    # /we4101   - Error: Local variable is unreferenced (unused-variable)
	# /wd4201   - Suppress "nameless struct/union" warning (Required for GLM) actually crazy that this isn't std..
    add_compile_options(/W4 /permissive- /we4715 /we4189 /we4101 /wd4201)
else()
    # Your existing GCC/Clang flags
    add_compile_options(
        -Wall -Wextra -Wpedantic
        -Wunused-variable -Werror=unused-variable
        -Wcast-qual
        -Werror=return-type
    )

    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        add_compile_options(
            -Wc99-extensions -Werror=c99-extensions 
            -Wvla-extension -Werror=vla-extension
        )
    endif()
endif()

add_library(${PROJECT_NAME} ${FLGL_SOURCES})
target_compile_definitions(${PROJECT_NAME} PUBLIC GLFW_INCLUDE_NONE) # hopefull fixes missing include on windows
target_link_libraries(${PROJECT_NAME} PRIVATE ${OPENGL_gl_LIBRARY})
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/lib/glfw/include)

target_link_libraries(${PROJECT_NAME} PUBLIC GLAD glfw ${OPENGL_LIBRARIES})
target_include_directories(${PROJECT_NAME} PUBLIC ${OPENGL_INCLUDE_DIR})
