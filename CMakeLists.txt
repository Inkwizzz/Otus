cmake_minimum_required(VERSION 3.12)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

project(untitled1 VERSION ${PROJECT_VERSION})

option(WITH_BOOST_TEST "Whether to build Boost test" ON)

configure_file(version.h.in version.h)

add_executable(untitled1_cli main.cpp)
add_library(untitled1 lib.cpp
        lib.cpp)

set_target_properties(untitled1_cli untitled1 PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
)

target_include_directories(untitled1
        PRIVATE "${CMAKE_BINARY_DIR}"
)

target_link_libraries(untitled1_cli PRIVATE
        untitled1
)


if(WITH_BOOST_TEST)
    find_package(Boost COMPONENTS unit_test_framework REQUIRED)
    add_executable(test_version test_version.cpp)

    set_target_properties(test_version PROPERTIES
            CXX_STANDARD 14
            CXX_STANDARD_REQUIRED ON
    )

    set_target_properties(test_version PROPERTIES
            COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
            INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
    )

    target_link_libraries(test_version
            ${Boost_LIBRARIES}
            untitled1
    )
endif()

if (MSVC)
    target_compile_options(untitled1_cli PRIVATE
            /W4
    )
    target_compile_options(untitled1 PRIVATE
            /W4
    )
    if(WITH_BOOST_TEST)
        target_compile_options(test_version PRIVATE
                /W4
        )
    endif()
else ()
    target_compile_options(untitled1_cli PRIVATE
            -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(untitled1 PRIVATE
            -Wall -Wextra -pedantic -Werror
    )
    if(WITH_BOOST_TEST)
        target_compile_options(test_version PRIVATE
                -Wall -Wextra -pedantic -Werror
        )
    endif()
endif()

install(TARGETS untitled1_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT example@example.com)
include(CPack)

if(WITH_BOOST_TEST)
    enable_testing()
    add_test(test_version test_version)
endif()