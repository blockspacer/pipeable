cmake_minimum_required(VERSION 3.8)

option(PIPEABLE_BUILD_TESTS "Build tests" ON)

project(pipeable CXX)

add_library( ${PROJECT_NAME} INTERFACE)
add_library(fho::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
target_include_directories( ${PROJECT_NAME} 
    INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    INTERFACE $<INSTALL_INTERFACE:include>
)
target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_17)

# ----------- INSTALL & EXPORT -----------
include(GNUInstallDirs)
install(
    TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-targets
)
install(
    DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(
    EXPORT ${PROJECT_NAME}-targets
    FILE ${PROJECT_NAME}-config.cmake
    NAMESPACE ${CMAKE_PROJECT_NAME}::
    DESTINATION lib/cmake/${PROJECT_NAME}
)
export(
    TARGETS ${PROJECT_NAME} 
    FILE ${PROJECT_NAME}-config.cmake
)

if(PIPEABLE_BUILD_TESTS)
    # ----------- TESTS -----------
    enable_testing()

    add_subdirectory(catch2)

    add_executable( pipeable_tests
        "tests/pipeable_tests.cpp"
        "tests/data_source_tests.cpp"
        "tests/data_generator_tests.cpp"
    )
    target_link_libraries( pipeable_tests
        pipeable
        catch2
    )
    add_test(
        NAME pipeable_tests
        COMMAND pipeable_tests
    )
endif()