include(FetchContent)

cmake_minimum_required(VERSION 3.6)

project(linuxdeploy C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake/Modules/")

find_program(PATCHELF patchelf REQUIRED)

# support for ccache
# call CMake with -DUSE_CCACHE=ON to make use of it
set(USE_CCACHE ON CACHE BOOL "")
if(USE_CCACHE)
    find_program(CCACHE ccache)
    if(CCACHE)
        message(STATUS "Using ccache")
        set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ${CCACHE})
        set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ${CCACHE})
    else()
        message(WARNING "USE_CCACHE set, but could not find ccache")
    endif()
endif()

set(ENABLE_COVERAGE OFF CACHE BOOL "Enable coverage measurements")
if(ENABLE_COVERAGE)
    include(CodeCoverage)
    message(WARNING "Enabling code coverage measurements -> disables optimizations and embeds debug information!")

    append_coverage_compiler_flags()

    set(COVERAGE_GCOVR_EXCLUDES ${PROJECT_SOURCE_DIR}/lib ${PROJECT_BINARY_DIR})

    include(ProcessorCount)
    ProcessorCount(processor_count)

    set(command cmake --build . --target all -- -j ${processor_count} && ctest -V)

    setup_target_for_coverage_gcovr_html(NAME coverage EXECUTABLE "${command}")
    setup_target_for_coverage_gcovr_xml(NAME coverage_xml EXECUTABLE "${command}")
    setup_target_for_coverage_gcovr_text(NAME coverage_text EXECUTABLE "${command}")
endif()

include(CTest)

if(BUILD_TESTING)
    # including this before including lib/ makes sure that the top level project's gtest is used everywhere
    include(${PROJECT_SOURCE_DIR}/lib/cmake-scripts/include-or-build-gtest.cmake)
endif()

add_subdirectory(lib)

add_subdirectory(src)

if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()
