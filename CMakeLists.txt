#
# CMakeQt: CMake build system template for Qt based projects
# Author: Rohit Yadav <rohityadav89 AT gmail.com>
#

# Project Info
SET(PROJECT_NAME_SHORT   "CMakeQt")
SET(PROJECT_NAME_LONG    "CMake template Qt project")
SET(PROJECT_DESCRIPTION  "CMake build system template for Qt based projects")
SET(PROJECT_COPYRIGHT    "Copyright (C) 2009-2015 Rohit Yadav")
SET(PROJECT_CONTACT      "rohit@scaleninja.com")
SET(PROJECT_VENDOR       "Rohit Yadav")
SET(ORG_WEBSITE          "http://bhaisaab.org")

# Configuring build type
IF(NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE "Debug" CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
    MESSAGE("Setting build type to '${CMAKE_BUILD_TYPE}'")
ELSE(NOT CMAKE_BUILD_TYPE)
    MESSAGE("Build type set to '${CMAKE_BUILD_TYPE}'")
ENDIF(NOT CMAKE_BUILD_TYPE)

IF(WIN32)
    # Building in release for WIN32
    SET(CMAKE_BUILD_TYPE "Release")
    SET_PROPERTY(DIRECTORY APPEND PROPERTY COMPILE_DEFINITIONS_RELEASE QT_NO_DEBUG)
    # Use mingw's sprintf instead of windows's
    ADD_DEFINITIONS( -D__USE_MINGW_ANSI_STDIO=1 )
ENDIF(WIN32)

# For projects using Git, use git version patch
IF("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    SET(PATCH_OUT "0")
ELSE("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    # Find Git Version Patch
    FIND_PROGRAM(GIT git)
    IF(GIT)
        EXECUTE_PROCESS(
            WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
            COMMAND ${GIT} rev-parse --short HEAD 
            OUTPUT_VARIABLE GIT_OUT OUTPUT_STRIP_TRAILING_WHITESPACE
        )
        SET(PATCH_OUT "0-git${GIT_OUT}")
    ELSE(GIT)
        SET(PATCH_OUT "0")
    ENDIF(GIT)
ENDIF("${CMAKE_BUILD_TYPE}" STREQUAL "Release")

SET(PROJECT_VERSION_PATCH ${PATCH_OUT})

# Project Version
SET(CPACK_PACKAGE_VERSION_MAJOR "0")
SET(CPACK_PACKAGE_VERSION_MINOR "3")
SET(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
SET(CPACK_PACKAGE_VERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH})
SET(PROJECT_VERSION_MAJOR ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR})
SET(PROJECT_VERSION ${CPACK_PACKAGE_VERSION})

MESSAGE(STATUS "Version: ${PROJECT_VERSION}")

STRING(TOLOWER ${PROJECT_NAME_SHORT} PROJECT_NAME_SHORT_LOWER)
PROJECT(${PROJECT_NAME_SHORT_LOWER}-${PROJECT_VERSION})

# Codename of the Project
SET(CODENAME "topsecret")

# System on which project is built
SITE_NAME(HOSTNAME)
SET(SYSNAME ${CMAKE_SYSTEM}-${CMAKE_SYSTEM_PROCESSOR})

# CMake Settings
CMAKE_MINIMUM_REQUIRED(VERSION 2.8.0)
SET(CMAKE_COLOR_MAKEFILE ON)
SET(CMAKE_VERBOSE_MAKEFILE OFF)

# Set path to additional CMake modules
SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

# Path to contribs
SET(CONTRIBS_PATH ${CMAKE_CURRENT_SOURCE_DIR}/contribs)

# On Mac OS build a universal 'release' binary
IF(APPLE)
    SET(CMAKE_BUILD_TYPE "Release")
    SET(CMAKE_OSX_ARCHITECTURES x86_64)
ENDIF(APPLE)

IF (UNIX)
    SET(WITH_PROFILING FALSE CACHE BOOL "Build with profiling support" )
    SET(CMAKE_INSTALL_PREFIX /usr)
ENDIF(UNIX)

# Search and include Qt4 parameters
# Set Qt min required version and add modules here

IF(WIN32)
    # FindQt4.cmake is so brain-dead when Xcompiling
    SET(QT_LIBRARY_DIR ${CONTRIBS_PATH}/lib)
    SET(QT_INCLUDE_DIR ${CONTRIBS_PATH}/include/qt4)
    ADD_DEFINITIONS( -I${CONTRIBS_PATH}/include/qt4/QtCore)
    ADD_DEFINITIONS( -I${CONTRIBS_PATH}/include/qt4/QtGui)
    ADD_DEFINITIONS( -I${CONTRIBS_PATH}/include/qt4/QtXml)
    ADD_DEFINITIONS( -I${CONTRIBS_PATH}/include/qt4/QtNetwork)
    ADD_DEFINITIONS( -I${CONTRIBS_PATH}/include/qt4/)
    FIND_PACKAGE(Qt4 4.6.0 COMPONENTS QtCore QtGui QtXml QtSvg QtNetwork REQUIRED )
ELSE(WIN32)
    FIND_PACKAGE(Qt4 4.6.0 COMPONENTS QtCore QtGui QtXml QtSvg QtNetwork REQUIRED )
ENDIF(WIN32)

IF (NOT WIN32)
    #FindQt4.cmake is so brain-dead when Xcompiling
    SET(CMAKEQT_LIBS ${QT_QTCORE_LIBRARY}
                     ${QT_QTGUI_LIBRARY}
                     ${QT_QTSVG_LIBRARY}
                     ${QT_QTNETWORK_LIBRARY}
                     ${QT_QTXML_LIBRARY}) # Add here any other QT module or other library
    INCLUDE(${QT_USE_FILE})
    ADD_DEFINITIONS(${QT_DEFINITIONS})
ELSE (NOT WIN32)
    SET (CMAKEQT_LIBS -L${CONTRIBS_PATH}/lib # Manually specify paths
            ${CONTRIBS_PATH}/lib/libQtNetwork.a
            ${CONTRIBS_PATH}/lib/libQtXml.a
            ${CONTRIBS_PATH}/lib/libQtGui.a
            ${CONTRIBS_PATH}/lib/libQtCore.a
            ws2_32 winmm imm32 libeay32 ssleay32 crypt32 gdi32)
ENDIF(NOT WIN32)

# Copy the sh launcher
IF(NOT WIN32)
    CONFIGURE_FILE("${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmakeqt.sh" "${CMAKE_CURRENT_BINARY_DIR}/cmakeqt" COPYONLY)
ENDIF(NOT WIN32)

# Layout of CMakeQt installation
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Set common dirs for MAC and UNIX
SET(DEFAULT_BIN_SUBDIR     bin)
SET(DEFAULT_LIB_SUBDIR     lib)
SET(DEFAULT_DATA_SUBDIR    share)
SET(DEFAULT_SRC_SUBDIR     src)
SET(DEFAULT_INCLUDE_SUBDIR include)

# Set CMAKEQT path params
SET(CMAKEQT_BIN_SUBDIR     "${DEFAULT_BIN_SUBDIR}"        CACHE STRING "Subdirectory where executables will be installed")
SET(CMAKEQT_LIB_SUBDIR     "${DEFAULT_LIB_SUBDIR}"        CACHE STRING "Subdirectory where libraries will be installed")
SET(CMAKEQT_DATA_SUBDIR    "${DEFAULT_DATA_SUBDIR}"       CACHE STRING "Subdirectory where CMAKEQT's data will be installed")
SET(CMAKEQT_SRC_SUBDIR     "${DEFAULT_SRC_SUBDIR}"        CACHE STRING "Subdirectory where CMAKEQT's source is")
SET(CMAKEQT_INCLUDE_SUBDIR "${DEFAULT_INCLUDE_SUBDIR}"    CACHE STRING "Subdirectory where header files will be installed")

# Full paths for the installation
SET(CMAKEQT_BIN_DIR     ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_BIN_SUBDIR})
SET(CMAKEQT_LIB_DIR     ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_LIB_SUBDIR})
SET(CMAKEQT_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_INCLUDE_SUBDIR})
SET(CMAKEQT_DATA_DIR    ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_DATA_SUBDIR}/cmakeqt/)
SET(CMAKEQT_DOC_DIR     ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_DATA_SUBDIR}/doc/cmakeqt/)
SET(CMAKEQT_EFFECTS_DIR ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_LIB_SUBDIR}/cmakeqt/effects/)
SET(CMAKEQT_DESKTOP_DIR ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_DATA_SUBDIR}/applications/)
SET(CMAKEQT_PIXMAPS_DIR ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_DATA_SUBDIR}/pixmaps/)

