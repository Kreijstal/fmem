set (TEST_SOURCES tests.c)
if (WIN32)
    set(LIB_EXT ".dll.a")
else()
    set(LIB_EXT ".a")
endif()
  
# Append additional libraries to CRITERION_LIBRARIES if they exist
if(EXISTS "${CMAKE_BINARY_DIR}/Criterion/subprojects/nanomsg/libnanomsg${LIB_EXT}")
    list(APPEND CRITERION_LIBRARIES "${CMAKE_BINARY_DIR}/Criterion/subprojects/nanomsg/libnanomsg${LIB_EXT}")
endif()

if(EXISTS "${CMAKE_BINARY_DIR}/Criterion/subprojects/nanopb/libprotobuf_nanopb_static${LIB_EXT}")
    list(APPEND CRITERION_LIBRARIES "${CMAKE_BINARY_DIR}/Criterion/subprojects/nanopb/libprotobuf_nanopb_static${LIB_EXT}")
endif()

if(EXISTS "${CMAKE_BINARY_DIR}/Criterion/subprojects/boxfort/src/libboxfort${LIB_EXT}")
    list(APPEND CRITERION_LIBRARIES "${CMAKE_BINARY_DIR}/Criterion/subprojects/boxfort/src/libboxfort${LIB_EXT}")
endif()

function(add_library_if_exists library_name direct_library_path lib_ext base_search_directory)
    # Check if the library exists at the direct path
    set(library_full_path "${direct_library_path}/${library_name}${lib_ext}")
    if(EXISTS "${library_full_path}")
        # If it exists, append to the specified libraries list
        list(APPEND ${CRITERION_LIBRARIES} "${library_full_path}")
    else()
        # If not found, execute the find command to locate the library in the base search directory
        execute_process(
            COMMAND find "${base_search_directory}" -name "${library_name}${lib_ext}"
            OUTPUT_VARIABLE found_library
            OUTPUT_STRIP_TRAILING_WHITESPACE
        )

        # If found, append to the specified libraries list
        if(found_library)
            message(STATUS "Found ${library_name}${lib_ext} at: ${found_library}")
            list(APPEND ${libraries_list} "${found_library}")
        else()
            message(WARNING "${library_name}${lib_ext} not found in ${base_search_directory}.")
        endif()
    endif()
endfunction()
# Example usage
add_library_if_exists("libgit2" "${CMAKE_BINARY_DIR}/Criterion/subprojects/libgit2" "${LIB_EXT}" "${CMAKE_BINARY_DIR}/Criterion")




include_directories(SYSTEM "${CRITERION_INCLUDE_DIRS}")

# Add a debug message for test sources
message(STATUS "Adding executable for unit_tests with sources: ${TEST_SOURCES}")

add_executable (unit_tests "${TEST_SOURCES}")
# Add a debug message
message(STATUS "Linking unit_tests with libraries: fmem, ${CRITERION_LIBRARIES}")

target_link_libraries (unit_tests fmem "${CRITERION_LIBRARIES}")

add_test (unit_tests unit_tests)
