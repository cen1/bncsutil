cmake_minimum_required(VERSION 3.25)

project(bncsutil VERSION 1.4.5)

set(CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR} "${PROJECT_SOURCE_DIR}/CMake/Modules")

add_library(bncsutil SHARED)

set(HEADERS
    "src/bncsutil/bncsutil.h"
    "src/bncsutil/bsha1.h"
    "src/bncsutil/buffer.h"
    "src/bncsutil/cdkeydecoder.h"
    "src/bncsutil/checkrevision.h"
    "src/bncsutil/decodekey.h"
    "src/bncsutil/keytables.h"
    "src/bncsutil/libinfo.h"
    "src/bncsutil/mutil.h"
    "src/bncsutil/mutil_types.h"
    "src/bncsutil/nls.h"
    "src/bncsutil/oldauth.h"
    "src/bncsutil/pe.h"
    "src/bncsutil/sha1.h"
)

set(SOURCES
    "src/bncsutil/bsha1.cpp"
    "src/bncsutil/cdkeydecoder.cpp"
    "src/bncsutil/checkrevision.cpp"
    "src/bncsutil/decodekey.cpp"
    "src/bncsutil/file.cpp"
    "src/bncsutil/libinfo.cpp"
    "src/bncsutil/nls.c"
    "src/bncsutil/oldauth.cpp"
    "src/bncsutil/pe.c"
    "src/bncsutil/sha1.c"
    "src/bncsutil/stack.c"
)

target_sources(bncsutil PRIVATE ${SOURCES} ${HEADERS})

if(WIN32)
    option(USE_SYSTEM_LIBS "Use system libraries" OFF)
else()
    option(USE_SYSTEM_LIBS "Use system libraries" ON)
endif()

if(CMAKE_GENERATOR_PLATFORM STREQUAL "x86")
    target_compile_options(bncsutil PRIVATE -m32)
    target_link_options(bncsutil PRIVATE -m32)
    message(STATUS "Excluding 64bit library paths from search.")
    set_property(GLOBAL PROPERTY FIND_LIBRARY_USE_LIB64_PATHS OFF)
elseif(CMAKE_GENERATOR_PLATFORM STREQUAL "x64")
    target_compile_options(bncsutil PRIVATE -m64)
    target_link_options(bncsutil PRIVATE -m64)
endif()

if(USE_SYSTEM_LIBS)
    message(STATUS "Using system dependencies")
    find_package(GMP REQUIRED)
    target_include_directories(bncsutil PRIVATE src ${GMP_INCLUDE_DIR})
    target_link_libraries(bncsutil PRIVATE ${GMP_LIBRARIES})
else()
    message(STATUS "Using conan dependencies")
    find_package(gmp REQUIRED)
    target_include_directories(bncsutil PRIVATE src)
    target_link_libraries(bncsutil PRIVATE gmp::gmp)
endif()

target_compile_definitions(bncsutil PRIVATE MUTIL_LIB_BUILD)

if(UNIX)
    set_target_properties(bncsutil PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
    )
    target_compile_options(bncsutil PRIVATE
        -Wall -O3 -Wno-multichar -fPIC
        -fvisibility=hidden
        $<$<COMPILE_LANGUAGE:CXX>:-fvisibility-inlines-hidden>
    )
endif()

if(WIN32)
    target_compile_definitions(bncsutil PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

if(MSVC)
    set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT "bncsutil")
endif()

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

install(TARGETS bncsutil
        EXPORT bncsutilTargets
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(FILES ${HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/bncsutil)

install(EXPORT bncsutilTargets
        FILE bncsutilTargets.cmake
        NAMESPACE bncsutil::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/bncsutil
)

configure_package_config_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/CMake/bncsutilConfig.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/bncsutilConfig.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/bncsutil
)

write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/bncsutilConfigVersion.cmake"
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY AnyNewerVersion
)

install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/bncsutilConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/bncsutilConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/bncsutil
)

# CPack configuration
set(CPACK_GENERATOR "DEB" "RPM")
set(CPACK_PACKAGE_NAME "bncsutil")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Battle.Net Chat Service Utility")
set(CPACK_PACKAGE_VENDOR "bncsutil")
set(CPACK_PACKAGE_DESCRIPTION "\
This will install the library to /usr/local/lib and header files to /usr/local/include.\n \
Make sure these directories are in your library and include paths. \
")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGING_INSTALL_PREFIX "/usr/local")

# DEB configuration
set(CPACK_DEBIAN_PACKAGE_SECTION "libs")
set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://github.com/BNETDocs/bncsutil")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "imbacen@gmail.com")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libgmp10")

# RPM configuration
set(CPACK_RPM_PACKAGE_RELEASE 1)
set(CPACK_RPM_PACKAGE_LICENSE "LGPL-2.1")
set(CPACK_RPM_PACKAGE_GROUP "bncsutil")
set(CPACK_RPM_PACKAGE_URL "https://github.com/BNETDocs/bncsutil")
set(CPACK_RPM_PACKAGE_REQUIRES "gmp")

include(CPack)
