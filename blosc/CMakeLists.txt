# a simple way to detect that we are using CMAKE
add_definitions(-DUSING_CMAKE)

set(INTERNAL_LIBS ${CMAKE_SOURCE_DIR}/internal-complibs)

# Hide symbols by default unless they're specifically exported.
# This makes it easier to keep the set of exported symbols the
# same across all compilers/platforms.
set(CMAKE_C_VISIBILITY_PRESET hidden)

# includes
if(NOT DEACTIVATE_LZ4)
    if (LZ4_FOUND)
        include_directories( ${LZ4_INCLUDE_DIR} )
    else()
        set(LZ4_LOCAL_DIR ${INTERNAL_LIBS}/lz4-1.7.2)
        include_directories( ${LZ4_LOCAL_DIR} )
    endif()
endif()

if(NOT DEACTIVATE_SNAPPY)
    if (SNAPPY_FOUND)
        include_directories( ${SNAPPY_INCLUDE_DIR} )
    else()
        set(SNAPPY_LOCAL_DIR ${INTERNAL_LIBS}/snappy-1.1.1)
        include_directories( ${SNAPPY_LOCAL_DIR} )
    endif()
endif()

if(NOT DEACTIVATE_ZLIB)
    if (ZLIB_FOUND)
        include_directories( ${ZLIB_INCLUDE_DIR} )
    else()
        set(ZLIB_LOCAL_DIR ${INTERNAL_LIBS}/zlib-1.2.8)
        include_directories( ${ZLIB_LOCAL_DIR} )
    endif()
endif()

# library sources
set(SOURCES blosc.h blosc.c blosclz.h blosclz.c shuffle-common.h shuffle-generic.c shuffle-generic.h bitshuffle-generic.c bitshuffle-generic.h)
if(COMPILER_SUPPORT_SSE2)
    message(STATUS "Adding run-time support for SSE2")
    set(SOURCES ${SOURCES} shuffle-sse2.c shuffle-sse2.h bitshuffle-sse2.c bitshuffle-sse2.h)
endif()
if(COMPILER_SUPPORT_AVX2)
    message(STATUS "Adding run-time support for AVX2")
    set(SOURCES ${SOURCES} shuffle-avx2.c shuffle-avx2.h bitshuffle-avx2.c bitshuffle-avx2.h)
endif()
set(SOURCES ${SOURCES} shuffle.h shuffle.c)

# library install directory
set(lib_dir lib${LIB_SUFFIX})
set(version_string ${BLOSC_VERSION_MAJOR}.${BLOSC_VERSION_MINOR}.${BLOSC_VERSION_PATCH})

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
if(WIN32)
    # try to use the system library
    find_package(Threads)
    if(NOT Threads_FOUND)
        message(STATUS "using the internal pthread library for win32 systems.")
        set(SOURCES ${SOURCES} win32/pthread.c win32/pthread.h  win32/stdint-windows.h)
    else()
        set(LIBS ${LIBS} ${CMAKE_THREAD_LIBS_INIT})
    endif()
else()
    find_package(Threads REQUIRED)
    set(LIBS ${LIBS} ${CMAKE_THREAD_LIBS_INIT})
endif()

