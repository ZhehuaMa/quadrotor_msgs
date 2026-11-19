# quadrotor_msgs

`quadrotor_msgs` defines shared ROS 2 Humble [messages](msg/) and [services](srv/) for quadrotor software. Build and install it before building packages that use its generated C/C++ libraries or Python modules.

## Build and check the interfaces

To build the package, use CMake, a C/C++ compiler, and the ROS dependencies in the [package manifest](package.xml). The Docker images below provide those dependencies and `rclpy` for the Python serialization check. Interface generation runs without RKNN or target hardware.

Prepare the image for your chosen mode in your Docker context. For image setup in the `nickluo` workspace, see the [tracker image guide](../drone_tracker/dockerfiles/README.amd64.md).

| `build_mode` | Docker image | Output |
| --- | --- | --- |
| `amd64` | `drone-tracker-rk3576:humble-amd64-rknn2.3.2` | Linux x86_64 |
| `arm64-cross` | `drone-tracker-rk3576:humble-amd64-rknn2.3.2` | Linux ARM64, cross-compiled in the x86_64 image |
| `arm64-image` | `drone-tracker-rk3576:humble-arm64-rknn2.3.2` | Linux ARM64, built inside the ARM64 image |

Run the example in Bash from this checkout. Set `build_mode` to the desired row. Each run creates a fresh output directory under your home directory and mounts the checkout read-only. Both directories must be accessible to Docker. Running an image built for a different CPU architecture requires emulation.

```bash
(
  set -e
  build_mode=amd64

  case "$build_mode" in
    amd64|arm64-cross)
      platform=linux/amd64
      image=drone-tracker-rk3576:humble-amd64-rknn2.3.2
      ;;
    arm64-image)
      platform=linux/arm64
      image=drone-tracker-rk3576:humble-arm64-rknn2.3.2
      ;;
    *)
      printf 'Unknown build mode: %s\n' "$build_mode" >&2
      exit 1
      ;;
  esac

  source_dir=$(git rev-parse --show-toplevel)
  output_dir=$(mktemp -d "$HOME/quadrotor-msgs-$build_mode.XXXXXX")
  printf 'Output directory: %s\n' "$output_dir"

  docker run --rm --network none --platform "$platform" \
    --user "$(id -u):$(id -g)" \
    -v "$source_dir:/src:ro" -v "$output_dir:/out" \
    "$image" bash -lc '
      set -eo pipefail
      source /opt/ros/humble/setup.bash
      cmake_args=()
      if [[ "$1" == arm64-cross ]]; then
        cmake_args+=(-DCMAKE_TOOLCHAIN_FILE=/opt/toolchains/aarch64-humble.cmake)
      fi

      cmake -S /src -B /out/build -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/out/install "${cmake_args[@]}"
      cmake --build /out/build -j4
      cmake --install /out/build

      if [[ "$1" == arm64-cross ]]; then
        file /out/install/lib/libquadrotor_msgs__rosidl_generator_c.so
      else
        source /out/install/share/quadrotor_msgs/local_setup.bash
        python3 - <<"PY"
from quadrotor_msgs.msg import ControlCommand
from rclpy.serialization import deserialize_message, serialize_message

message = ControlCommand()
assert deserialize_message(serialize_message(message), ControlCommand) == message
PY
      fi
    ' build "$build_mode"
)
```

For `arm64-cross`, check that `file` reports an ELF library for `ARM aarch64`. To load the generated Python extension, use a compatible ARM64 Python interpreter on an ARM64 system or under emulation. A Python extension contains compiled code and must match both the interpreter's version and CPU architecture.

The other modes check that Python can import `ControlCommand` and recover the same message after encoding and decoding it. Use a consumer integration check to verify delivery between ROS nodes. This package has no registered CTest cases.

## Use the generated interfaces

Build each consumer against an install generated from this source for the consumer's target architecture. Set `quadrotor_msgs_DIR` to `share/quadrotor_msgs/cmake` under the install directory as seen by the consumer build. For example, pass `-Dquadrotor_msgs_DIR=/out/install/share/quadrotor_msgs/cmake` if the install is mounted at `/out/install`. For workspace examples, see the [bridge build guide](../apm_bridge/doc/build-and-test.md) and [interceptor build guide](../drone_interceptor/docs/DOCKER_BUILD.md).
