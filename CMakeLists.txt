cmake_minimum_required(VERSION 3.16)
project(piper_sdk_description NONE)

find_package(ament_cmake REQUIRED)

install(DIRECTORY meshes urdf DESTINATION share/${PROJECT_NAME})

if(BUILD_TESTING)
    # Every model must expand to a valid URDF, with and without the gripper.
    find_program(XACRO xacro REQUIRED)
    foreach(model piper piper_h piper_l piper_x)
        foreach(gripper true false)
            add_test(NAME xacro_${model}_${gripper}
                COMMAND ${XACRO} ${CMAKE_CURRENT_SOURCE_DIR}/urdf/piper_description.urdf.xacro
                        model:=${model} with_gripper:=${gripper} -o ${CMAKE_CURRENT_BINARY_DIR}/${model}_${gripper}.urdf)
        endforeach()
    endforeach()
    # urdf/generated/ (plain URDFs that piper_sdk compiles in) must be what the xacros give
    # now (including generated/sim/, the full URDFs for simulators): regenerate them into the
    # build dir and compare.
    add_test(NAME generated_urdfs_up_to_date
        COMMAND bash -c "${CMAKE_CURRENT_SOURCE_DIR}/scripts/generate_plain_urdfs.sh 0.035 ${CMAKE_CURRENT_BINARY_DIR}/generated \
            && diff -r ${CMAKE_CURRENT_SOURCE_DIR}/urdf/generated ${CMAKE_CURRENT_BINARY_DIR}/generated \
            || { echo 'urdf/generated is out of date: run scripts/generate_plain_urdfs.sh'; exit 1; }")
endif()

ament_package()
