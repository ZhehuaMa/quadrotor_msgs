cmake_minimum_required(VERSION 3.10)
project(quadrotor_msgs)

# --------------------------------------------------------------------------- #
# Find dependencies.

find_package(ament_cmake REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)

# --------------------------------------------------------------------------- #
# Generate and export message interfaces.

rosidl_generate_interfaces(${PROJECT_NAME}
    msg/AutopilotFeedback.msg
    msg/ControlCommand.msg
    msg/LowLevelFeedback.msg
    msg/Trajectory.msg
    msg/TrajectoryPoint.msg
    srv/CliCommand.srv
    DEPENDENCIES geometry_msgs std_msgs nav_msgs
)

ament_export_dependencies(rosidl_default_runtime)


# --------------------------------------------------------------------------- #
# Call last.

ament_package()