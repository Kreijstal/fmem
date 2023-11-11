set (TEST_SOURCES tests.c)
if (WIN32)
    set(LIB_EXT ".dll.a")
else()
    set(LIB_EXT ".a")
endif()
  
function(append_library LIB_PATH LIB_NAME)
    if(EXISTS "${CMAKE_BINARY_DIR}/Criterion/subprojects/${LIB_PATH}/lib${LIB_NAME}${LIB_EXT}")
        list(APPEND CRITERION_LIBRARIES "${CMAKE_BINARY_DIR}/Criterion/subprojects/${LIB_PATH}/lib${LIB_NAME}${LIB_EXT}")
    else()
        execute_process(
            COMMAND find "${CMAKE_BINARY_DIR}/Criterion" -name "lib${LIB_NAME}${LIB_EXT}"
            OUTPUT_VARIABLE FOUND_LIB
            OUTPUT_STRIP_TRAILING_WHITESPACE
        )
        if(FOUND_LIB)
            message(STATUS "Found lib${LIB_NAME}${LIB_EXT} at: ${FOUND_LIB}")
            list(APPEND CRITERION_LIBRARIES "${FOUND_LIB}")
        else()
            message(WARNING "lib${LIB_NAME}${LIB_EXT} not found in Criterion subprojects.")
        endif()
    endif()
endfunction()

# Usage examples
append_library("nanomsg" "nanomsg")
append_library("nanopb" "protobuf_nanopb_static")
append_library("boxfort/src" "boxfort")
append_library("libgit2" "libgit2")


include_directories(SYSTEM "${CRITERION_INCLUDE_DIRS}")

# Add a debug message for test sources
message(STATUS "Adding executable for unit_tests with sources: ${TEST_SOURCES}")

add_executable (unit_tests "${TEST_SOURCES}")
# Add a debug message
message(STATUS "Linking unit_tests with libraries: fmem, ${CRITERION_LIBRARIES}")

target_link_libraries (unit_tests fmem "${CRITERION_LIBRARIES}")

add_test (unit_tests unit_tests)
