# Contributing to Aura

Thanks for your interest in contributing to Aura! This document covers the process for contributing to this project.

## Getting Started

### Prerequisites

- **Rust** (stable toolchain, 1.75+): Install via [rustup](https://rustup.rs/)
- **System dependencies** (Linux):
  ```bash
  sudo apt-get install -y \
    libxkbcommon-dev libwayland-dev \
    libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev \
    libgtk-3-dev libwebkit2gtk-4.1-dev \
    libsoup-3.0-dev libjavascriptcoregtk-4.1-dev \
    libvulkan-dev mesa-vulkan-drivers libegl1-mesa-dev
  ```
- **macOS**: Xcode Command Line Tools (`xcode-select --install`)

### Building

```bash
git clone https://github.com/cjvana/aura.git
cd aura
cargo build
```

### Running

```bash
cargo run
```

Enable debug logging for the mood engine:

```bash
AURA_DEBUG=1 cargo run
```

## Development Workflow

1. **Fork** the repository
2. **Create a branch** from `main`: `git checkout -b my-feature`
3. **Make your changes** following the conventions below
4. **Test**: `cargo test && cargo clippy -- -D warnings && cargo fmt --all -- --check`
5. **Commit** using [Conventional Commits](https://www.conventionalcommits.org/):
   - `feat: add new mood signal for clipboard activity`
   - `fix: correct OKLCH interpolation at hue boundary`
   - `docs: update configuration examples`
   - `refactor: extract signal processing into separate module`
6. **Push** and open a **Pull Request** against `main`

## Code Conventions

These are the project's core conventions — please follow them:

- **Error handling**: Use `thiserror` for library error types, `anyhow` in `main.rs` / binary code
- **Logging**: Use `log` + `env_logger`. Gate debug-heavy logging behind `AURA_DEBUG`
- **Color math**: All color operations happen in OKLCH. Convert to sRGB only at the GPU boundary
- **Architecture**: Keep the mood engine decoupled from rendering. Communicate via channels (`std::sync::mpsc` or `tokio::sync`)
- **Unix socket**: Path at `$XDG_RUNTIME_DIR/aura.sock` or `/tmp/aura-$UID.sock`
- **Formatting**: `cargo fmt` with default settings
- **Linting**: `cargo clippy -- -D warnings` must pass with zero warnings

## Architecture Overview

```
src/
  main.rs              — Entry point, window + webview setup
  terminal/            — PTY, grid renderer, input handling
  mood/                — Mood engine (signals -> EMA -> zones)
  theme/               — Reactive theming (OKLCH interpolation)
  claude/              — Hook CLI, Unix socket IPC, mood injection
  config/              — TOML settings
panel/                 — Webview sidebar (HTML/JS/CSS, WebGL orb)
```

Key design principle: **The mood engine is the core abstraction.** Everything else reads from or writes to it. The theme system reacts to mood. Claude hooks inject into mood. The panel visualizes mood.

## What to Work On

- Check [open issues](https://github.com/cjvana/aura/issues) for things labeled `good first issue` or `help wanted`
- Look at the project plan in `docs/project-plan.md` for the overall roadmap
- If you want to work on something not listed, open an issue first to discuss

## Pull Request Guidelines

- Keep PRs focused — one feature or fix per PR
- Include tests for new functionality where possible
- Update documentation if your change affects the user-facing behavior or configuration
- Fill out the PR template completely
- Make sure CI passes before requesting review

## Mood Engine Contributions

The mood engine is the heart of Aura. If you're working on it:

- New signals should be added to `src/mood/signals.rs`
- Signal weights are configurable — provide sensible defaults
- Test mood transitions with the debug overlay (`AURA_DEBUG=1`)
- The EMA smoothing constant (`alpha=0.15`) was chosen to balance responsiveness and stability. Changes to this value need justification

## Theme Contributions

When adding or modifying themes:

- Remember the design philosophy: **supportive, not mirror**. Frustrated developers should see calming colors, not angry ones
- All colors must be defined in OKLCH
- Test transitions between all mood zones — they should feel smooth and natural
- Built-in theme profiles go in the theme module with the naming convention: `ocean`, `ember`, `arctic`, `neon`, `minimal`

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.
