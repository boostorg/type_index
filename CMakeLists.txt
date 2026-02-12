# -----------------------------------------------------------------------------
# Boost.type_index CMake
# Handles: no modules, modules, modules + import std;
# -----------------------------------------------------------------------------
# Copyright 2020, 2021 Peter Dimov
# Distributed under the Boost Software License, Version 1.0.
# https://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 3.21...4.2)

project(boost_type_index VERSION "${BOOST_SUPERPROJECT_VERSION}" LANGUAGES CXX)

if(PROJECT_IS_TOP_LEVEL)
    find_package(Boost 1.90.0 CONFIG)
endif()

# -----------------------------------------------------------------------------
# User option: enable C++ modules
# -----------------------------------------------------------------------------
option(BOOST_USE_MODULES "Build Boost using C++ modules" OFF)

# -----------------------------------------------------------------------------
# Determine target type and sources
# -----------------------------------------------------------------------------
if(BOOST_USE_MODULES)

    # Ensure CMAKE_CXX_STANDARD is set for module detection
    if(NOT DEFINED CMAKE_CXX_STANDARD)
        set(CMAKE_CXX_STANDARD 20)
    endif()

    add_library(boost_type_index)

    target_sources(boost_type_index
        PUBLIC
            FILE_SET modules_public
            TYPE CXX_MODULES
            FILES modules/boost_type_index.cppm
    )

    # Require C++20 for modules
    target_compile_features(boost_type_index PUBLIC cxx_std_${CMAKE_CXX_STANDARD})
    # Define macro indicating modules usage
    target_compile_definitions(boost_type_index PUBLIC BOOST_USE_MODULES)

    # Check if import std; is available for the current standard
    if(${CMAKE_CXX_STANDARD} IN_LIST CMAKE_CXX_COMPILER_IMPORT_STD)
        target_compile_definitions(boost_type_index PUBLIC BOOST_TYPE_INDEX_USE_STD_MODULE)
        set_propery(TARGET boost_type_index CXX_MODULE_STD ON)
        message(STATUS "Boost.type_index: Using `import std;`")
    else()
        message(WARNING "Boost.type_index: `import std;` is not available for C++${CMAKE_CXX_STANDARD}")
    endif()

    set(__scope PUBLIC)

else()

    # Modules disabled -> INTERFACE library
    add_library(boost_type_index INTERFACE)

    # Verify interface headers only at top level
    if(PROJECT_IS_TOP_LEVEL)
        set(CMAKE_VERIFY_INTERFACE_HEADER_SETS ON)
    endif()

    set(__scope INTERFACE)

endif()

# -----------------------------------------------------------------------------
# Include headers
# -----------------------------------------------------------------------------
if(CMAKE_VERSION VERSION_LESS 3.23)
    target_include_directories(boost_type_index ${__scope} include)
else()
    target_sources(
        boost_type_index
        ${__scope}
        FILE_SET headers_public
        TYPE HEADERS
        BASE_DIRS include
        FILES
            include/boost/type_index.hpp
            include/boost/type_index/runtime_cast.hpp
            include/boost/type_index/stl_type_index.hpp
            include/boost/type_index/detail/compile_time_type_info.hpp
            include/boost/type_index/detail/stl_register_class.hpp
            include/boost/type_index/detail/config.hpp
            include/boost/type_index/detail/ctti_register_class.hpp
            include/boost/type_index/ctti_type_index.hpp
            include/boost/type_index/runtime_cast/std_shared_ptr_cast.hpp
            include/boost/type_index/runtime_cast/detail/runtime_cast_impl.hpp
            include/boost/type_index/runtime_cast/register_runtime_class.hpp
            include/boost/type_index/runtime_cast/pointer_cast.hpp
            include/boost/type_index/runtime_cast/reference_cast.hpp
            include/boost/type_index/runtime_cast/boost_shared_ptr_cast.hpp
            include/boost/type_index/type_index_facade.hpp
    )

    # If modules are disabled, require C++17 for headers
    if(NOT BOOST_USE_MODULES)
        target_compile_features(boost_type_index ${__scope} cxx_std_17)
    endif()
endif()

# -----------------------------------------------------------------------------
# Link dependencies
# -----------------------------------------------------------------------------
if(PROJECT_IS_TOP_LEVEL)
    target_link_libraries(boost_any ${__scope} Boost::headers)
else()
    target_link_libraries(boost_type_index
        ${__scope}
        Boost::config
        Boost::container_hash
        Boost::throw_exception
    )
endif()

# Alias for convenient import
add_library(Boost::type_index ALIAS boost_type_index)

# -----------------------------------------------------------------------------
# Testing
# -----------------------------------------------------------------------------
if(BUILD_TESTING)
    add_subdirectory(test)
endif()