# Manual page - makes sense only on unix systems
IF(UNIX)
    SET(DEFAULT_MANUAL_SUBDIR  ${CMAKEQT_DATA_SUBDIR}/man)
    SET(CMAKEQT_MANUAL_SUBDIR  "${DEFAULT_MANUAL_SUBDIR}"  CACHE STRING "Subdirectory where manual files will be installed")
    MARK_AS_ADVANCED(CMAKEQT_MANUAL_SUBDIR)
    SET(CMAKEQT_MANUAL_DIR  ${CMAKE_INSTALL_PREFIX}/${CMAKEQT_MANUAL_SUBDIR})
ENDIF(UNIX)

# Recursively add sub dirs
SUBDIRS(ts)
SUBDIRS(src)
SUBDIRS(nsis)

# Copy stuff to doc subdirectory
INSTALL (FILES AUTHORS COPYING INSTALL NEWS README TRANSLATORS
         DESTINATION ${CMAKEQT_DOC_DIR})

# Copy .desktop file
INSTALL (FILES share/cmakeqt.desktop DESTINATION ${CMAKEQT_DESKTOP_DIR})

# Copy the CMAKEQT pixmap
INSTALL (FILES share/cmakeqt.png DESTINATION ${CMAKEQT_PIXMAPS_DIR})

