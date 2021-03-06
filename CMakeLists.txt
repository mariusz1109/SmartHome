cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

project(smart_home)

enable_language(C CXX)

if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
    set(CMAKE_CXX_FLAGS         "-Wall -Wno-unknown-pragmas -Wno-sign-compare -Woverloaded-virtual -Wwrite-strings -Wno-unused -std=c++0x")
    set(CMAKE_CXX_FLAGS_DEBUG   "-O0 -g3")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fprofile-arcs -ftest-coverage")
endif()

include_directories(
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_SOURCE_DIR}/src/drivers
    )

add_library(
    SmartHomeLib
    src/drivers/I2C.cpp
    )

#This should be remove !!
add_library(
    example
    src/example.cpp
    )

set(GOOGLETEST_ROOT external/googletest/googletest CACHE STRING "Google Test source root")

include_directories(
     ${PROJECT_SOURCE_DIR}/${GOOGLETEST_ROOT}
     ${PROJECT_SOURCE_DIR}/${GOOGLETEST_ROOT}/include
     )
            

set(GOOGLETEST_SOURCES
    ${PROJECT_SOURCE_DIR}/${GOOGLETEST_ROOT}/src/gtest-all.cc
    ${PROJECT_SOURCE_DIR}/${GOOGLETEST_ROOT}/src/gtest_main.cc
    )

foreach(_source ${GOOGLETEST_SOURCES})
    set_source_files_properties(${_source} PROPERTIES GENERATED 1)
endforeach()

add_library(googletest ${GOOGLETEST_SOURCES})

add_executable(
    unit_tests
    test/main.cpp
    test/drivers/test_I2c.cpp
    test/example_add.cpp
    )

add_dependencies(unit_tests googletest)

target_link_libraries(
    unit_tests
    googletest
    example
    SmartHomeLib
    pthread
    )

if(PRODUCTION_BUILD MATCHES Prod)
    message("Production build")

    if(ARCH_TYPE MATCHES Linux_Rpi)
        message("ARCH = Linux_Rpi")
 
        set(ARCH
            arch/linux_Rpi/
            )

        include_directories(
            arch/linux_Rpi
            )
    else()
        message("Lack of ARCH specified")
    endif()

    add_executable(
        SmartHomeMain
        main.cpp
        ${ARCH}/i2c_driver.cpp
        )

    target_link_libraries(
        SmartHomeMain
        SmartHomeLib
        )
else()
    message("Test build")
endif()

include(CTest)
enable_testing()

add_test(unit ${PROJECT_BINARY_DIR}/unit_tests)
