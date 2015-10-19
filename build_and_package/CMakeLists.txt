cmake_minimum_required(VERSION 2.8)

project(build_ardronelib)

#include(ardrone_build_sdk_in_source.cmake)


## Build ARDrone SDK here
set (ARDRONE_SDK_FLAVOR "official-patched")
# official-patched
# autonomy-lastest
# autonomy-gps


if (${ARDRONE_SDK_FLAVOR} STREQUAL "official-patched")
    set(GIT_TAG bdacd1cbd3fbc54263d29e6e2067265e5941d10e)
elseif (${ARDRONE_SDK_FLAVOR} STREQUAL "autonomy-lastest")
    set(GIT_TAG master)
elseif (${ARDRONE_SDK_FLAVOR} STREQUAL "autonomy-gps")
    set(GIT_TAG gps)
endif()

message("ardronelib version: ${ARDRONE_SDK_FLAVOR}")
set(ARDRONE_SDK_NAME "ardronelib" )
set(ARDRONE_SDK ${CMAKE_BINARY_DIR}/src/${ARDRONE_SDK_NAME}/ARDroneLib)
set(ARDRONE_SDK_LIB_PATH ${CMAKE_BINARY_DIR}/lib/ardrone)

include(ExternalProject)
ExternalProject_Add(ardronelib
        GIT_REPOSITORY git://github.com/AutonomyLab/ardronelib.git
        GIT_TAG ${GIT_TAG}
        PREFIX ${CMAKE_BINARY_DIR}
        CONFIGURE_COMMAND echo "No configure"
        BUILD_COMMAND make
        INSTALL_COMMAND make install INSTALL_PREFIX=${ARDRONE_SDK_LIB_PATH}
        BUILD_IN_SOURCE 1
)

add_custom_target(no_builtin_sdk_examples ALL
	DEPENDS ardronelib
	COMMAND rm -fr "${ARDRONE_SDK}/VP_SDK/Build/PROD_MODE_gcc_4.3.3_Examples"
)
file(WRITE dummy_main.c "int main(int argc, char** argv){return 0;}")
add_executable(dummy_main dummy_main.c)


install(DIRECTORY ${CMAKE_BINARY_DIR}/lib/ardrone/ DESTINATION /usr/local/lib/jderobot/ardrone)
install(DIRECTORY ${CMAKE_BINARY_DIR}/src/ardronelib/ARDroneLib/ DESTINATION /usr/local/include/jderobot/ardrone)
INSTALL (FILES ${CMAKE_CURRENT_SOURCE_DIR}/dummy_main DESTINATION /tmp OPTIONAL PERMISSIONS OWNER_EXECUTE GROUP_EXECUTE WORLD_EXECUTE)

## Include ARDrone into build toolchain
message( "ARDrone SDK at ${ARDRONE_SDK}" )
message( "ARDrone LIBS at ${ARDRONE_SDK_LIB_PATH}" )

include_directories(
    ${ARDRONE_SDK}
    ${ARDRONE_SDK}/FFMPEG/Includes
    ${ARDRONE_SDK}/Soft/Common
    ${ARDRONE_SDK}/Soft/Lib
    ${ARDRONE_SDK}/VP_SDK
    ${ARDRONE_SDK}/VP_SDK/VP_Os/linux
)
link_directories(${ARDRONE_SDK_LIB_PATH})



#
# Deb packages
#

# Determine current architecture
macro(dpkg_arch VAR_NAME)
        find_program(DPKG_PROGRAM dpkg DOC "dpkg program of Debian-based systems")
        if (DPKG_PROGRAM) 
          execute_process(
            COMMAND ${DPKG_PROGRAM} --print-architecture
            OUTPUT_VARIABLE ${VAR_NAME}
            OUTPUT_STRIP_TRAILING_WHITESPACE
          )
        endif(DPKG_PROGRAM)
endmacro(dpkg_arch)

include (InstallRequiredSystemLibraries)
SET (CPACK_GENERATOR "DEB")
SET (CPACK_SOURCE_GENERATOR TGZ)
SET (CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
SET (CPACK_SET_DESTDIR "on")
SET (CPACK_PACKAGING_INSTALL_PREFIX "/usr/local")


SET (VERSION 1.0.0)
# CPack version numbers for release tarball name.
SET (CPACK_PACKAGE_VERSION_MAJOR 1)
SET (CPACK_PACKAGE_VERSION_MINOR 0)
SET (CPACK_PACKAGE_VERSION_PATCH 0)
SET (CPACK_DEBIAN_PACKAGE_VERSION ${VERSION})


SET (CPACK_DEBIAN_PACKAGE_PRIORITY "extra")
SET (CPACK_DEBIAN_PACKAGE_SECTION "net")
dpkg_arch(CPACK_DEBIAN_PACKAGE_ARCHITECTURE)

#MESSAGE("Dependencias: ${DEPS}")

SET(CPACK_DEBIAN_PACKAGE_DEPENDS "daemontools, libsdl1.2-dev, libgtk2.0-dev, libxml2-dev, libudev-dev")

#set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA
#    "${CMAKE_CURRENT_SOURCE_DIR}/scripts/cmake/postinst"
#    "${CMAKE_CURRENT_SOURCE_DIR}/scripts/cmake/postrm")

SET (CPACK_PACKAGE_DESCRIPTION_SUMMARY "ArDrone SDK for JdeRobot")
SET (CPACK_PACKAGE_DESCRIPTION "ARDrone SDK for Jderobot description")

SET (CPACK_PACKAGE_CONTACT "Roberto Calvo <rocapal@gsyc.urjc.es>")
SET (CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}_${VERSION}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")

SET (CPACK_COMPONENTS_ALL Libraries ApplicationData)
include (CPack Documentation)
