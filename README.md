# piper_sdk_description

Meshes, visuals and simulator/RViz URDFs of the AgileX Piper series (piper, piper_h,
piper_l, piper_x, each bare or with the AgileX gripper). Only needed for simulation and
digital twins; piper_sdk itself needs just its own plain URDFs.

Part of Piper Lab as a shared resource repository beside `piper_control`, `piper_sim` and
`piper_studio`. The kinematic/inertial source URDFs belong to Control; meshes, visual overlays
and generated ROS/simulator URDFs belong here so all consumers can use them independently.

## Source of truth

The plain URDFs of the SDK (`piper_sdk/urdf/<model>[_gripper].urdf`: kinematics and
inertials, no meshes) are THE model of the arm. This package holds no kinematic or inertial
data. It adds what the SDK does not need, from `overlay/`:

- `overlay/materials.json`: material name -> rgba
- `overlay/<model>.json`: arm link -> `{mesh, material}` (mesh path under `meshes/`; visual
  and collision use the same mesh unless `collision` is given; `origin: {xyz, rpy}` optional)
- `overlay/agx_gripper.json`: the same for the flange and gripper links (`{model}` in a mesh
  path is the arm model)

`scripts/generate_urdfs.py` (python3 only, no ROS) writes, from an SDK URDF directory:

- `urdf/generated/ros/<name>.urdf`: `package://piper_sdk_description/meshes/...`
- `urdf/generated/sim/<name>.urdf`: meshes relative to the file (`../../../meshes/...`),
  movable gripper joints (`gripper` = jaw opening, `gripper_joint1/2` mimic it); for MuJoCo
  and other simulators that read a plain file

Both are committed, so this package works without an SDK checkout. After the SDK URDFs
change, run `scripts/generate_urdfs.py [--sdk-urdf-dir DIR]` and commit the result. The
directory defaults to `$PIPER_SDK_URDF_DIR`, `../piper_sdk/urdf` (this package as the SDK's
submodule) or `../piper_control/piper_sdk/urdf`. The test `generated_urdfs_up_to_date`
(`generate_urdfs.py --check`) fails when they are stale and is skipped without an SDK.

## ROS

`urdf/piper_description.urdf.xacro` (args `model`, `with_gripper`) and the macro
`urdf/piper_arm.xacro` (`piper_arm model= with_gripper=`) include the matching file of
`urdf/generated/ros/`.

## Provenance

Kinematics, inertials and meshes come from AgileX agx_arm_urdf (MIT); meshes (STL) were
copied once into `meshes/<model>/` and `meshes/agx_gripper/`, and the kinematics now live in
the SDK's URDFs.
