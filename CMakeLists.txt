cmake_minimum_required(VERSION 3.16)
project(piper_description NONE)

find_package(ament_cmake REQUIRED)

install(DIRECTORY meshes overlay scripts urdf DESTINATION share/${PROJECT_NAME})

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
    # urdf/generated/ (the full URDFs for ROS and simulators) must be what the generator gives
    # from the SDK's plain URDFs and overlay/ now. The SDK checkout is found as
    # scripts/generate_urdfs.py documents (sibling/parent checkout, $PIPER_SDK_URDF_DIR); without
    # one the test is skipped (exit code 77).
    find_package(Python3 REQUIRED COMPONENTS Interpreter)
    add_test(NAME generated_urdfs_up_to_date
        COMMAND ${Python3_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/scripts/generate_urdfs.py --check)
    set_tests_properties(generated_urdfs_up_to_date PROPERTIES SKIP_RETURN_CODE 77)
endif()

ament_package()
