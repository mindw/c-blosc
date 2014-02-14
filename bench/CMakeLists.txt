# sources
set(SOURCES bench.c)


# targets
add_executable(bench ${SOURCES})
if(UNIX AND NOT APPLE)
  # cmake is complaining about LINK_PRIVATE in original PR
  # and removing it does not seem to hurt, so be it.
  # target_link_libraries(bench LINK_PRIVATE rt)
  target_link_libraries(bench rt)
endif()
target_link_libraries(bench blosc_shared)

# have to copy blosc dlls for mingw
# tests
if(BUILD_TESTS)

    # The commented tests below take too much time to complete
    option(TEST_INCLUDE_BENCH_SHUFFLE_1 "Include bench shuffle (1 thread) in the tests" ON)
    if(TEST_INCLUDE_BENCH_SHUFFLE_1)
        set(SHUFFLE_1_OPTS shuffle test 1)
        add_test(NAME test_blosclz_shuffle_1 COMMAND bench blosclz ${SHUFFLE_1_OPTS}
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        if (HAVE_LZ4)
            add_test(NAME test_lz4_shuffle_1 COMMAND bench lz4 ${SHUFFLE_1_OPTS}
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
            # add_test(NAME test_lz4hc_shuffle_1 COMMAND bench lz4hc ${SHUFFLE_1_OPTS}
                # WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
        if (HAVE_SNAPPY)
            add_test(NAME test_snappy_shuffle_1 
                COMMAND bench snappy ${SHUFFLE_1_OPTS}
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif()
        if (HAVE_ZLIB)
            # add_test(NAME test_zlib_shuffle_1 COMMAND bench zlib ${SHUFFLE_1_OPTS} 
                # WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif()
    endif()

    option(TEST_INCLUDE_BENCH_SHUFFLE_N "Include bench shuffle (multithread) in the tests" ON)
    if(TEST_INCLUDE_BENCH_SHUFFLE_N)
        set(SHUFFLE_N_OPTS shuffle test)
        add_test(NAME test_blosclz_shuffle_n COMMAND bench blosclz ${SHUFFLE_N_OPTS}
          WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        if (HAVE_LZ4)
            add_test(NAME test_lz4_shuffle_n COMMAND bench lz4 ${SHUFFLE_N_OPTS}
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
            add_test(NAME test_lz4hc_shuffle_n COMMAND bench lz4hc ${SHUFFLE_N_OPTS}
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
        if (HAVE_SNAPPY)
            add_test(NAME test_snappy_n COMMAND bench snappy ${SHUFFLE_N_OPTS}
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
        if (HAVE_ZLIB)
            add_test(NAME test_zlib_shuffle_n COMMAND bench zlib ${SHUFFLE_N_OPTS}
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
    endif()

    option(TEST_INCLUDE_BENCH_BITSHUFFLE_1 "Include bench bitshuffle (1 thread) in the tests" ON)
    if(TEST_INCLUDE_BENCH_BITSHUFFLE_1)
        set(BITSHUFFLE_1_OPTS bitshuffle test 1)
        add_test(NAME test_blosclz_bitshuffle_1 COMMAND bench blosclz ${BITSHUFFLE_1_OPTS}
          WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        if (HAVE_LZ4)
            add_test(NAME test_lz4_bitshuffle_1 COMMAND bench lz4 ${BITSHUFFLE_1_OPTS}
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
            # add_test(NAME test_lz4hc_bitshuffle_1 COMMAND bench lz4hc ${BITSHUFFLE_1_OPTS}
            #   WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
        if (HAVE_SNAPPY)
            add_test(NAME test_snappy_bitshuffle_1 COMMAND bench snappy ${BITSHUFFLE_1_OPTS}
              WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
        if (HAVE_ZLIB)
            # add_test(NAME test_zlib_bitshuffle_1 COMMAND bench zlib ${BITSHUFFLE_1_OPTS}
              # WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
    endif()

    option(TEST_INCLUDE_BENCH_BITSHUFFLE_N "Include bench bitshuffle (multithread) in the tests" ON)
    if(TEST_INCLUDE_BENCH_BITSHUFFLE_N)
        set(BITSHUFFLE_N_OPTS bitshuffle test)
        add_test(NAME test_blosclz_bitshuffle_n COMMAND bench blosclz ${BITSHUFFLE_N_OPTS}
              WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        
        if (HAVE_LZ4)
            add_test(NAME test_lz4_bitshuffle_n COMMAND bench lz4 ${BITSHUFFLE_N_OPTS}
              WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
            # add_test(NAME test_lz4hc_bitshuffle_n COMMAND bench lz4hc ${BITSHUFFLE_N_OPTS})
              # WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
        if (HAVE_SNAPPY)
            add_test(NAME test_snappy_bitshuffle_n COMMAND bench snappy ${BITSHUFFLE_N_OPTS}
              WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
        if (HAVE_ZLIB)
            # add_test(NAME test_zlib_bitshuffle_n COMMAND bench zlib ${BITSHUFFLE_N_OPTS}
            #   WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
        endif ()
    endif()

    option(TEST_INCLUDE_BENCH_SUITE "Include bench suite in the tests" OFF)
    if(TEST_INCLUDE_BENCH_SUITE)
        add_test(NAME test_hardsuite COMMAND blosc blosclz shuffle suite
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
    endif()

    option(TEST_INCLUDE_BENCH_DEBUGSUITE "Include bench debugsuite in the tests" OFF)
    if(TEST_INCLUDE_BENCH_DEBUGSUITE)
        add_test(NAME test_debugsuite COMMAND bench blosclz shuffle debugsuite
          WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
    endif()
endif()
