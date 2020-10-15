cmake_minimum_required(VERSION 3.17)

project(METIS LANGUAGES C)

set(GKLIB_PATH "${CMAKE_SOURCE_DIR}/GKlib" CACHE PATH "path to GKlib")

option(IDX64 "enable 64 bit ints" OFF)
option(REAL64 "enable 64 bit floating point numbers (i.e., double)" OFF)
option(BUILD_APPS "Build applications" OFF)

include(${GKLIB_PATH}/GKlibSystem.cmake)

# METIS' custom options
if(IDX64)
	set(METIS_IDXDEF "IDXTYPEWIDTH=64")
else()
	set(METIS_IDXDEF "IDXTYPEWIDTH=32")
endif(IDX64)
if(REAL64)
	set(METIS_REALDEF "REALTYPEWIDTH=64")
else()
	set(METIS_REALDEF "REALTYPEWIDTH=32")
endif(REAL64)

file(GLOB metis_sources ./libmetis/*.c)
add_library(metis ${GKlib_sources} ${metis_sources})
target_include_directories(metis
	PUBLIC
	"$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
	"$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/libmetis>"
	"$<BUILD_INTERFACE:${GKLIB_PATH}>"
)
target_compile_definitions(metis PUBLIC "${METIS_IDXDEF}")
target_compile_definitions(metis PUBLIC "${METIS_REALDEF}")
target_link_libraries(metis PUBLIC $<$<BOOL:UNIX>:m>)

# For variables `CMAKE_INSTALL_LIBDIR` and `CMAKE_INSTALL_INCLUDEDIR`. See also
# https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html
include(GNUInstallDirs)

install(DIRECTORY "${PROJECT_SOURCE_DIR}/include/" DESTINATION "include")
install(
  TARGETS metis
  EXPORT ${PROJECT_NAME}Targets
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

# With this alias users can also this CMakeLists.txt with add_subdirectory and
# still use `METIS::metis` in their target_link_libraries calls
add_library(METIS::metis ALIAS metis)

install(
	EXPORT ${PROJECT_NAME}Targets
	DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
	NAMESPACE "${PROJECT_NAME}::"
	FILE ${PROJECT_NAME}Config.cmake
)

if(BUILD_APPS)
	add_subdirectory(programs)
endif()
