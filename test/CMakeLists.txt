# Copyright  (c) 2016-2026 Antony Polukhin
# Copyright  (c) 2025-2026 Fedor Osetrov
# Distributed under the Boost Software License, Version 1.0.
# See accompanying file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt

if(NOT TARGET tests)
  add_custom_target(tests)
endif()

if(MSVC)
    set(BOOST_TYPEINDEX_DETAIL_NO_RTTI "/GR-")
    set(BOOST_TYPEINDEX_DETAIL_RTTI "/GR")
else()
    set(BOOST_TYPEINDEX_DETAIL_NO_RTTI "-fno-rtti")
    set(BOOST_TYPEINDEX_DETAIL_RTTI "-frtti")
endif()

# Making libraries that CANNOT work between rtti-on/rtti-off modules
# add_library(boost_type_index_test_lib_nortti SHARED test_lib.cpp)
# target_compile_options(boost_type_index_test_lib_nortti PRIVATE ${BOOST_TYPEINDEX_DETAIL_NO_RTTI})
# target_link_libraries(boost_type_index_test_lib_nortti PRIVATE Boost::type_index)

# add_library(boost_type_index_test_lib_anonymous_nortti SHARED test_lib_anonymous.cpp)
# target_compile_options(boost_type_index_test_lib_anonymous_nortti PRIVATE ${BOOST_TYPEINDEX_DETAIL_NO_RTTI})
# target_link_libraries(boost_type_index_test_lib_anonymous_nortti PRIVATE Boost::type_index)

add_library(boost_type_index_test_lib_rtti SHARED test_lib.cpp)
target_compile_options(boost_type_index_test_lib_rtti PRIVATE ${BOOST_TYPEINDEX_DETAIL_RTTI})
target_link_libraries(boost_type_index_test_lib_rtti PRIVATE Boost::type_index)

add_library(boost_type_index_test_lib_anonymous_rtti SHARED test_lib_anonymous.cpp)
target_compile_options(boost_type_index_test_lib_anonymous_rtti PRIVATE ${BOOST_TYPEINDEX_DETAIL_RTTI})
target_link_libraries(boost_type_index_test_lib_anonymous_rtti PRIVATE Boost::type_index)

# Making libraries that can work between rtti-on/rtti-off modules
# add_library(boost_type_index_test_lib_nortti_compat SHARED test_lib.cpp)
# target_compile_options(boost_type_index_test_lib_nortti_compat PRIVATE ${BOOST_TYPEINDEX_DETAIL_NO_RTTI})
# target_compile_definitions(boost_type_index_test_lib_nortti_compat PUBLIC BOOST_TYPE_INDEX_FORCE_NO_RTTI_COMPATIBILITY=1)
# target_link_libraries(boost_type_index_test_lib_nortti_compat PRIVATE Boost::type_index)

# add_library(boost_type_index_test_lib_rtti_compat SHARED test_lib.cpp)
# target_compile_options(boost_type_index_test_lib_rtti_compat PRIVATE ${BOOST_TYPEINDEX_DETAIL_RTTI})
# target_compile_definitions(boost_type_index_test_lib_rtti_compat PUBLIC BOOST_TYPE_INDEX_FORCE_NO_RTTI_COMPATIBILITY=1)
# target_link_libraries(boost_type_index_test_lib_rtti_compat PRIVATE Boost::type_index)

function(boost_type_index_test name sources)
    add_executable(${name} "${sources}")
    target_link_libraries(${name} PRIVATE Boost::type_index Boost::core ${ARGN})
    add_test(NAME ${name} COMMAND ${name})
    add_dependencies(tests ${name})
endfunction()

boost_type_index_test(type_index_test type_index_test.cpp)
boost_type_index_test(type_index_constexpr_test type_index_constexpr_test.cpp)
boost_type_index_test(type_index_ctti_print_name_test ctti_print_name.cpp)
# boost_test(TYPE run SOURCES type_index_runtime_cast_test.cpp LINK_LIBRARIES Boost::smart_ptr)
# boost_test(TYPE run SOURCES type_index_test.cpp COMPILE_OPTIONS ${BOOST_TYPEINDEX_DETAIL_NO_RTTI} NAME type_index_test_no_rtti)

boost_type_index_test(type_index_crossmodule_test testing_crossmodule.cpp boost_type_index_test_lib_rtti)
# boost_test(TYPE run SOURCES testing_crossmodule.cpp
#     LINK_LIBRARIES boost_type_index_test_lib_nortti
#     COMPILE_OPTIONS ${BOOST_TYPEINDEX_DETAIL_NO_RTTI}
#     NAME testing_crossmodule_no_rtti
# )
boost_type_index_test(type_index_crossmodule_anonymous_test testing_crossmodule_anonymous.cpp boost_type_index_test_lib_anonymous_rtti)

boost_type_index_test(type_index_compare_ctti_stl_test compare_ctti_stl.cpp)
boost_type_index_test(type_index_track_13621 track_13621.cpp)

boost_type_index_test(type_index_ctti_alignment_test type_index_test_ctti_alignment.cpp)

# # Mixing RTTI on and off
# if(NOT MSVC)  # MSVC sometimes overrides the /GR- and the tests link
#     boost_test(TYPE link-fail SOURCES testing_crossmodule.cpp
#             LINK_LIBRARIES boost_type_index_test_lib_rtti
#             COMPILE_OPTIONS  ${BOOST_TYPEINDEX_DETAIL_NO_RTTI}
#             NAME link_fail_nortti_rtti)
#     boost_test(TYPE link-fail SOURCES testing_crossmodule.cpp
#             LINK_LIBRARIES boost_type_index_test_lib_nortti
#             NAME link_fail_rtti_nortti)

#     boost_test(TYPE run SOURCES testing_crossmodule.cpp
#         LINK_LIBRARIES boost_type_index_test_lib_rtti_compat
#         COMPILE_OPTIONS ${BOOST_TYPEINDEX_DETAIL_NO_RTTI}
#         COMPILE_DEFINITIONS BOOST_TYPE_INDEX_FORCE_NO_RTTI_COMPATIBILITY=1
#         NAME testing_crossmodule_nortti_rtti_compat
#     )
#     boost_test(TYPE run SOURCES testing_crossmodule.cpp
#         LINK_LIBRARIES boost_type_index_test_lib_nortti_compat
#         COMPILE_DEFINITIONS BOOST_TYPE_INDEX_FORCE_NO_RTTI_COMPATIBILITY=1
#         NAME testing_crossmodule_rtti_nortti_compat
#     )
# endif()

file(GLOB EXAMPLE_FILES "../examples/*.cpp")
foreach (testsourcefile ${EXAMPLE_FILES})
    get_filename_component(testname ${testsourcefile} NAME_WE)
    boost_type_index_test(type_index_${testname}_example ${testsourcefile} Boost::unordered)
    target_include_directories(type_index_${testname}_example PRIVATE ../examples)

    # if(NOT testname STREQUAL "table_of_names")
    #     boost_test(TYPE run SOURCES ${testsourcefile} NAME ${testname}_no_rtti COMPILE_OPTIONS  ${BOOST_TYPEINDEX_DETAIL_NO_RTTI} INCLUDE_DIRECTORIES ../examples LINK_LIBRARIES Boost::unordered)
    # endif()
endforeach()
