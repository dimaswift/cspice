fips_begin_lib(cspice)

fips_files(cspice.c)

target_compile_definitions(cspice PUBLIC USE_CSPICE=1)
set(CSPICE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
target_include_directories(cspice PUBLIC $ENV{CSPICE_DIR}/include)
target_link_directories(cspice PUBLIC $ENV{CSPICE_DIR}/lib)

if (FIPS_WINDOWS)
    target_link_libraries(cspice PUBLIC
            "${CSPICE_DIR}/lib/cspice.lib"
            "${CSPICE_DIR}/lib/csupport.lib"
    )
else ()
    target_link_libraries(cspice PUBLIC
            "${CSPICE_DIR}/lib/cspice.a"
            "${CSPICE_DIR}/lib/csupport.a"
    )
endif ()

fips_end_lib()

macro(cspice_fetch_kernels)

    set(KERNELS_DIR "${CMAKE_SOURCE_DIR}/kernels")
    file(MAKE_DIRECTORY "${KERNELS_DIR}")

    function(fetch_kernel filename url)
        set(dst "${KERNELS_DIR}/${filename}")
        if (NOT EXISTS "${dst}")
            message(STATUS "Fetching kernel: ${filename}")
            file(DOWNLOAD
                    "${url}" "${dst}"
                    SHOW_PROGRESS
                    TLS_VERIFY ON
                    STATUS st
            )
            list(GET st 0 code)
            list(GET st 1 msg)
            if (NOT code EQUAL 0)
                file(REMOVE "${dst}")
                message(FATAL_ERROR "Download failed for ${filename}: ${msg}")
            endif()
        endif()
    endfunction()

    fetch_kernel("naif0012.tls"
            "https://naif.jpl.nasa.gov/pub/naif/generic_kernels/lsk/naif0012.tls"
    )

    fetch_kernel("de442s.bsp"
            "https://naif.jpl.nasa.gov/pub/naif/generic_kernels/spk/planets/de442s.bsp"
    )


    fetch_kernel("pck00011.tpc"
            "https://naif.jpl.nasa.gov/pub/naif/generic_kernels/pck/pck00011.tpc"
    )

endmacro()


macro(cspice_copy_kernels)
    if (FIPS_WINDOWS OR (FIPS_OSX AND NOT FIPS_IOS) OR FIPS_LINUX)
        add_custom_command(TARGET main POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy_directory
                "${CMAKE_SOURCE_DIR}/kernels"
                "$<TARGET_FILE_DIR:main>/kernels"
        )
    endif()
endmacro()

