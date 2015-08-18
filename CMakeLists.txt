cmake_minimum_required(VERSION 2.8)
project(bsd-xdr C)

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
add_library(bsd-xdr STATIC lib/xdr.c lib/xdr_array.c lib/xdr_float.c lib/xdr_mem.c lib/xdr_private.c lib/xdr_private.h lib/xdr_rec.c lib/xdr_reference.c lib/xdr_sizeof.c lib/xdr_stdio.c rpc/types.h rpc/xdr.h)


file(GLOB TEST_SOURCES src/test/*_test.c)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/test)

foreach(TEST_FILE ${TEST_SOURCES})
	get_filename_component(TEST_NAME ${TEST_FILE} NAME_WE)
	add_executable(${TEST_NAME} ${TEST_FILE} src/test/test_common.c src/test/test_data.c src/test/test_xdrs.c)
	target_link_libraries(${TEST_NAME} bsd-xdr)
endforeach()
