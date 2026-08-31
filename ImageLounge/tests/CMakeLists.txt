include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/DkCore)

include(GoogleTest)

# dependencies used for all tests
add_library(test_libs INTERFACE)
target_link_libraries(
    test_libs #
    INTERFACE #
        ${DLL_CORE_NAME}
        ${OpenCV_LIBS}
        GTest::gtest_main
        Qt${QT_VERSION_MAJOR}::Core
        Qt${QT_VERSION_MAJOR}::Gui
)

add_custom_target(tests)

function(add_ctest_test name)
    add_executable(${name} ${ARGN})
    target_link_libraries(${name} test_libs)
    add_test(NAME ${name} COMMAND ${name})
    add_dependencies(tests ${name})
endfunction()

# tests with no side-effects can all go in the same binary
add_ctest_test(
    tests_core #
    DkUtils_test.cpp
    DkBaseViewPort_test.cpp
    DkNativeImage_test.cpp
    DkMetaData_test.cpp
)

add_ctest_test(
    test_shortcutsmodel
    DkShortcutsModel_test.cpp
)

# set environment variables so `ctest` is run against this build's products
file(
    WRITE
    "${CMAKE_CURRENT_BINARY_DIR}/GlobalCTestEnv.cmake"
    "# environment overrides for ctest
set(ENV{NOMACS_TESTING} 1)
set(ENV{QT_PLUGIN_PATH} \"${CMAKE_BINARY_DIR}/nomacs-plugins:\$ENV{QT_PLUGIN_PATH}\")
set(ENV{LD_LIBRARY_PATH} \"${CMAKE_BINARY_DIR}:\$ENV{LD_LIBRARY_PATH}\")
set(ENV{DYLD_LIBRARY_PATH} \"${CMAKE_BINARY_DIR}:\$ENV{DYLD_LIBRARY_PATH}\")
"
)
set_property(DIRECTORY PROPERTY TEST_INCLUDE_FILE "${CMAKE_CURRENT_BINARY_DIR}/GlobalCTestEnv.cmake")

# convenience script for running test binaries directly (gdb etc)
file(
    WRITE
    "${CMAKE_BINARY_DIR}/test.env"
    "# source this file when running nomacs test binaries directly (not required for ctest)
set -a
export NOMACS_TESTING=1
export QT_PLUGIN_PATH=\"${CMAKE_BINARY_DIR}/nomacs-plugins:\$QT_PLUGIN_PATH\"
export LD_LIBRARY_PATH=\"${CMAKE_BINARY_DIR}:\$LD_LIBRARY_PATH\"
export DYLD_LIBRARY_PATH=\"${CMAKE_BINARY_DIR}:\$DYLD_LIBRARY_PATH\"
"
)

# cmake's "make test" target does not have any dependencies, add "make check" for that
add_custom_target(
    check #
    COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure
    DEPENDS tests
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
