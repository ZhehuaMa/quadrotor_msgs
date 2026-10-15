# quadrotor_msgs 仓库约定

- 本仓库是独立 Git 仓库，职责为共享 ROS 2 消息包。修改前读取 `AGENTS.local.md` 与 `/home/mzh/nickluo/AGENTS.md`。
- 只在本仓库实现和交付本仓库改动；不修改相邻仓库，不自动提交、推送或更新根仓库的子模块版本。
- 修改 C++ 时读取 `~/.codex/language_norms/cpp_norms.md`。文档使用简体中文，记录实际验证平台与范围。
- Generate interfaces from `msg/` with ROS 2 Humble. Default to the x86_64 image for native x86_64 output or use its ARM cross toolchain for ARM64 output; the ARM64 image is an explicit fallback. See `README.md` for commands and runtime boundaries. Build this package for the same target before its consumers; never copy generated headers as a substitute. Keep generated Python extensions with their matching interpreter architecture.
- 修改消息字段、单位或坐标系后核对相邻消费者；控制角速度为 rad/s，质量归一化推力为 m/s²。
