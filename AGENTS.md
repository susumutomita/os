# Repository Guidelines

## Project Structure & Module Organization
- `src/main.rs` owns the UEFI entry point (`efi_main`) plus inline unit tests; keep new modules under `src/` and gate hardware-specific code behind feature flags when possible.
- `chapter2/boot/` contains reference boot assets (`boot.sh`, `image.hex`, `img.bin`) used by the book workflow—treat them as generated inputs, not Rust modules.
- `scripts/launch_qemu.sh` wraps QEMU with the bundled OVMF image in `third_party/ovmf/`; point it at the compiled EFI binary when testing graphics changes.
- Top-level automation lives in `Taskfile.yml`, `Dockerfile`, and `docker-compose.yml`; the `task` runner mirrors common CI steps.

## Build, Test, and Development Commands
- `cargo build --target x86_64-unknown-uefi` produces the bare-metal binary without launching it.
- `task run` (or `cargo run -- --target x86_64-unknown-uefi --workspace`) builds and runs the firmware entry point locally.
- `task run:watch` keeps running `cargo watch -x "run --workspace"` for rapid feedback during driver experiments.
- `task lint` chains `cargo fmt`, `cargo clippy`, and `yamllint`; use it before reviews.
- `task shell` opens the dev container (`docker compose exec os-dev bash`) with toolchains pre-installed.

## Coding Style & Naming Conventions
- Rust code is formatted with `cargo fmt --all`; keep 4-space indentation and let the formatter win conflicts.
- Enable `#![deny(warnings)]`-friendly code: resolve `cargo clippy --workspace -D warnings` locally.
- Use `snake_case` for functions and locals, `PascalCase` for types, and `SCREAMING_SNAKE_CASE` for constants; keep module names lowercase.
- Group unsafe blocks with brief comments describing invariants so reviewers can reason about UEFI calls quickly.

## Testing Guidelines
- Run `task test` (`cargo test --workspace`) before pushing; add `#[cfg(test)] mod tests` near logic you touch.
- For coverage reports use `task test:coverage`, which installs and runs `cargo tarpaulin`—aim to exercise new protocol paths.
- Name new tests after the behavior under test (e.g., `locate_graphic_protocol_handles_missing_guid`).

## Commit & Pull Request Guidelines
- Follow the existing Conventional Commit style: `type(scope): summary`, e.g., `build(efi): update bootloader` or `fix(scripts): ensure ovmf path`.
- Keep commits focused and include rationale or references in the body when touching firmware interactions.
- Pull requests should summarize changes, list manual test evidence (command output or screenshots), and link related issues or book chapters.

## Firmware & Emulator Workflow
- After building, run `scripts/launch_qemu.sh target/x86_64-unknown-uefi/debug/os.efi` to boot the image with bundled OVMF.
- The script stages artifacts under `mnt/EFI/BOOT/`; clean or regenerate that directory if you swap binaries between runs.
