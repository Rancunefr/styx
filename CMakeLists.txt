### CMake file for styx
###
###   -  2019: Initial version <glaconbleu@codices.org>
###
###
###############################################################################


cmake_minimum_required (VERSION 2.6)

project (styx)
set (styx_VERSION_MAJOR 0)
set (styx_VERSION_MINOR 1)

set( CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} /lib /usr/lib /usr/local/lib )
set( CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH} /usr/include /usr/local/include )
set( CMAKE_MODULE_PATH  ${CMAKE_MODULE_PATH}  "${PROJECT_SOURCE_DIR}/cmake" ) 

set ( INSTALL_BIN_PATH bin )
set ( INSTALL_LIB_PATH lib )
set ( INSTALL_AR_PATH  lib )
set ( INSTALL_HEADERS_PATH include )


# Make relative paths absolute (needed later on)

foreach(p LIB BIN HEADERS AR)
    set(var INSTALL_${p}_PATH)
    if(NOT IS_ABSOLUTE "${${var}}")
        set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()


### Search for Lib/Headers

find_package(X11)
find_package(Freetype)
find_package(Fontconfig)

### Configure a header file to pass some of the CMake settings
### to the source code

configure_file (
    "${PROJECT_SOURCE_DIR}/config.h.in"
    "${PROJECT_SOURCE_DIR}/src/config_cmake.h"
)


### Set include directories 
        
include_directories( "${CMAKE_CURRENT_SOURCE_DIR}/include"
	             "${CMAKE_CURRENT_SOURCE_DIR}/src"
        	     "${CMAKE_CURRENT_BINARY_DIR}"
                 "${CMAKE_INSTALL_PREFIX}/include"
                 "${FREETYPE_INCLUDE_DIRS}"
                 "${Fontconfig_INCLUDE_DIRS}"
                 "${X11_INCLUDE_DIR}" )


### File Lists

file( GLOB hdr "src/*.h*" )
file( GLOB src "src/*.c" )

### Main Target and linking

add_executable( "styx" ${src} ${hdr} )

target_link_libraries ( "styx" m util 
                        ${FREETYPE_LIBRARIES} 
                        ${X11_LIBRARIES} 
                        ${Fontconfig_LIBRARIES} 
                        ${X11_Xft_LIB} )

target_compile_definitions( "styx" PRIVATE _XOPEN_SOURCE=600)

### Install targets

install( TARGETS "styx"
        RUNTIME DESTINATION ${INSTALL_BIN_PATH}  COMPONENT main
        LIBRARY DESTINATION ${INSTALL_LIB_PATH}  COMPONENT main
        ARCHIVE DESTINATION ${INSTALL_AR_PATH}   COMPONENT main )

### Manpage

add_custom_target(man ALL)

add_custom_command(
	TARGET man 
	COMMAND sed
	"s/VERSION/${styx_VERSION_MAJOR}.${styx_VERSION_MINOR}/g"
		< ${CMAKE_CURRENT_SOURCE_DIR}/styx.1
		> ${CMAKE_CURRENT_BINARY_DIR}/styx.1
	OUTPUTS ${CMAKE_CURRENT_BINARY_DIR}/styx.1 )

install( FILES ${CMAKE_CURRENT_BINARY_DIR}/styx.1 
	 DESTINATION ${CMAKE_INSTALL_PREFIX}/share/man/man1 )

### terminfo

install(CODE "execute_process( COMMAND tic -sx ${CMAKE_CURRENT_SOURCE_DIR}/styx.info )")








