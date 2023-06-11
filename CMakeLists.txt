cmake_minimum_required(VERSION 3.12)
project(ChatGLM.cpp VERSION 0.0.1 LANGUAGES CXX)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib CACHE STRING "")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib CACHE STRING "")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin CACHE STRING "")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -Wall")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native")

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif ()

# third-party libraries
set(GGML_NO_ACCELERATE ON CACHE BOOL "chatglm: disable Accelerate framework to reduce memory overhead")
include_directories(third_party/ggml/include)
add_subdirectory(third_party/ggml)

include_directories(third_party/sentencepiece/src)
add_subdirectory(third_party/sentencepiece)

add_library(chatglm STATIC chatglm.cpp)
add_executable(main main.cpp)
target_link_libraries(main chatglm ggml sentencepiece-static)

# GoogleTest
option(CHATGLM_ENABLE_TESTING "chatglm: enable testing" OFF)
if (CHATGLM_ENABLE_TESTING)
    enable_testing()

    # ref: https://github.com/google/googletest/blob/main/googletest/README.md
    include(FetchContent)
    FetchContent_Declare(
      googletest
      # Specify the commit you depend on and update it regularly.
      URL https://github.com/google/googletest/archive/refs/heads/main.zip
    )
    # For Windows: Prevent overriding the parent project's compiler/linker settings
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    FetchContent_MakeAvailable(googletest)

    # Now simply link against gtest or gtest_main as needed. Eg
    add_executable(chatglm_test chatglm_test.cpp)
    target_link_libraries(chatglm_test chatglm gtest_main ggml sentencepiece-static)
    add_test(NAME chatglm_test COMMAND chatglm_test)
endif ()

# lint
set(CPP_SOURCES
    ${PROJECT_SOURCE_DIR}/chatglm.h
    ${PROJECT_SOURCE_DIR}/chatglm.cpp
    ${PROJECT_SOURCE_DIR}/chatglm_test.cpp)
set(PY_SOURCES
    ${PROJECT_SOURCE_DIR}/convert.py)
add_custom_target(lint
    COMMAND clang-format -i ${CPP_SOURCES}
    COMMAND isort ${PY_SOURCES}
    COMMAND black ${PY_SOURCES} --line-length 120)