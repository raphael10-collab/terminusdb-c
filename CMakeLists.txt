cmake_minimum_required(VERSION 3.15.0)
project(terminusdb-c VERSION 0.0.1)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)


include(FetchContent)

FetchContent_Declare(
    Corrosion
    GIT_REPOSITORY https://github.com/corrosion-rs/corrosion.git
    GIT_TAG v0.5 # Optionally specify a commit hash, version tag or branch here
)
FetchContent_MakeAvailable(Corrosion)


option(TERMINUSDB_TESTS_AND_EXAMPLES "Build test and example targets" OFF)

if (TERMINUSDB_TESTS_AND_EXAMPLES)
    include(CTest)
    enable_testing()
endif()

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(MINIMUM_REQUIRED_CXX_STANDARD 17)

if((NOT CMAKE_CXX_STANDARD) OR (CMAKE_CXX_STANDARD LESS MINIMUM_REQUIRED_CXX_STANDARD))
    set(CMAKE_CXX_STANDARD ${MINIMUM_REQUIRED_CXX_STANDARD})
endif()
message(STATUS "C++ standard: ${CMAKE_CXX_STANDARD}")

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(
    BUILD_SHARED_LIBS
    "Build shared library dependencies instead of static."
    OFF)

# Build static and dynamic lib written in Rust by invoking `cargo`.
# Imports `questdb_client` target.
#add_subdirectory(corrosion)
corrosion_import_crate(
    MANIFEST_PATH terminusdb-rs/Cargo.toml
    )
#target_include_directories(
    #terminusdb_client INTERFACE
    #${CMAKE_CURRENT_SOURCE_DIR}/include)
if(WIN32)
    #set_target_properties(
        #terminusdb_client-shared
        #PROPERTIES
        #DEFINE_SYMBOL "LINESENDER_DYN_LIB")
    target_link_libraries(terminusdb_client-shared)
    target_link_libraries(terminusdb_client-static)
endif(WIN32)
if(APPLE)
    target_link_libraries(terminusdb_client)
    target_link_libraries(terminusdb_client)
endif()

function(set_compile_flags TARGET_NAME)
    if(MSVC)
        # We disable warning C5105 via `/wd5105`
        # to work around non-C11-compliant
        # code in WinBase.h which is included from
        # #include <winsock2.h>
        # See: https://docs.microsoft.com/en-us/cpp/
        #   error-messages/compiler-warnings/c5105?view=msvc-170
        # And: https://developercommunity2.visualstudio.com/t/
        #   std:c17-generates-warning-compiling-Win/1249671?preview=true
        # The warning is not applicable in new releases of the Windows SDK.
        target_compile_options(
            ${TARGET_NAME} PRIVATE
            /W4 /WX $<$<COMPILE_LANGUAGE:C>:/wd5105>)
    else()
        target_compile_options(
            ${TARGET_NAME} PRIVATE
            -Wall -Wextra -Wpedantic -Werror)
    endif()
endfunction()

# Examples
function(compile_example TARGET_NAME)
    list(POP_FRONT ARGV)
    add_executable(
        ${TARGET_NAME}
        ${ARGV})
    target_link_libraries(
        ${TARGET_NAME}
        terminusdb_client)
endfunction()