# Copy the share directory
#INSTALL (DIRECTORY share/ DESTINATION ${CMAKEQT_DATA_DIR}
#         PATTERN "*.desktop" EXCLUDE
#         PATTERN "*.ico" EXCLUDE
#         PATTERN "*.icns" EXCLUDE
#         PATTERN "cmakeqt.png" EXCLUDE)

#############################################################
# Enable packaging

INCLUDE(InstallRequiredSystemLibraries)

SET(CPACK_SYSTEM_NAME ${CMAKE_SYSTEM_NAME})
SET(CPACK_PACKAGE_NAME ${PROJECT_NAME_SHORT})
SET(CPACK_PACKAGE_DESCRIPTION "${PROJECT_NAME_LONG}")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${PROJECT_NAME_SHORT} - ${PROJECT_DESCRIPTION}")
SET(CPACK_PACKAGE_VENDOR ${PROJECT_VENDOR})
SET(CPACK_PACKAGE_CONTACT ${PROJECT_CONTACT})
SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/README")
SET(CPACK_RESOURCE_FILE_README ${CMAKE_SOURCE_DIR}/README)
SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/COPYING")
SET(CPACK_PACKAGE_INSTALL_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}")

# Force Package Name
SET(CPACK_PACKAGE_FILE_NAME ${PROJECT_NAME}-${CPACK_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR})

