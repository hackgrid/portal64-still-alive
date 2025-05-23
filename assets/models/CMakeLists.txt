set(GEN_DYNAMIC_MODEL_LIST      "${PROJECT_SOURCE_DIR}/tools/models/generate_dynamic_model_list.js")
set(GEN_DYNAMIC_ANIM_MODEL_LIST "${PROJECT_SOURCE_DIR}/tools/models/generate_dynamic_animated_model_list.js")

cmake_path(
    RELATIVE_PATH CMAKE_CURRENT_SOURCE_DIR
    BASE_DIRECTORY "${PROJECT_SOURCE_DIR}"
    OUTPUT_VARIABLE RELATIVE_CURRENT_DIR
)

##################
## Model export ##
##################

# Models whose data is always loaded
set(STATIC_MODELS
    fleck_ash2
    grav_flare
    player/chell
    portal_gun/ball_trail
    portal_gun/v_portalgun
    portal_gun/w_portalgun
    portal/portal_blue
    portal/portal_blue_face
    portal/portal_blue_filled
    portal/portal_collider
    portal/portal_collider_vertical
    portal/portal_orange
    portal/portal_orange_face
    portal/portal_orange_filled
    props/round_elevator
    props/round_elevator_collision
    props/round_elevator_interior
    props/signage
)

# Models whose data is loaded/unloaded as needed
set(DYNAMIC_MODELS
    cube/cube
    pedestal
    props/autoportal_frame/autoportal_frame
    props/box_dropper
    props/box_dropper_glass
    props/button
    props/combine_ball_catcher
    props/combine_ball_launcher
    props/cylinder_test
    props/door_01
    props/door_02
    props/lab_chair
    props/lab_desk/lab_desk01
    props/lab_desk/lab_desk02
    props/lab_desk/lab_desk03
    props/lab_desk/lab_desk04
    props/lab_monitor
    props/light_rail_endcap
    props/portal_cleanser
    props/radio
    props/security_camera
    props/switch001
    props/turret_01
    signage/clock
    signage/clock_digits
)

# Models with predefined animations
set(KEYFRAMED_ANIMATED_MODELS
    pedestal
    player/chell
    portal_gun/v_portalgun
    props/box_dropper
    props/combine_ball_catcher
    props/combine_ball_launcher
    props/door_01
    props/door_02
    props/switch001
    props/turret_01
)

# All animated models, including those which are only procedurally animated
set(ANIMATED_MODELS
    ${KEYFRAMED_ANIMATED_MODELS}
    props/button
    props/security_camera
)

# All models
set(MODELS
    ${DYNAMIC_MODELS}
    ${STATIC_MODELS}
)

# Most models depend on both of these. Some only depend on one or the other.
# Use both here for simplicity. Otherwise there would be many special cases.
set(MODEL_DEFAULT_MATERIALS
    ${ASSETS_DIR}/materials/static.skm.yaml
    ${ASSETS_DIR}/materials/objects.skm.yaml
)

