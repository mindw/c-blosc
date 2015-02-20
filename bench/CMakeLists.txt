# sources
set(SOURCES bench.c)


# targets
add_executable(bench ${SOURCES})
target_link_libraries(bench blosc_shared)

# tests
if(BUILD_TESTS)

    option(TEST_INCLUDE_BENCH_SINGLE_1 "Include bench single (1 thread) in the tests" ON)
    if(TEST_INCLUDE_BENCH_SINGLE_1)
        add_test(NAME test_blosclz_1
            COMMAND bench blosclz single 1
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
        if (HAVE_LZ4)
            add_test(NAME test_lz4_1
                COMMAND bench lz4     single 1
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
                )
            add_test(NAME test_lz4hc_1
                COMMAND bench lz4hc single 1
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
                )
        endif ()
        if (HAVE_SNAPPY)
            add_test(NAME test_snappy_1
                COMMAND bench snappy  single 1
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
                )
        endif()
        if (HAVE_ZLIB)
            add_test(NAME test_zlib_1
                COMMAND bench zlib    single 1
                WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
                )
        endif ()
    endif()

    option(TEST_INCLUDE_BENCH_SINGLE_N "Include bench single (multithread) in the tests" ON)
    if(TEST_INCLUDE_BENCH_SINGLE_N)
        add_test(NAME test_blosclz_n COMMAND bench blosclz single
          WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
          )
        if (HAVE_LZ4)
            add_test(NAME test_lz4_n    COMMAND bench lz4     single
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
            add_test(NAME test_lz4hc_n   COMMAND bench lz4hc   single
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
        endif ()
        if (HAVE_SNAPPY)
            add_test(NAME test_snappy_n  COMMAND bench snappy  single
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
        endif ()
        if (HAVE_ZLIB)
            add_test(NAME test_zlib_n    COMMAND bench zlib    single
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
        endif ()
    endif()

    option(TEST_INCLUDE_BENCH_SUITE "Include bench suite in the tests" OFF)
    if(TEST_INCLUDE_BENCH_SUITE)
        add_test(NAME test_blosclz COMMAND bench blosclz suite
          WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
        )
        if (HAVE_LZ4)
            add_test(NAME test_lz4     COMMAND bench lz4     suite
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
            add_test(NAME test_lz4hc   COMMAND bench lz4hc   suite
            WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
            )
        endif ()
        if (HAVE_SNAPPY)
            add_test(NAME test_snappy  COMMAND bench snappy  suite
              WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
              )
        endif ()
        if (HAVE_ZLIB)
            add_test(NAME test_zlib    COMMAND bench zlib    suite
              WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
              )
        endif ()
    endif()

    option(TEST_INCLUDE_BENCH_HARDSUITE "Include bench hardsuite in the tests" OFF)
    if(TEST_INCLUDE_BENCH_HARDSUITE)
        add_test(NAME test_hardsuite COMMAND blosc blosclz hardsuite
        WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>"
        )
    endif()

    option(TEST_INCLUDE_BENCH_EXTREMESUITE "Include bench extremesuite in the tests" OFF)
    if(TEST_INCLUDE_BENCH_EXTREMESUITE)
        add_test(NAME test_extremesuite COMMAND bench blosclz extremesuite
        WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
    endif()

    option(TEST_INCLUDE_BENCH_DEBUGSUITE "Include bench debugsuite in the tests" OFF)
    if(TEST_INCLUDE_BENCH_DEBUGSUITE)
        add_test(NAME test_debugsuite COMMAND bench debugsuite
          WORKING_DIRECTORY "$<TARGET_FILE_DIR:blosc_shared>")
    endif()
endif()
