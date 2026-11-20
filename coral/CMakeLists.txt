cmake_minimum_required(VERSION 3.19)
project(z)

set(OUR_DIR ${CMAKE_CURRENT_SOURCE_DIR}/..)

# zlib is tracked as the `zlib` git submodule of this repository instead of
# being vendored in-tree. Fetch it once after cloning:
#   git submodule update --init --recursive
# A parent project that keeps the zlib sources elsewhere may preset ZLIB_DIR
# before adding this directory.
if (NOT ZLIB_DIR)
    set(ZLIB_DIR ${OUR_DIR}/zlib)
endif()

set(TARGET              z)
set(SRC_DIR             ${ZLIB_DIR})
set(PUBLIC_HDR_DIR      ${ZLIB_DIR})
set(PRIVATE_HDR_DIR     ${ZLIB_DIR})

if (NOT EXISTS "${SRC_DIR}/zlib.h")
    message(FATAL_ERROR
            "zlib sources are missing at ${SRC_DIR}.\n"
            "Initialize the submodule with: git submodule update --init --recursive")
endif()

# public headers dependencies
set(PUBLIC_HDRS
    ${PUBLIC_HDR_DIR}/zconf.h
    ${PUBLIC_HDR_DIR}/zlib.h
)

# private headers dependencies
set(PRIVATE_HDRS
    ${PRIVATE_HDR_DIR}/crc32.h
    ${PRIVATE_HDR_DIR}/deflate.h
    ${PRIVATE_HDR_DIR}/gzguts.h
    ${PRIVATE_HDR_DIR}/inffast.h
    ${PRIVATE_HDR_DIR}/inffixed.h
    ${PRIVATE_HDR_DIR}/inflate.h
    ${PRIVATE_HDR_DIR}/inftrees.h
    ${PRIVATE_HDR_DIR}/trees.h
    ${PRIVATE_HDR_DIR}/zutil.h
)

# sources dependencies
set(SRCS
    ${SRC_DIR}/adler32.c
    ${SRC_DIR}/compress.c
    ${SRC_DIR}/crc32.c
    ${SRC_DIR}/deflate.c
    ${SRC_DIR}/gzclose.c
    ${SRC_DIR}/gzlib.c
    ${SRC_DIR}/gzread.c
    ${SRC_DIR}/gzwrite.c
    ${SRC_DIR}/inflate.c
    ${SRC_DIR}/infback.c
    ${SRC_DIR}/inftrees.c
    ${SRC_DIR}/inffast.c
    ${SRC_DIR}/trees.c
    ${SRC_DIR}/uncompr.c
    ${SRC_DIR}/zutil.c
)

# we're building a library
add_library(${TARGET} STATIC ${PRIVATE_HDRS} ${PUBLIC_HDRS} ${SRCS})

if (NOT MSVC)
    set(TARGET_FLAGS
            -Wno-implicit-function-declaration
            -Wno-shift-negative-value
    )

    # -Wdeprecated-non-prototype is a new warning introduced in Clang 15.
    # https://www.redhat.com/en/blog/new-warnings-and-errors-clang-15
    if (CMAKE_C_COMPILER_ID MATCHES "Clang" AND CMAKE_C_COMPILER_VERSION VERSION_GREATER_EQUAL "15.0")
        list(APPEND TARGET_FLAGS -Wno-deprecated-non-prototype) # See https://github.com/madler/zlib/issues/633
    endif()

    target_compile_options(${TARGET} PRIVATE ${TARGET_FLAGS})
endif()

# specify where the public headers of this library are
target_include_directories (${TARGET} PUBLIC ${PUBLIC_HDR_DIR})
set_target_properties(${TARGET} PROPERTIES FOLDER ThirdParty)