function(_add_model_export_command MODEL_NAME OUTPUT_VARIABLE)
    set(INPUT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/${MODEL_NAME}.blend")
    set(OUTPUT_FILE "${PROJECT_BINARY_DIR}/${RELATIVE_CURRENT_DIR}/${MODEL_NAME}.fbx")

    add_custom_command(
        DEPENDS
            ${EXPORT_FBX} ${INPUT_FILE}
        OUTPUT
            ${OUTPUT_FILE}
        COMMAND
           ${CMAKE_COMMAND} -E make_directory "$<PATH:GET_PARENT_PATH,${OUTPUT_FILE}>"
        COMMAND
            ${Blender_EXECUTABLE}
            ${INPUT_FILE} --background --python ${EXPORT_FBX} -- ${OUTPUT_FILE}
        COMMENT
            "Exporting $<PATH:RELATIVE_PATH,${INPUT_FILE},${PROJECT_SOURCE_DIR}>"
        VERBATIM
    )

    set(${OUTPUT_VARIABLE} ${OUTPUT_FILE})
    return(PROPAGATE ${OUTPUT_VARIABLE})
endfunction()

function(_add_model_generate_command MODEL_NAME MODEL_FBX MODEL_OUTPUT_LIST ANIM_OUTPUT_LIST)
    set(MODEL_FLAGS "${CMAKE_CURRENT_SOURCE_DIR}/${MODEL_NAME}.flags")
    set(MODEL_MATERIALS ${MODEL_DEFAULT_MATERIALS})

    get_source_file_property(FILE_MATERIALS ${MODEL_FLAGS} ADDITIONAL_MATERIALS)
    if (FILE_MATERIALS)
        list(APPEND MODEL_MATERIALS ${FILE_MATERIALS})
    endif()

    set(OUTPUT_FILE_NO_EXTENSION "${GENERATED_CODE_DIR}/${RELATIVE_CURRENT_DIR}/${MODEL_NAME}")
    set(OUTPUT_FILE_H "${OUTPUT_FILE_NO_EXTENSION}.h")
    set(OUTPUT_FILE_GEO_C "${OUTPUT_FILE_NO_EXTENSION}_geo.c")
    set(MODEL_OUTPUT_FILES
        ${OUTPUT_FILE_H}
        ${OUTPUT_FILE_GEO_C}
    )
    set(ANIM_OUTPUT_FILES "")

    if (MODEL_NAME IN_LIST KEYFRAMED_ANIMATED_MODELS)
        set(OUTPUT_FILE_ANIM_C "${OUTPUT_FILE_NO_EXTENSION}_anim.c")
        list(APPEND ANIM_OUTPUT_FILES ${OUTPUT_FILE_ANIM_C})
    endif()

    add_custom_command(
        DEPENDS
            textures "$<TARGET_PROPERTY:textures,OUTPUTS>"
            skeletool64::skeletool64
            ${RUN_COMMAND}
            ${MODEL_FBX}
            ${MODEL_FLAGS}
            ${MODEL_MATERIALS}
        OUTPUT
            ${MODEL_OUTPUT_FILES}
            ${ANIM_OUTPUT_FILES}
        COMMAND
            ${CMAKE_COMMAND} -E make_directory "$<PATH:GET_PARENT_PATH,${OUTPUT_FILE_NO_EXTENSION}>"
        COMMAND
            ${Python3_EXECUTABLE} ${RUN_COMMAND}
            $<TARGET_FILE:skeletool64::skeletool64>
            ${MODEL_FBX}
            --fixed-point-scale ${SCENE_SCALE}
            --model-scale ${MODEL_SCALE}
            --name ${MODEL_NAME}
            --output ${OUTPUT_FILE_H}
            @${MODEL_FLAGS}
        WORKING_DIRECTORY
            ${PROJECT_SOURCE_DIR}
        COMMENT
            "Generating model ${MODEL_NAME}"
        VERBATIM
    )

    list(APPEND ${MODEL_OUTPUT_LIST} ${MODEL_OUTPUT_FILES})
    list(APPEND ${ANIM_OUTPUT_LIST} ${ANIM_OUTPUT_FILES})
    return(PROPAGATE ${MODEL_OUTPUT_LIST} ${ANIM_OUTPUT_LIST})
endfunction()

function(_add_model_lib LIB_NAME LIB_SOURCES)
    add_library(${LIB_NAME} OBJECT)
    add_dependencies(${LIB_NAME}
        materials
        models
    )
    target_sources(${LIB_NAME} PRIVATE
        ${LIB_SOURCES}
    )
    target_link_libraries(${LIB_NAME}
        platform
    )
endfunction()

# Add commands for exporting model files
# There are some special cases for models which use non-default materials

set_property(SOURCE player/chell.flags
    PROPERTY ADDITIONAL_MATERIALS
        ${ASSETS_DIR}/materials/chell.skm.yaml
)

set_property(SOURCE props/combine_ball_launcher.flags
    PROPERTY ADDITIONAL_MATERIALS
        ${ASSETS_DIR}/materials/ball_catcher.skm.yaml
)

set_property(SOURCE props/combine_ball_catcher.flags
    PROPERTY ADDITIONAL_MATERIALS
        ${ASSETS_DIR}/materials/ball_catcher.skm.yaml
)

set_property(SOURCE props/round_elevator.flags
    PROPERTY ADDITIONAL_MATERIALS
        ${ASSETS_DIR}/materials/elevator.skm.yaml
)

set_property(SOURCE props/round_elevator_interior.flags
    PROPERTY ADDITIONAL_MATERIALS
        ${ASSETS_DIR}/materials/elevator.skm.yaml
)

set(STATIC_MODEL_GENERATED_FILES "")
set(DYNAMIC_MODEL_GENERATED_FILES "")
set(ANIM_GENERATED_FILES "")

foreach(MODEL ${MODELS})
    set(MODEL_GENERATED_FILES "")

    _add_model_export_command(${MODEL} MODEL_FBX)
    _add_model_generate_command(${MODEL} ${MODEL_FBX}
        MODEL_GENERATED_FILES
        ANIM_GENERATED_FILES
    )

    if (MODEL IN_LIST DYNAMIC_MODELS)
        list(APPEND DYNAMIC_MODEL_GENERATED_FILES ${MODEL_GENERATED_FILES})
    else()
        list(APPEND STATIC_MODEL_GENERATED_FILES ${MODEL_GENERATED_FILES})
    endif()
endforeach()

add_custom_target(models
    DEPENDS
        ${STATIC_MODEL_GENERATED_FILES}
        ${DYNAMIC_MODEL_GENERATED_FILES}
        ${ANIM_GENERATED_FILES}
)

_add_model_lib(models_static    "${STATIC_MODEL_GENERATED_FILES}")
_add_model_lib(models_dynamic   "${DYNAMIC_MODEL_GENERATED_FILES}")
_add_model_lib(model_animations "${ANIM_GENERATED_FILES}")

# Add commands for generating dynamic model lists

set(DYNAMIC_MODEL_GENERATED_HEADERS ${DYNAMIC_MODEL_GENERATED_FILES})
list(FILTER
    DYNAMIC_MODEL_GENERATED_HEADERS
    INCLUDE REGEX "\.h$"
)

set(DYNAMIC_NON_ANIMATED_MODEL_HEADERS "")
set(DYNAMIC_ANIMATED_MODEL_HEADERS "")

foreach(GENERATED_HEADER ${DYNAMIC_MODEL_GENERATED_HEADERS})
    cmake_path(
        RELATIVE_PATH GENERATED_HEADER
        BASE_DIRECTORY "${GENERATED_CODE_DIR}/${RELATIVE_CURRENT_DIR}"
        OUTPUT_VARIABLE MODEL_NAME
    )
    cmake_path(
        REMOVE_EXTENSION MODEL_NAME
        OUTPUT_VARIABLE MODEL_NAME
    )

    if (MODEL_NAME IN_LIST ANIMATED_MODELS)
        list(APPEND DYNAMIC_ANIMATED_MODEL_HEADERS ${GENERATED_HEADER})
    else()
        list(APPEND DYNAMIC_NON_ANIMATED_MODEL_HEADERS ${GENERATED_HEADER})
    endif()
endforeach()

set(DYNAMIC_MODEL_LIST_OUT_DIR "${GENERATED_CODE_DIR}/${RELATIVE_CURRENT_DIR}")

set(DYNAMIC_MODEL_LIST_H "${DYNAMIC_MODEL_LIST_OUT_DIR}/dynamic_model_list.h")
set(DYNAMIC_MODEL_LIST_C "${DYNAMIC_MODEL_LIST_OUT_DIR}/dynamic_model_list.c")
add_custom_command(
    DEPENDS
        models ${DYNAMIC_NON_ANIMATED_MODEL_HEADERS}
        ${GEN_DYNAMIC_MODEL_LIST} ${MODEL_LIST_UTILS}
    OUTPUT
        ${DYNAMIC_MODEL_LIST_H} ${DYNAMIC_MODEL_LIST_C}
    COMMAND
        ${NodeJs_EXECUTABLE} ${GEN_DYNAMIC_MODEL_LIST}
        ${DYNAMIC_MODEL_LIST_H}
        ${DYNAMIC_NON_ANIMATED_MODEL_HEADERS}
    COMMENT
        "Generating dynamic model list"
    VERBATIM
)

set(DYNAMIC_ANIMATED_MODEL_LIST_H "${DYNAMIC_MODEL_LIST_OUT_DIR}/dynamic_animated_model_list.h")
set(DYNAMIC_ANIMATED_MODEL_LIST_C "${DYNAMIC_MODEL_LIST_OUT_DIR}/dynamic_animated_model_list.c")
add_custom_command(
    DEPENDS
        models ${DYNAMIC_ANIMATED_MODEL_HEADERS}
        ${GEN_DYNAMIC_ANIM_MODEL_LIST} ${MODEL_LIST_UTILS}
    OUTPUT
        ${DYNAMIC_ANIMATED_MODEL_LIST_H} ${DYNAMIC_ANIMATED_MODEL_LIST_C}
    COMMAND
        ${NodeJs_EXECUTABLE} ${GEN_DYNAMIC_ANIM_MODEL_LIST}
        ${DYNAMIC_ANIMATED_MODEL_LIST_H}
        ${DYNAMIC_ANIMATED_MODEL_HEADERS}
    COMMENT
        "Generating dynamic animated model list"
    VERBATIM
)

set(MODEL_LIST_FILES
    ${DYNAMIC_MODEL_LIST_H}
    ${DYNAMIC_MODEL_LIST_C}
    ${DYNAMIC_ANIMATED_MODEL_LIST_H}
    ${DYNAMIC_ANIMATED_MODEL_LIST_C}
)

add_custom_target(generated_model_lists
    DEPENDS ${MODEL_LIST_FILES}
)

add_library(model_lists INTERFACE)
add_dependencies(model_lists generated_model_lists)
target_sources(model_lists INTERFACE
    ${MODEL_LIST_FILES}
)

# Add command for generating dynamic model linker script fragment

set(DYNAMIC_MODELS_LINKER_SCRIPT "${LINKER_SCRIPT_DIR}/dynamic_models.ld")
add_custom_command(
    DEPENDS
        ${GEN_SEGMENT_LD}
    OUTPUT
        ${DYNAMIC_MODELS_LINKER_SCRIPT}
    COMMAND
        ${NodeJs_EXECUTABLE} ${GEN_SEGMENT_LD}
        ${DYNAMIC_MODELS_LINKER_SCRIPT}
        0x03000000
        "$<TARGET_OBJECTS:models_dynamic>"
    COMMENT
        "Generating $<PATH:RELATIVE_PATH,${DYNAMIC_MODELS_LINKER_SCRIPT},${PROJECT_SOURCE_DIR}>"
    COMMAND_EXPAND_LISTS
    VERBATIM
)

add_custom_target(models_dynamic_linker_script
    DEPENDS ${DYNAMIC_MODELS_LINKER_SCRIPT}
)
set_target_properties(models_dynamic_linker_script PROPERTIES
    SCRIPT_FILE "${DYNAMIC_MODELS_LINKER_SCRIPT}"
)
