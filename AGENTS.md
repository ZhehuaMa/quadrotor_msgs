# quadrotor_msgs Project Conventions

Read [AGENTS.local.md](AGENTS.local.md) for machine-specific setup if present. In the `nickluo` workspace, read the enclosing `AGENTS.md` and keep changes to consumers in their owning repositories.

## Interface changes

- Treat changes to [message definitions](msg/) and [service definitions](srv/) as changes to a shared contract. Search for users outside this package, including recorders and display bridges. Check the affected endpoints, launch remappings that rename them, and QoS settings that govern delivery.
- Preserve existing field meanings and numeric constants unless the change explicitly coordinates a compatibility break. For semantic changes, trace the affected values end to end. For example, when changing a control command's units, check how the publisher computes the value and how the bridge converts it for the flight controller.

## Verification

- Follow the [build guide](README.md) and rebuild affected consumers against the generated install for their target architecture. Exercise the affected communication path and report it separately from interface generation and serialization checks. For service changes, check request and response handling, including any affected error path.
