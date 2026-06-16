cmake_minimum_required(VERSION 3.10)
project(variant VERSION 1.0.0 LANGUAGES CXX)

option(VARIANT_BUILD_TESTS "Build the testing suite" ON)

add_library(variant INTERFACE)
add_library(variant::variant ALIAS variant)

target_compile_features(variant INTERFACE cxx_std_17)

target_include_directories(variant INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(VARIANT_BUILD_TESTS)
    find_package(GTest REQUIRED)

    enable_testing()
    
    add_executable(unit_tests 
        test/test_variant.cpp
    )
     
    target_link_libraries(unit_tests 
        PRIVATE 
            variant::variant
            GTest::gtest_main
    )

    # Discover tests automatically for the 'ctest' command
    include(GoogleTest)
    gtest_discover_tests(unit_tests)
endif()

include(GNUInstallDirs)

install(TARGETS variant
    EXPORT variantTargets
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/variantConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(EXPORT variantTargets
    FILE variantTargets.cmake
    NAMESPACE variant::
    DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/variant
)
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/variantConfig.cmake"
    "include(\"\${CMAKE_CURRENT_LIST_DIR}/variantTargets.cmake\")\n"
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/variantConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/variantConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/variant
)
