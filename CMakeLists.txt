cmake_minimum_required(VERSION 3.19)
project(change-refactoring-cpp-kata CXX)

function(unzip_file zip_file destination)
    if(WIN32)
        # Windows example using PowerShell
        # Ensures PowerShell is available to unzip the file
        execute_process(
                COMMAND powershell -Command "Expand-Archive -Path '${zip_file}' -DestinationPath '${destination}' -Force"
                RESULT_VARIABLE result
        )
    elseif(UNIX)
        # UNIX example using the unzip command
        # Assumes unzip command is available
        execute_process(
                COMMAND unzip -o "${zip_file}" -d "${destination}"
                RESULT_VARIABLE result
        )
    else()
        message(FATAL_ERROR "Unzip not supported on this platform")
    endif()

    if(NOT result EQUAL 0)
        message(FATAL_ERROR "Failed to unzip ${zip_file}")
    endif()
endfunction()

macro(get_downloads_folder downloads_folder)
    if(WIN32)
        # On Windows, use backslashes
        set(home_dir "$ENV{USERPROFILE}")
        string(REPLACE "\\" "/" home_dir "${home_dir}") # Ensure backslashes
        set(${downloads_folder} "${home_dir}/Downloads")
    elseif(UNIX)
        # On Unix-like systems, use forward slashes
        set(home_dir "$ENV{HOME}")
        set(${downloads_folder} "${home_dir}/Downloads")
    else()
        message(FATAL_ERROR "Unsupported platform")
    endif()
endmacro()


include(FetchContent)
get_downloads_folder(downloads_folder)
#set(home_dir "$ENV{USERPROFILE}")
#string(REPLACE "\\" "/" home_dir "${home_dir}")

unzip_file(${downloads_folder}/googletest-5376968f6948923e2411081fd9372e71a59d8e77.zip ${downloads_folder})

FetchContent_Declare(
        googletest
        # Specify the commit you depend on and update it regularly.
        SOURCE_DIR ${downloads_folder}/googletest-5376968f6948923e2411081fd9372e71a59d8e77
)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

enable_testing()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_INCLUDE_DIRECTORIES ${CMAKE_CXX_IMPLICIT_INCLUDE_DIRECTORIES})
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

#find_package(googletest REQUIRED)

add_executable(main main.cpp)

file(GLOB tests test_*.cpp)
foreach(test ${tests})
    get_filename_component(name ${test} NAME_WE)
    add_executable(${name} ${test})
    add_test(${name} ${name})
    #target_link_libraries(${name} PRIVATE googletest::gmock googletest::gmock_main)
    target_link_libraries(${name} PRIVATE GTest::gtest_main GTest::gmock_main)
endforeach()

add_custom_target(  
    copy-compile-commands ALL
    ${CMAKE_COMMAND} -E copy_if_different
    ${CMAKE_BINARY_DIR}/compile_commands.json
    ${CMAKE_CURRENT_LIST_DIR}
)
