﻿include(CheckCXXCompilerFlag)

set(SOURCES test.cpp)

if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    set(OPTIONS /W4 /WX)
    check_cxx_compiler_flag(/permissive HAS_PERMISSIVE_FLAG)
    if(HAS_PERMISSIVE_FLAG)
        set(OPTIONS ${OPTIONS} /permissive-)
    endif()

    check_cxx_compiler_flag(/std:c++17 HAS_CPP17_FLAG)
    check_cxx_compiler_flag(/std:c++20 HAS_CPP20_FLAG)
    check_cxx_compiler_flag(/std:c++latest HAS_CPPLATEST_FLAG)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    set(CMAKE_VERBOSE_MAKEFILE ON)
    set(OPTIONS -Wall -Wextra -pedantic-errors -Werror)

    check_cxx_compiler_flag(-std=c++17 HAS_CPP17_FLAG)
    check_cxx_compiler_flag(-std=c++20 HAS_CPP20_FLAG)
endif()

function(make_test target std)
    add_executable(${target} ${SOURCES})
    target_compile_options(${target} PRIVATE ${OPTIONS})
    target_include_directories(${target} PRIVATE 3rdparty/Catch2)
    target_link_libraries(${target} PRIVATE ${CMAKE_PROJECT_NAME})
    if(std)
        if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
            target_compile_options(${target} PRIVATE /std:${std})
        else()
            target_compile_options(${target} PRIVATE -std=${std})
        endif()
    endif()
    add_test(NAME ${target} COMMAND ${target})
endfunction()

if(HAS_CPP17_FLAG)
    make_test(${CMAKE_PROJECT_NAME}-cpp17.t c++17)
endif()

if(HAS_CPPLATEST_FLAG)
    make_test(${CMAKE_PROJECT_NAME}-cpplatest.t c++latest)
endif()