# NSIS Windows installer
IF(WIN32 AND NOT UNIX)
    # There is a bug in NSI that does not handle full unix paths properly. Make
    # sure there is at least one set of four (4) backslashes.
    SET(CPACK_PACKAGE_ICON "${CMAKE_SOURCE_DIR}/share\\\\cmakeqt.png")
    SET(CPACK_NSIS_INSTALLED_ICON_NAME "\\\\cmakeqt.exe")
    SET(CPACK_NSIS_DISPLAY_NAME "${CPACK_PACKAGE_INSTALL_DIRECTORY} CMAKEQT")
    SET(CPACK_NSIS_HELP_LINK "http:\\\\\\\\rohityadav.in")
    SET(CPACK_NSIS_URL_INFO_ABOUT "http:\\\\\\\\rohityadav.in")
    SET(CPACK_NSIS_CONTACT "rohityadav89 AT gmail.com")
    SET(CPACK_NSIS_MODIFY_PATH ON)
ENDIF(WIN32 AND NOT UNIX)

# RPM packages
INCLUDE(${CMAKE_MODULE_PATH}/RpmBuild.cmake)
IF(RPMBUILD_FOUND AND NOT WIN32)
    SET(CPACK_GENERATOR "RPM")
    SET(CPACK_RPM_PACKAGE_SUMMARY ${CPACK_PACKAGE_DESCRIPTION_SUMMARY})
    SET(CPACK_RPM_PACKAGE_NAME ${CPACK_PACKAGE_NAME})
    SET(CPACK_RPM_PACKAGE_VERSION ${PROJECT_VERSION})
    SET(CPACK_RPM_PACKAGE_LICENSE "GPLv2+ License")
    SET(CPACK_RPM_PACKAGE_GROUP ${PROJECT_VENDOR})
    SET(CPACK_RPM_PACKAGE_VENDOR ${PROJECT_VENDOR})
    SET(CPACK_RPM_PACKAGE_DESCRIPTION ${PROJECT_DESCRIPTION})
    SET(CPACK_RPM_PACKAGE_DEPENDS "qt ( >= 4.6 ), qt-x11 ( >= 4.6 )") # Specify dependencies here
    SET(CPACK_SET_DESTDIR TRUE)
ENDIF(RPMBUILD_FOUND AND NOT WIN32)

# Debian packages
INCLUDE (${CMAKE_MODULE_PATH}/DpkgBuild.cmake)
IF(DPKG_FOUND AND NOT WIN32)
    SET(CPACK_GENERATOR "DEB")
    SET(CPACK_DEBIAN_PACKAGE_NAME ${CPACK_PACKAGE_NAME})
    SET(CPACK_DEBIAN_PACKAGE_VERSION ${CPACK_PACKAGE_VERSION})
    SET(CPACK_DEBIAN_PACKAGE_DESCRIPTION ${CPACK_PACKAGE_DESCRIPTION_SUMMARY})
    SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_VENDOR} <${CPACK_PACKAGE_CONTACT}>")
    SET(CPACK_DEBIAN_PACKAGE_PRIORITY "optional")
    SET(CPACK_DEBIAN_PACKAGE_DEBUG ON)
    SET(CPACK_DEBIAN_PACKAGE_DEPENDS ${PACKAGE_REQUIRES})
    SET(CPACK_DEBIAN_PACKAGE_DEPENDS "libqt4-gui ( >= 4.6 ), libqt4-xml ( >= 4.6 )") # Specify dependencies here
    SET(CPACK_SET_DESTDIR TRUE)
ENDIF(DPKG_FOUND AND NOT WIN32)

# Mac App Bundle
IF(APPLE)
    SET(CPACK_GENERATOR "DragNDrop")
    # Libraries are bundled directly
    SET(CPACK_COMPONENT_LIBRARIES_HIDDEN TRUE)
    # Bundle Properties
    SET(MACOSX_BUNDLE_BUNDLE_NAME CMakeQt)
    SET(MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION})
    SET(MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION})
    SET(MACOSX_BUNDLE_LONG_VERSION_STRING "Version ${PROJECT_VERSION}")
ENDIF(APPLE)

SET(CPACK_PACKAGE_EXECUTABLES "cmakeqt" "CMAKEQT")
INCLUDE(CPack)
