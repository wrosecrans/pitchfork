cmake_minimum_required(VERSION 3.12)

list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

project(pitchfork
    VERSION 0.0.1
    LANGUAGES C CXX
    DESCRIPTION "Libraries and programs for working with Pitchfork Layout projects"
    )

if(NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
endif()

option(BUILD_SPEC "Build the Pitchfork document" ON)

include(pmm.cmake)
pmm(CONAN)
conan_set_find_paths()

# Find-package the Boost that Conan gave us
find_package(Boost 1.68.0 REQUIRED)

# Other deps not controlled by Conan
find_package(Filesystem REQUIRED)
if(BUILD_SPEC)
    find_package(Bikeshed)
endif()
find_package(Yarn)
include(CMakeRC)

if(NOT MSVC)
    add_compile_options(-Wall -Wextra)
    if(PROJECT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
        add_compile_options(-Werror)
    endif()
else()
    add_compile_options(/permissive-)
endif()

add_subdirectory(data)

set(PF_URL "file://${PROJECT_SOURCE_DIR}/extras/pf-cmake" CACHE INTERNAL "")
include(Pitchfork)
file(GLOB_RECURSE pf_cmake_files CONFIGURE_DEPENDS extras/pf-cmake/*)
set_property(DIRECTORY APPEND PROPERTY CMAKE_CONFIGURE_DEPENDS ${pf_cmake_files})


pf_auto(
    OUTPUT_NAME pf
    ALIAS pf::pitchfork
    LINK
        CXX::Filesystem
        Boost::boost
        CONAN_PKG::spdlog
    PRIVATE_LINK
        kainjow::mustache
        pf::templates
    EXE_LINK
        taywee::args
    )

include(CPack)
