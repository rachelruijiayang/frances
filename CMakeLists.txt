cmake_minimum_required (VERSION 2.7)

project (frances)

###############################################################################
# for testing
find_package (Threads REQUIRED)
#include (CTest)
include (ExternalProject)

#ExternalProject_Add (
#	gtest
#	URL https://github.com/google/googletest/archive/master.zip
#	PREFIX ${PROJECT_SOURCE_DIR}/test/ext
#	INSTALL_COMMAND ""
#)
# Get GTest informations
#ExternalProject_Get_Property (gtest source_dir binary_dir)

#add_library (libgtest IMPORTED STATIC GLOBAL)
#add_dependencies (libgtest gtest)

# Set libgtest properties
#set_target_properties(libgtest PROPERTIES
#    "IMPORTED_LOCATION" "${binary_dir}/googlemock/gtest/libgtest.a"
#    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
#)
#include_directories("${source_dir}/googletest/include"
#                    "${source_dir}/googlemock/include")

###############################################################################

set (LIBSIGNAL_DIR "${PROJECT_SOURCE_DIR}/ext/libsignal/build/src")
set (LIBSIGNAL_INCLUDE_DIR "${LIBSIGNAL_DIR}/src/libsignal/src")

ExternalProject_Add (
	libsignal
	GIT_REPOSITORY "https://github.com/WhisperSystems/libsignal-protocol-c.git"
	GIT_TAG "master"
	PREFIX ${LIBSIGNAL_DIR}
	SOURCE_DIR ""
	BINARY_DIR libsignal
	CMAKE_ARGS -D CMAKE_C_FLAGS=-fPIC -Wswitch -Wunused-variable -Wunused-value -Wshadow -Wint-conversion -Wpointer-sign -Wprotocol -Wshorten-64-to-32
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/libsignal/src/libsignal-protocol-c.a ${LIBSIGNAL_DIR}/
)

IF(APPLE AND ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	set(common_SRCS ${common_SRCS}
		test_common_ccrypto.c
	)
ELSE()
	set(common_SRCS ${common_SRCS}
		test_common_openssl.c
	)
	include_directories(${OPENSSL_INCLUDE_DIR})
ENDIF()


include_directories (${LIBSIGNAL_INCLUDE_DIR})

set (LIBSIGNAL ${LIBSIGNAL_DIR}/libsignal-protocol-c.a)
find_library (LIBGCRYPT libgcrypt.so)
find_package (LibXml2 REQUIRED)

include_directories (${LIBXML2_INCLUDE_DIR})

set (CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/lib/DEBUG)
set (CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/lib/DEBUG) 
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/out)
set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/out)

set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -pedantic -std=c99 -fPIC")

include_directories (${PROJECT_SOURCE_DIR})

#include ("${PROJECT_SOURCE_DIR}/crypto/CMakeLists.txt")
#include ("${PROJECT_SOURCE_DIR}/store/CMakeLists.txt")
#include ("${PROJECT_SOURCE_DIR}/structs/CMakeLists.txt")
#include ("${PROJECT_SOURCE_DIR}/xmpp/CMakeLists.txt")

#add_library (
#	${CRYPTO_SRC}
#	${STORE_SRC}
#	${STRUCTS_SRC}
#	${XMPP_SRC}
#)

add_executable(frances src/frances.cpp ${LIBSIGNAL} ${LIBXML2_LIBRARIES} ${LIBGCRYPT})
add_dependencies (frances libsignal)
target_link_libraries (frances "${LIBSIGNAL}" "${LIBXML2_LIBRARIES}"
	"${LIBGCRYPT}")
