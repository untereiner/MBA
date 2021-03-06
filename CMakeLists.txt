cmake_minimum_required(VERSION 3.7.2 FATAL_ERROR)

project(mba)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

option(BUILD_SHARED_LIBS "Build shared library" OFF)
option(BUILD_EXAMPLES "Build examples" OFF)

add_library(${PROJECT_NAME} "")

FILE(GLOB_RECURSE mba_SRCS src/*.cpp include/*.h)
target_sources(${PROJECT_NAME}
	PRIVATE
		${mba_SRCS}
)


# Set various compiler flags

target_compile_options(${PROJECT_NAME} PUBLIC
	$<$<CXX_COMPILER_ID:GNU>:$<BUILD_INTERFACE:-Wall>>
	$<$<CXX_COMPILER_ID:MSVC>:/MP8>
)

target_compile_options(${PROJECT_NAME} PUBLIC $<$<CONFIG:Release>:-DNVERBOSE>)

target_include_directories(${PROJECT_NAME} PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}> #<prefix>/${CMAKE_INSTALL_INCLUDEDIR}/mba
)

set_target_properties(${PROJECT_NAME} PROPERTIES
	DEBUG_POSTFIX "_d")


if(BUILD_EXAMPLES)
	add_subdirectory(app)
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

#build tree

export(TARGETS ${PROJECT_NAME}
	NAMESPACE mba::
	FILE "${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake"
)

configure_package_config_file(
	"${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in"
	"${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Config.cmake"
	INSTALL_DESTINATION "${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}"
	NO_SET_AND_CHECK_MACRO
	NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

write_basic_package_version_file(
	"${CMAKE_BINARY_DIR}/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}ConfigVersion.cmake"
	VERSION 1.3.0
	COMPATIBILITY ExactVersion
)


#install tree

install(TARGETS ${PROJECT_NAME}
	EXPORT ${PROJECT_NAME}Targets
	RUNTIME 
		DESTINATION ${CMAKE_INSTALL_BINDIR}
		COMPONENT ${PROJECT_NAME}_runtime
	LIBRARY 
		DESTINATION ${CMAKE_INSTALL_LIBDIR}
		COMPONENT ${PRJECT_NAME}_libraries
	ARCHIVE 
		DESTINATION ${CMAKE_INSTALL_LIBDIR}
		COMPONENT ${PROJECT_NAME}_libraries
)

install(EXPORT ${PROJECT_NAME}Targets
		NAMESPACE mba::
		DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
		COMPONENT ${PROJECT_NAME}_config
)

## <package_name>ConfigVersion.cmake
write_basic_package_version_file(
	"${CMAKE_BINARY_DIR}/cmake/${PROJECT_NAME}/${PROJECT_NAME}ConfigVersion.cmake"
	VERSION 1.3.0
	COMPATIBILITY ExactVersion
)

## <package_name>Config.cmake
configure_package_config_file(
	"${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in"
	"${CMAKE_BINARY_DIR}/cmake/${PROJECT_NAME}/${PROJECT_NAME}Config.cmake"
	INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
	NO_SET_AND_CHECK_MACRO
	NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

install(FILES "${CMAKE_BINARY_DIR}/cmake/${PROJECT_NAME}/${PROJECT_NAME}ConfigVersion.cmake" 
	DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
	COMPONENT ${PROJECT_NAME}_config
)

install(FILES "${CMAKE_BINARY_DIR}/cmake/${PROJECT_NAME}/${PROJECT_NAME}Config.cmake" 
	DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
	COMPONENT ${PROJECT_NAME}_config	
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
	COMPONENT ${PROJECT_NAME}_headers
	FILES_MATCHING PATTERN "*.h"
	REGEX "doxy*" EXCLUDE
)