if(NOT DEACTIVATE_LZ4)
    if(LZ4_FOUND)
        set(LIBS ${LIBS} ${LZ4_LIBRARY})
    else()
        file(GLOB LZ4_FILES ${LZ4_LOCAL_DIR}/*.*)
        set(SOURCES ${SOURCES} ${LZ4_FILES})
    endif()
endif()

if(NOT DEACTIVATE_SNAPPY)
    if(SNAPPY_FOUND)
        set(LIBS ${LIBS} ${SNAPPY_LIBRARY})
    else()
        file(GLOB SNAPPY_FILES ${SNAPPY_LOCAL_DIR}/*.cc)
        set(SOURCES ${SOURCES} ${SNAPPY_FILES})
        file(GLOB SNAPPY_FILES ${SNAPPY_LOCAL_DIR}/*.h)
        set(SOURCES ${SOURCES} ${SNAPPY_FILES})
    endif()
endif()

if(NOT DEACTIVATE_ZLIB)
    if(ZLIB_FOUND)
        set(LIBS ${LIBS} ${ZLIB_LIBRARY})
    else()
        file(GLOB ZLIB_FILES ${ZLIB_LOCAL_DIR}/*.c)
        set(SOURCES ${SOURCES} ${ZLIB_FILES})
    endif()
endif()

if (MSVC)
    if (NOT DEFINED ARCH)
        string(TOLOWER ${MSVC_C_ARCHITECTURE_ID} ARCH)
    endif()

    # use boost convention - you cant' get the VC compiler without VS
    if (NOT DEFINED MSVCVER)
        math(EXPR MSVCVER "(${MSVC_VERSION} / 10) - 60")
    endif()
    
    # add the runtime version and target architecture into the DLL name per MS
    # guidelines
    set(MSVC_SHARED_SUFFIX_BASE "-vc${MSVCVER}-mt-${ARCH}")
    set(CMAKE_SHARED_LIBRARY_SUFFIX "${MSVC_SHARED_SUFFIX_BASE}.dll")
    
    set(VER_PRODUCTNAME_STR "Blosc: A blocking, shuffling and lossless compression library.")
    set(VER_LEGALCOPYRIGHT_STR "Copyright (C) 2009-2015 Francesc Alted <francesc@blosc.org>.")
    set(VER_STR "${BLOSC_VERSION_STRING}")
    set(VER_NUM "${BLOSC_VERSION_MAJOR},${BLOSC_VERSION_MINOR},${BLOSC_VERSION_PATCH},0")
    
    if (MSVCVER LESS "120")
        file(COPY "win32/stdbool.h" DESTINATION "${CMAKE_BINARY_DIR}")
    endif()
endif()

function(add_win32_rc target)

    get_target_property(VER_INTERNALNAME_STR ${target} OUTPUT_NAME)
    if (VER_INTERNALNAME_STR MATCHES "VER_INTERNALNAME_STR-NOTFOUND")
        set(VER_INTERNALNAME_STR "${target}")
    endif()
    get_target_property(VER_FILEDESCRIPTION_STR ${target} DESCRIPTION)

    get_target_property(type ${target} TYPE)
    if (${type} STREQUAL "EXECUTABLE")
        set(VER_FILETYPE "VFT_APP")
        set(VER_ORIGINALFILENAME_STR "${VER_INTERNALNAME_STR}${CMAKE_EXECUTABLE_SUFFIX}")
    elseif (${type} STREQUAL "SHARED_LIBRARY")
        set(VER_FILETYPE "VFT_DLL")
        set(VER_ORIGINALFILENAME_STR "${CMAKE_SHARED_LIBRARY_PREFIX}${VER_INTERNALNAME_STR}${CMAKE_SHARED_LIBRARY_SUFFIX}")
    endif()

    set(OUT_FNAME "${CMAKE_CURRENT_BINARY_DIR}/${target}.rc")
    configure_file(
         ${CMAKE_SOURCE_DIR}/blosc/win32/version.rc.cmakein
         ${OUT_FNAME}
         @ONLY
     )         

    target_sources(${target} PRIVATE ${OUT_FNAME})

endfunction(add_win32_rc)

# targets
add_library(blosc_shared SHARED ${SOURCES})
set_target_properties(blosc_shared PROPERTIES OUTPUT_NAME blosc)

if(MSVC)
    set_target_properties(blosc_shared PROPERTIES ARCHIVE_OUTPUT_NAME blosc)
    target_sources(blosc_shared PRIVATE win32/blosc.def)
    set_property(TARGET blosc_shared PROPERTY DESCRIPTION "blosc compression dynamic link library")
    add_win32_rc(blosc_shared)
endif()

set_target_properties(blosc_shared PROPERTIES
        VERSION ${version_string}
        SOVERSION 1  # Change this when an ABI change happens
    )
set_property(
    TARGET blosc_shared
    APPEND PROPERTY COMPILE_DEFINITIONS BLOSC_SHARED_LIBRARY)

# Based on the target architecture and hardware features supported
# by the C compiler, set hardware architecture optimization flags
# for specific shuffle implementations.
if(COMPILER_SUPPORT_SSE2)
    if (MSVC)
        # MSVC targets SSE2 by default on 64-bit configurations, but not 32-bit configurations.
        if (${CMAKE_SIZEOF_VOID_P} EQUAL 4)
            set_source_files_properties(shuffle-sse2.c bitshuffle-sse2.c PROPERTIES COMPILE_FLAGS "/arch:SSSE3")
        endif()
    else ()
        set_source_files_properties(shuffle-sse2.c bitshuffle-sse2.c PROPERTIES COMPILE_FLAGS -msse2)
    endif ()

    # Define a symbol for the shuffle-dispatch implementation
    # so it knows SSE2 is supported even though that file is
    # compiled without SSE2 support (for portability).
    set_property(
        SOURCE shuffle.c
        APPEND PROPERTY COMPILE_DEFINITIONS SHUFFLE_SSE2_ENABLED)
endif()
if(COMPILER_SUPPORT_AVX2)
    if (MSVC)
        set_source_files_properties(shuffle-avx2.c bitshuffle-avx2.c PROPERTIES COMPILE_FLAGS "/Qax:CORE-AVX2")
    else()
        set_source_files_properties(shuffle-avx2.c bitshuffle-avx2.c PROPERTIES COMPILE_FLAGS -mavx2)
    endif()

    # Define a symbol for the shuffle-dispatch implementation
    # so it knows AVX2 is supported even though that file is
    # compiled without AVX2 support (for portability).
    set_property(
        SOURCE shuffle.c
        APPEND PROPERTY COMPILE_DEFINITIONS SHUFFLE_AVX2_ENABLED)
endif()

target_link_libraries(blosc_shared ${LIBS})

if(BUILD_STATIC)
    add_library(blosc_static STATIC ${SOURCES})
	if (NOT MSVC)
        set_target_properties(blosc_static PROPERTIES OUTPUT_NAME blosc)
	endif()

    target_link_libraries(blosc_static ${LIBS})
endif()


# install
install(FILES blosc.h blosc-export.h DESTINATION include COMPONENT DEV)
install(TARGETS blosc_shared
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)

if(BUILD_STATIC)
    install(TARGETS blosc_static DESTINATION ${lib_dir} COMPONENT DEV)
endif()
