# quadrotor_msgs local development

- Read `/home/mzh/nickluo/AGENTS.local.md`. Source is `/home/mzh/nickluo/quadrotor_msgs`.
- Follow `README.md`. Default image is `drone-tracker-rk3576:humble-amd64-rknn2.3.2` with `--platform linux/amd64`. It supports native x86_64 generation or real ARM64 cross-generation with `/opt/toolchains/aarch64-humble.cmake`.
- Keep native, ARM-cross and ARM-image outputs outside source in separate directories. Preserve earlier `/home/mzh/nickluo-validation-20261015/quadrotor_msgs/` evidence.
- ARM64-image fallback requires `--platform linux/arm64` and `drone-tracker-rk3576:humble-arm64-rknn2.3.2`; it is not a prerequisite for native or cross compilation.
- Rebuild actual consumers against the same-target generated package. Do not load ARM extensions into native Python. No registered CTests exist; import/serialization and consumer checks are distinct from build success.
