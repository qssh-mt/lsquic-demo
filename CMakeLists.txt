CMAKE_MINIMUM_REQUIRED(VERSION 3.5)
PROJECT(lsquic-demo C)

IF(CMAKE_BUILD_TYPE STREQUAL "")
    SET(CMAKE_BUILD_TYPE Debug)
ENDIF()
MESSAGE(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
IF (CMAKE_BUILD_TYPE STREQUAL Debug)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O0")
ELSEIF (CMAKE_BUILD_TYPE STREQUAL Debug)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2")
ENDIF()
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} $ENV{EXTRA_CFLAGS}")

MESSAGE(STATUS "Compiler flags: ${CMAKE_C_FLAGS}")

ADD_SUBDIRECTORY(boringssl)

INCLUDE_DIRECTORIES(boringssl/include)
INCLUDE_DIRECTORIES(lsquic/include)
INCLUDE_DIRECTORIES(lsquic/src/liblsquic)       # For lsquic_xxhash.h
ADD_SUBDIRECTORY(lsquic/src/liblsquic)

INCLUDE_DIRECTORIES(libev)
add_executable(client client.c net.c net.h lsquic_utils.c lsquic_utils.h)
add_executable(server server.c net.c net.h lsquic_utils.c lsquic_utils.h)

SET(LIBS lsquic crypto ssl z m ev)
IF(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    # If using older glibc, need to link with -lrt.  See clock_getres(2).
    EXECUTE_PROCESS(
            COMMAND ${PROJECT_SOURCE_DIR}/print-glibc-version.sh ${CMAKE_C_COMPILER}
            OUTPUT_VARIABLE GLIBC_VERSION)
    IF(NOT GLIBC_VERSION EQUAL "" AND GLIBC_VERSION VERSION_LESS 2.17)
        SET(LIBS ${LIBS} rt)
    ENDIF()
ENDIF()
TARGET_LINK_LIBRARIES(client ${LIBS})
TARGET_LINK_LIBRARIES(server ${LIBS})
