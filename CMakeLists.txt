#*******************************************************************************
#  Copyright (c) 2015, 2018 logi.cals GmbH and others
#
#  All rights reserved. This program and the accompanying materials
#  are made available under the terms of the Eclipse Public License v1.0
#  and Eclipse Distribution License v1.0 which accompany this distribution.
#
#  The Eclipse Public License is available at
#     http://www.eclipse.org/legal/epl-v10.html
#  and the Eclipse Distribution License is available at
#    http://www.eclipse.org/org/documents/edl-v10.php.
#
#  Contributors:
#     Rainer Poisel - initial version
#     Genis Riera Perez - Add support for building debian package
#*******************************************************************************/

# Note: on OS X you should install XCode and the associated command-line tools

CMAKE_MINIMUM_REQUIRED(VERSION 3.0.0)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.190.tar.gz"
    SHA1 "bc7da329d2493201de28f0c2a5fcd4de1415c477"
)

PROJECT(paho.mqtt.c VERSION 1.3.0 LANGUAGES C)
MESSAGE(STATUS "CMake version: " ${CMAKE_VERSION})
MESSAGE(STATUS "CMake system name: " ${CMAKE_SYSTEM_NAME})

SET(CMAKE_SCRIPTS "${CMAKE_SOURCE_DIR}/cmake")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

## build settings
SET(PAHO_VERSION_MAJOR 1)
SET(PAHO_VERSION_MINOR 3)
SET(PAHO_VERSION_PATCH 0)
SET(CLIENT_VERSION ${PAHO_VERSION_MAJOR}.${PAHO_VERSION_MINOR}.${PAHO_VERSION_PATCH})

INCLUDE(GNUInstallDirs)

STRING(TIMESTAMP BUILD_TIMESTAMP UTC)
MESSAGE(STATUS "Timestamp is ${BUILD_TIMESTAMP}")

IF(WIN32)
  ADD_DEFINITIONS(-D_CRT_SECURE_NO_DEPRECATE -DWIN32_LEAN_AND_MEAN -MD)
ELSEIF(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
  ADD_DEFINITIONS(-DOSX)
ENDIF()

## build options
SET(PAHO_WITH_SSL TRUE CACHE BOOL "Flag that defines whether to build ssl-enabled binaries too. ")
SET(PAHO_BUILD_STATIC TRUE CACHE BOOL "Build static library")
SET(PAHO_BUILD_DOCUMENTATION FALSE CACHE BOOL "Create and install the HTML based API documentation (requires Doxygen)")
SET(PAHO_BUILD_SAMPLES FALSE CACHE BOOL "Build sample programs")
SET(PAHO_BUILD_DEB_PACKAGE FALSE CACHE BOOL "Build debian package")
SET(PAHO_ENABLE_TESTING TRUE CACHE BOOL "Build tests and run")
SET(PAHO_ENABLE_CPACK TRUE CACHE BOOL "Enable CPack")


ADD_SUBDIRECTORY(src)
IF(PAHO_BUILD_SAMPLES)
    ADD_SUBDIRECTORY(src/samples)
ENDIF()

IF(PAHO_BUILD_DOCUMENTATION)
    ADD_SUBDIRECTORY(doc)
ENDIF()

IF (PAHO_ENABLE_CPACK)
       ### packaging settings
       FILE(GLOB samples "src/samples/*.c")
       INSTALL(FILES ${samples} DESTINATION samples)

       SET(CPACK_PACKAGE_VENDOR "Eclipse Paho")
       SET(CPACK_PACKAGE_NAME "Eclipse-Paho-MQTT-C")
       INSTALL(FILES CONTRIBUTING.md epl-v10 edl-v10 README.md notice.html DESTINATION .)

       IF (WIN32)
               SET(CPACK_GENERATOR "ZIP")
       ELSEIF(PAHO_BUILD_DEB_PACKAGE)
               INSTALL(FILES CONTRIBUTING.md epl-v10 edl-v10 README.md notice.html DESTINATION .)

               SET(CPACK_GENERATOR "DEB")
               CONFIGURE_FILE(${CMAKE_SCRIPTS}/CPackDebConfig.cmake.in
                       ${CMAKE_BINARY_DIR}/CPackDebConfig.cmake @ONLY)
               SET(CPACK_PROJECT_CONFIG_FILE ${CMAKE_BINARY_DIR}/CPackDebConfig.cmake)
               ADD_SUBDIRECTORY(debian)
       ELSE()
               SET(CPACK_GENERATOR "TGZ")
       ENDIF()
ELSE()
       FILE(GLOB samples "src/samples/*.c")
       INSTALL(FILES ${samples} DESTINATION ${CMAKE_INSTALL_DOCDIR})

ENDIF()

SET(CPACK_PACKAGE_VERSION_MAJOR ${PAHO_VERSION_MAJOR})
SET(CPACK_PACKAGE_VERSION_MINOR ${PAHO_VERSION_MINOR})
SET(CPACK_PACKAGE_VERSION_PATCH ${PAHO_VERSION_PATCH})
INCLUDE(CPack)

IF(PAHO_ENABLE_TESTING)
    ENABLE_TESTING()
    INCLUDE_DIRECTORIES(test src)
    ADD_SUBDIRECTORY(test)
ELSE()
    INCLUDE_DIRECTORIES(src)
ENDIF()

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "libpahomqttcTargets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: variable 'targets_export_name' used
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)

install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
)

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
)