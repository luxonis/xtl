############################################################################
# Copyright (c) Johan Mabille, Sylvain Corlay and Wolf Vollprecht          #
# Copyright (c) QuantStack                                                 #
#                                                                          #
# Distributed under the terms of the BSD 3-Clause License.                 #
#                                                                          #
# The full license is in the file LICENSE, distributed with this software. #
############################################################################

cmake_minimum_required(VERSION 3.16)
project(xtl)

set(XTL_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Versioning
# ===========

file(STRINGS "${XTL_INCLUDE_DIR}/xtl/xtl_config.hpp" xtl_version_defines
     REGEX "#define XTL_VERSION_(MAJOR|MINOR|PATCH)")
foreach(ver ${xtl_version_defines})
    if(ver MATCHES "#define XTL_VERSION_(MAJOR|MINOR|PATCH) +([^ ]+)$")
        set(XTL_VERSION_${CMAKE_MATCH_1} "${CMAKE_MATCH_2}" CACHE INTERNAL "")
    endif()
endforeach()
set(${PROJECT_NAME}_VERSION
    ${XTL_VERSION_MAJOR}.${XTL_VERSION_MINOR}.${XTL_VERSION_PATCH})
message(STATUS "xtl v${${PROJECT_NAME}_VERSION}")

# Dependencies
# ============

if(NOT TARGET nlohmann_json)
    find_package(nlohmann_json QUIET)
endif()

# Build
# =====

set(XTL_HEADERS
    ${XTL_INCLUDE_DIR}/xtl/xbasic_fixed_string.hpp
    ${XTL_INCLUDE_DIR}/xtl/xbase64.hpp
    ${XTL_INCLUDE_DIR}/xtl/xclosure.hpp
    ${XTL_INCLUDE_DIR}/xtl/xcompare.hpp
    ${XTL_INCLUDE_DIR}/xtl/xcomplex.hpp
    ${XTL_INCLUDE_DIR}/xtl/xcomplex_sequence.hpp
    ${XTL_INCLUDE_DIR}/xtl/xspan.hpp
    ${XTL_INCLUDE_DIR}/xtl/xspan_impl.hpp
    ${XTL_INCLUDE_DIR}/xtl/xdynamic_bitset.hpp
    ${XTL_INCLUDE_DIR}/xtl/xfunctional.hpp
    ${XTL_INCLUDE_DIR}/xtl/xhalf_float.hpp
    ${XTL_INCLUDE_DIR}/xtl/xhalf_float_impl.hpp
    ${XTL_INCLUDE_DIR}/xtl/xhash.hpp
    ${XTL_INCLUDE_DIR}/xtl/xhierarchy_generator.hpp
    ${XTL_INCLUDE_DIR}/xtl/xiterator_base.hpp
    ${XTL_INCLUDE_DIR}/xtl/xjson.hpp
    ${XTL_INCLUDE_DIR}/xtl/xmasked_value_meta.hpp
    ${XTL_INCLUDE_DIR}/xtl/xmasked_value.hpp
    ${XTL_INCLUDE_DIR}/xtl/xmeta_utils.hpp
    ${XTL_INCLUDE_DIR}/xtl/xmultimethods.hpp
    ${XTL_INCLUDE_DIR}/xtl/xoptional_meta.hpp
    ${XTL_INCLUDE_DIR}/xtl/xoptional.hpp
    ${XTL_INCLUDE_DIR}/xtl/xoptional_sequence.hpp
    ${XTL_INCLUDE_DIR}/xtl/xplatform.hpp
    ${XTL_INCLUDE_DIR}/xtl/xproxy_wrapper.hpp
    ${XTL_INCLUDE_DIR}/xtl/xsequence.hpp
    ${XTL_INCLUDE_DIR}/xtl/xsystem.hpp
    ${XTL_INCLUDE_DIR}/xtl/xtl_config.hpp
    ${XTL_INCLUDE_DIR}/xtl/xtype_traits.hpp
    ${XTL_INCLUDE_DIR}/xtl/xvisitor.hpp
)

add_library(xtl INTERFACE)

target_include_directories(xtl INTERFACE
    $<BUILD_INTERFACE:${XTL_INCLUDE_DIR}>
    $<INSTALL_INTERFACE:include>)

set_target_properties(xtl
    PROPERTIES
    PUBLIC_HEADER "${XTL_HEADERS}")

target_compile_features(xtl INTERFACE cxx_std_17)

option(BUILD_TESTS "xtl test suite" OFF)
option(DOWNLOAD_GTEST "build gtest from downloaded sources" OFF)
option(XTL_DISABLE_EXCEPTIONS "Disable C++ exceptions" OFF)

if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(test)
endif()

# Installation
# ============

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

install(TARGETS xtl
        EXPORT ${PROJECT_NAME}-targets
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})

# Makes the project importable from the build directory
export(EXPORT ${PROJECT_NAME}-targets
       FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake")

set(XTL_CMAKECONFIG_INSTALL_DIR "${CMAKE_INSTALL_DATAROOTDIR}/cmake/${PROJECT_NAME}" CACHE
    STRING "install path for xtlConfig.cmake")

configure_package_config_file(${PROJECT_NAME}Config.cmake.in
                              "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
                              INSTALL_DESTINATION ${XTL_CMAKECONFIG_INSTALL_DIR})

# xtl is header-only and does not depend on the architecture.
# Remove CMAKE_SIZEOF_VOID_P from xtlConfigVersion.cmake so that an xtlConfig.cmake
# generated for a 64 bit target can be used for 32 bit targets and vice versa.
set(_XTL_CMAKE_SIZEOF_VOID_P ${CMAKE_SIZEOF_VOID_P})
unset(CMAKE_SIZEOF_VOID_P)
# ARCH_INDEPENDENT feature was introduced in cmake 3.14
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
                                 VERSION ${${PROJECT_NAME}_VERSION}
                                 COMPATIBILITY AnyNewerVersion
                                 ARCH_INDEPENDENT)
set(CMAKE_SIZEOF_VOID_P ${_XTL_CMAKE_SIZEOF_VOID_P})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION ${XTL_CMAKECONFIG_INSTALL_DIR})
install(EXPORT ${PROJECT_NAME}-targets
        FILE ${PROJECT_NAME}Targets.cmake
        DESTINATION ${XTL_CMAKECONFIG_INSTALL_DIR})

configure_file(${PROJECT_NAME}.pc.in
               "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc"
                @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc"
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig/")
