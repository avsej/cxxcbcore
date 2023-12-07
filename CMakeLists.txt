cmake_minimum_required(VERSION 3.14)

project(cxxcbcore LANGUAGES CXX VERSION 0.0.1)

if (DEFINED BUILD_SHARED_LIBS)
    set(CXXCBCORE_SHARED_LIBRARY ${BUILD_SHARED_LIBS})
endif()


set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(GNUInstallDirs)

set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)

include(GenerateExportHeader)
generate_export_header(cxxcbcore)

configure_file("${PROJECT_SOURCE_DIR}/include/cxxcbcore/cxxcbcore_config.hxx.in"
               "${PROJECT_BINARY_DIR}/cxxcbcore/cxxcbcore_config.hxx")

set(CXXCBCORE_INCLUDES
    ${PROJECT_SOURCE_DIR}/include/cxxcbcore/cxxcbcore.hxx
    ${PROJECT_BINARY_DIR}/cxxcbcore/cxxcbcore_config.hxx)

set(CXXCBCORE_SOURCES
    ${PROJECT_SOURCE_DIR}/src/cxxcbcore.cxx)

function(configure_snitch_exports TARGET)
    if (BUILD_SHARED_LIBS)
        target_compile_definitions(${TARGET} PRIVATE CXXCBCORE_EXPORTS)
        if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC" OR MINGW)
            # Nothing to do; default is already to hide symbols unless exported.
        elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR
            CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR
            CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang")
            # Set default visibility to "hidden" so only exported symbols are visible.
            target_compile_options(${TARGET} PRIVATE -fvisibility=hidden)
            target_compile_options(${TARGET} PRIVATE -fvisibility-inlines-hidden)
        endif()
    endif()
endfunction()


add_library(${CXXCBCORE_TARGET_NAME} ${CXXCBCORE_INCLUDES} ${CXXCBCORE_SOURCES})
target_compile_features(${CXXCBCORE_TARGET_NAME} PUBLIC cxx_std_23)
target_include_directories(${CXXCBCORE_TARGET_NAME} PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include>)

configure_snitch_exports(${CXXCBCORE_TARGET_NAME})

install(
    FILES ${CXXCBCORE_INCLUDES}
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/snitch)

add_library(cxxcbcore::${CXXCBCORE_TARGET_NAME} ALIAS ${CXXCBCORE_TARGET_NAME})
set_target_properties(${CXXCBCORE_TARGET_NAME} PROPERTIES EXPORT_NAME cxxcbcore::${CXXCBCORE_TARGET_NAME})

set(CXXCBCORE_TARGET_NAME cxxcbcore)

install(TARGETS ${CXXCBCORE_TARGET_NAME} EXPORT ${CXXCBCORE_TARGET_NAME}-targets)

install(EXPORT ${CXXCBCORE_TARGET_NAME}-targets
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/cxxcbcore COMPONENT Development)

export(EXPORT ${CXXCBCORE_TARGET_NAME}-targets)

include(CMakePackageConfigHelpers)
configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/cxxcbcore-config.cmake.in"
    "${PROJECT_BINARY_DIR}/cxxcbcore-config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
    NO_SET_AND_CHECK_MACRO)

install(FILES
    "${PROJECT_BINARY_DIR}/cxxcbcore-config.cmake"
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/cxxcbcore COMPONENT Development)

if (CXXCBCORE_DO_TEST)
    enable_testing()


    add_subdirectory(tests)
endif()
