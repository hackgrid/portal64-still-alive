####################
## Assembly files ##
####################

set(ASM_FILES
    boot.s
    entry.s
    rom_header.s
    sound_data.s
)

add_library(asm OBJECT)
target_sources(asm PRIVATE
    ${ASM_FILES}
)
target_link_libraries(asm
    platform
)

# CMake doesn't track .incbin file dependencies, so some manual work is needed
add_dependencies(asm
    sound_data_tables
)
get_target_property(SOUND_DATA_TABLE_FILES sound_data_tables OUTPUTS)
set_source_files_properties(sound_data.s PROPERTIES
    OBJECT_DEPENDS "${SOUND_DATA_TABLE_FILES}"
)
