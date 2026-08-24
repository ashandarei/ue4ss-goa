include(FetchContent)

set(FETCHCONTENT_QUIET OFF)

FetchContent_MakeAvailable(corrosion)

corrosion_import_crate(MANIFEST_PATH ../../first/patternsleuth_bind/Cargo.toml)

function(_ue4ss_strip_bad_rust_link_entries target_name)
    if(NOT TARGET ${target_name})
        return()
    endif()

    foreach(prop_name IN ITEMS
        INTERFACE_LINK_LIBRARIES
        IMPORTED_LINK_INTERFACE_LIBRARIES
        IMPORTED_LINK_INTERFACE_LIBRARIES_DEBUG
        IMPORTED_LINK_INTERFACE_LIBRARIES_RELEASE
        IMPORTED_LINK_INTERFACE_LIBRARIES_RELWITHDEBINFO
        IMPORTED_LINK_INTERFACE_LIBRARIES_MINSIZEREL
    )
        get_target_property(prop_value ${target_name} ${prop_name})
        if(NOT prop_value OR prop_value STREQUAL "prop_value-NOTFOUND")
            continue()
        endif()

        set(filtered_value "")
        foreach(entry IN LISTS prop_value)
            if(entry STREQUAL "/defaultlib:msvcrt" OR entry STREQUAL "\\defaultlib:msvcrt")
                continue()
            endif()
            list(APPEND filtered_value "${entry}")
        endforeach()

        set_target_properties(${target_name} PROPERTIES ${prop_name} "${filtered_value}")
    endforeach()
endfunction()

foreach(_patternsleuth_target IN ITEMS patternsleuth_bind patternsleuth_bind-shared patternsleuth_bind-static)
    _ue4ss_strip_bad_rust_link_entries(${_patternsleuth_target})
endforeach()
