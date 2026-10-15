# quadrotor_msgs build architectures

This ROS 2 Humble source package defines shared messages. ROS interface generation creates C/C++ libraries and Python extension modules for the selected target architecture. It has no ARM64-container requirement and no dependency on RKNN or physical hardware.

| Compiler environment | Generated output | Verified scope |
| --- | --- | --- |
| Default `drone-tracker-rk3576:humble-amd64-rknn2.3.2` | Native x86_64 interfaces/libraries | Real `apm_bridge` and interceptor consumers built and linked; native simulation imports. |
| ARM cross toolchain in the default image | ARM64 interfaces/libraries and Python 3.10 extensions | Clean generation and both cross-built consumers verified; ARM Python imports/type support previously verified. |
| Explicit `drone-tracker-rk3576:humble-arm64-rknn2.3.2` | ARM64 interfaces/libraries | Existing build and all five message serialization round trips verified under QEMU on the x86 host. |

A Python extension module is compiled code loaded by Python. Its architecture must match the interpreter: ARM extensions cannot be loaded by native x86_64 Python. Keep native and ARM build/install directories separate. Do not copy generated headers into consumers or substitute runtime deployment bundles for this source dependency.

## Native generation and a serialization example

```bash
mkdir -p /tmp/quadrotor-msgs-native
docker run --rm --network none --platform linux/amd64 \
  --user "$(id -u):$(id -g)" \
  -v /home/mzh/nickluo/quadrotor_msgs:/src:ro \
  -v /tmp/quadrotor-msgs-native:/out \
  drone-tracker-rk3576:humble-amd64-rknn2.3.2 bash -lc '
    set -eo pipefail
    source /opt/ros/humble/setup.bash
    cmake -S /src -B /out/build -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=/out/install
    cmake --build /out/build -j4
    cmake --install /out/build
    source /out/install/share/quadrotor_msgs/local_setup.bash
    python3 -c "from quadrotor_msgs.msg import ControlCommand; from rclpy.serialization import serialize_message, deserialize_message; m = ControlCommand(); assert deserialize_message(serialize_message(m), ControlCommand) == m"
  '
```

The example sends a default message through ROS serialization and deserialization and checks equality. It does not test a publisher/subscriber link. No CTest cases are registered by this package; a zero-test discovery is not a regression pass.

## ARM64 generation in the x86_64 image

Use the native recipe with a separate `/tmp/quadrotor-msgs-cross` host output directory. Add `-DCMAKE_TOOLCHAIN_FILE=/opt/toolchains/aarch64-humble.cmake` to its `cmake -S` command. Stop after installation; do not source the ARM install into the native interpreter or run the native serialization line. Verify an output library inside the same container:

```bash
file /out/install/lib/libquadrotor_msgs__rosidl_generator_c.so
```

It must report `ELF 64-bit ... ARM aarch64`. The toolchain uses native Python for generation, real ARM Python 3.10 headers/libraries for extension compilation, and extension filename tag `cpython-310-aarch64-linux-gnu`. ARM runtime tests need ARM Python under QEMU or a real ARM64 system. The tracker helper `../drone_tracker/tests/tracking_regression/target_python.sh` configures that target runtime for its own test installation; do not mix unrelated prefixes.

Build each consumer against this same-target install using `-Dquadrotor_msgs_DIR=/out/install/share/quadrotor_msgs/cmake`. Complete consumer recipes are in `../apm_bridge/docs/build-and-test.md` and `../drone_interceptor/docs/DOCKER_BUILD.md`.

## Explicit fallback and evidence

For the ARM64-image fallback, use the native recipe with `--platform linux/arm64`, image `drone-tracker-rk3576:humble-arm64-rknn2.3.2`, and a separate `/tmp/quadrotor-msgs-arm64-image` output directory. It produces ARM64 directly inside the container. The recipe's Python then runs as ARM64 through host QEMU. There is no automatic image switch after failures.

Current audit evidence is `/tmp/nickluo-submodule-image-audit-20261015/cross-ros.sh` and `logs/`: clean cross-generation, actual ARM64 consumers and full target-loader dependency lists. Earlier native, ARM Python and ARM-image evidence is preserved in the parent workspace's `docs/amd64-build-validation-20261015.md` and `/home/mzh/nickluo-validation-20261015/VALIDATION_REPORT.md`. Message serialization does not establish communication timing, flight units or target-board behavior.
