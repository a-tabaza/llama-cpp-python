cmake_minimum_required(VERSION 3.21)

project(llama_cpp)

option(LLAMA_BUILD "Build llama.cpp shared library and install alongside python package" ON)
option(LLAVA_BUILD "Build llava shared library and install alongside python package" ON)

function(llama_cpp_python_install_target target)
    if(NOT TARGET ${target})
        return()
    endif()

    install(
        TARGETS ${target}
        LIBRARY DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/llama_cpp/lib
        RUNTIME DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/llama_cpp/lib
        ARCHIVE DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/llama_cpp/lib
        FRAMEWORK DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/llama_cpp/lib
        RESOURCE DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/llama_cpp/lib
    )
    install(
        TARGETS ${target}
        LIBRARY DESTINATION ${SKBUILD_PLATLIB_DIR}/llama_cpp/lib
        RUNTIME DESTINATION ${SKBUILD_PLATLIB_DIR}/llama_cpp/lib
        ARCHIVE DESTINATION ${SKBUILD_PLATLIB_DIR}/llama_cpp/lib
        FRAMEWORK DESTINATION ${SKBUILD_PLATLIB_DIR}/llama_cpp/lib
        RESOURCE DESTINATION ${SKBUILD_PLATLIB_DIR}/llama_cpp/lib
    )
    set_target_properties(${target} PROPERTIES
        INSTALL_RPATH "$ORIGIN"
        BUILD_WITH_INSTALL_RPATH TRUE
    )
    if(UNIX)
        if(APPLE)
            set_target_properties(${target} PROPERTIES
                INSTALL_RPATH "@loader_path"
                BUILD_WITH_INSTALL_RPATH TRUE
            )
        else()
            set_target_properties(${target} PROPERTIES
                INSTALL_RPATH "$ORIGIN"
                BUILD_WITH_INSTALL_RPATH TRUE
            )
        endif()
    endif()
endfunction()

if (LLAMA_BUILD)
    set(BUILD_SHARED_LIBS "On")

    set(CMAKE_SKIP_BUILD_RPATH FALSE)

    # When building, don't use the install RPATH already
    # (but later on when installing)
    set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
 
    # Add the automatically determined parts of the RPATH
    # which point to directories outside the build tree to the install RPATH
    set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
    set(CMAKE_SKIP_RPATH FALSE)

    # Enable building of the common library
    set(LLAMA_BUILD_COMMON ON CACHE BOOL "Build llama.cpp common library" FORCE)

    add_subdirectory(vendor/llama.cpp.qwen2.5vl)
    llama_cpp_python_install_target(llama)
    llama_cpp_python_install_target(ggml)
    llama_cpp_python_install_target(ggml-base)
    llama_cpp_python_install_target(ggml-cpu)


    if (LLAVA_BUILD)
        if (LLAMA_CUBLAS OR LLAMA_CUDA)
            add_compile_definitions(GGML_USE_CUBLAS)
            add_compile_definitions(GGML_USE_CUDA)
        endif()

        if (LLAMA_METAL)
            add_compile_definitions(GGML_USE_METAL)
        endif()

        # Building llava
        add_subdirectory(vendor/llama.cpp.qwen2.5vl/examples/llava)
        set_target_properties(llava_shared PROPERTIES OUTPUT_NAME "llava")

        llama_cpp_python_install_target(llava_shared)

        # Fix for llava build: Add include directory for llama.h
        # Move these commands after the add_subdirectory call
        target_include_directories(llava PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/vendor/llama.cpp.qwen2.5vl/include)
        target_include_directories(llava PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/vendor/llama.cpp.qwen2.5vl/include)

        if (BUILD_SHARED_LIBS)
            target_include_directories(llava_shared PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/vendor/llama.cpp.qwen2.5vl/include)
            target_include_directories(llava_shared PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/vendor/llama.cpp.qwen2.5vl/ggml/include)
        endif()

        target_include_directories(llama-llava-cli PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/vendor/llama.cpp.qwen2.5vl/include)
        target_include_directories(llama-minicpmv-cli PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/vendor/llama.cpp.qwen2.5vl/include)
    endif()
endif()
