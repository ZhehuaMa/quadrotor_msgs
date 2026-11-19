# quadrotor_msgs Local Setup

- Read `../AGENTS.local.md` if the enclosing workspace provides it.
- On this ARM64 macOS workstation, run `docker context show` before using the [build guide](README.md). Verify that the selected Colima context has the build image and can mount both the checkout and the output directory under your home directory.
- Enable x86_64 emulation for the `linux/amd64` image, including when it cross-compiles ARM64 output.
