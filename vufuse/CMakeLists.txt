cmake_minimum_required(VERSION 2.6)

find_library(FUSEEXT
	NAMES libfuseext2.so
	NAMES umfuseext2.so
	PATHS ${CMAKE_INSTALL_PREFIX}/lib/fuse-ext2
	PATHS ${CMAKE_INSTALL_PREFIX}/lib/umview/modules
)
if(FUSEEXT)
	message(STATUS "fuse-ext2: ${FUSEEXT}")
else()
	message(FATAL_ERROR "fuse-ext2 not found")
endif()

# get api version
execute_process(
	COMMAND bash "-c" "fuse-ext2 -V | grep -m 1 -oE '\\s[0-9]{2}\\s' -"
	OUTPUT_VARIABLE FUSE_USE_VERSION
)
string(STRIP "${FUSE_USE_VERSION}" FUSE_USE_VERSION)	# remove space, tabs...
message(STATUS "fuse-ext2 api level: ${FUSE_USE_VERSION}")

# if FUSE_USE_VERSION > DEVELOPED_FUSE_VERSION set DEVELOPED_FUSE_VERSION as default
set(DEVELOPED_FUSE_VERSION 32)
if("${FUSE_USE_VERSION}" VERSION_GREATER "${DEVELOPED_FUSE_VERSION}")
	message(DEPRECATION "api level mismatch: using ${DEVELOPED_FUSE_VERSION}")
	set(FUSE_USE_VERSION ${DEVELOPED_FUSE_VERSION})
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FILE_OFFSET_BITS=64 -DFUSE_USE_VERSION=${FUSE_USE_VERSION}")

include_directories(${VU_HEADERS} ${CMAKE_CURRENT_SOURCE_DIR})

set(VU_MOD_TARGET vufuse)
file(GLOB_RECURSE VUFUSE_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.c)

add_library(${VU_MOD_TARGET} SHARED ${VUFUSE_SOURCES})
set_target_properties(${VU_MOD_TARGET} PROPERTIES PREFIX "")
target_link_libraries(${VU_MOD_TARGET} stropt volatilestream execs)

ADD_CUSTOM_TARGET(link_target ALL COMMAND ${CMAKE_COMMAND} -E create_symlink ${FUSEEXT} ${CMAKE_CURRENT_SOURCE_DIR}/vufuseext2.so)

install(TARGETS ${VU_MOD_TARGET} LIBRARY DESTINATION ${MODULES_INSTALL_PATH})

# ~ install(FILES ${FUSEEXT} DESTINATION ${MODULES_INSTALL_PATH} RENAME vufuseext2.so)
install(FILES vufuseext2.so DESTINATION ${MODULES_INSTALL_PATH})
