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

function(add_library_if_exists library_path library_name)
    message(STATUS "Adding library if exists: Path=${library_path}, Name=${library_name}")

    set(library_full_path "${library_path}/${library_name}${LIB_EXT}")
    message(STATUS "Constructed full library path: ${library_full_path}")

    if(EXISTS "${library_full_path}")
        message(STATUS "${library_name}${LIB_EXT} exists at ${library_full_path}")
        list(APPEND CRITERION_LIBRARIES "${library_full_path}")
        message(STATUS "Appended to CRITERION_LIBRARIES: ${library_full_path}")
    else()
        message(STATUS "${library_name}${LIB_EXT} not found at ${library_full_path}, trying to find in directory")

        # Use find command to locate the library
        execute_process(
            COMMAND find "${library_path}" -name "${library_name}${LIB_EXT}"
            OUTPUT_VARIABLE found_library
            OUTPUT_STRIP_TRAILING_WHITESPACE
        )

        message(STATUS "Find command output: ${found_library}")

        # If found, append to CRITERION_LIBRARIES
        if(found_library)
            message(STATUS "Found ${library_name}${LIB_EXT} at: ${found_library}")
            list(APPEND CRITERION_LIBRARIES "${found_library}")
            message(STATUS "Appended to CRITERION_LIBRARIES: ${found_library}")
        else()
            message(WARNING "${library_name}${LIB_EXT} not found in the specified path.")
        endif()
    endif()
endfunction()

add_library_if_exists("${CMAKE_BINARY_DIR}/Criterion/subprojects/libgit2" "libgit2")




include_directories(SYSTEM "${CRITERION_INCLUDE_DIRS}")

# Add a debug message for test sources
message(STATUS "Adding executable for unit_tests with sources: ${TEST_SOURCES}")

add_executable (unit_tests "${TEST_SOURCES}")
# Add a debug message
message(STATUS "Linking unit_tests with libraries: fmem, ${CRITERION_LIBRARIES}")

target_link_libraries (unit_tests fmem "${CRITERION_LIBRARIES}")

add_test (unit_tests unit_tests)
