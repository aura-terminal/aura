# Aura — Mood-Reactive Terminal Emulator

## What This Is
A GPU-accelerated terminal emulator in Rust that detects coding mood from behavioral signals and dynamically shifts its appearance. Native Claude Code integration via hooks creates a feedback loop where the AI adapts to the developer's state.

## Tech Stack
- **Rust** with Cargo workspace
- **wgpu** for GPU rendering (Metal/Vulkan/DX12)
- **winit** for windowing
- **alacritty_terminal** for VTE parsing, grid management, PTY
- **wry** (Tauri webview) for the side panel
- **cosmic-text** for font shaping
- **serde + toml** for configuration

## Architecture
```
src/
  main.rs              — Entry point, window + webview setup
  terminal/            — PTY, grid renderer, input handling
  mood/                — Mood engine (signals → EMA → zones)
  theme/               — Reactive theming (OKLCH interpolation)
  claude/              — Hook CLI, Unix socket IPC, mood injection
  config/              — TOML settings
panel/                 — Webview sidebar (HTML/JS/CSS, WebGL orb)
```

## Key Design Decisions
- Mood model is 2D (valence + arousal) plus a flow coefficient
- EMA smoothing (alpha=0.15) on a 2-second tick
- Theme philosophy: **supportive, not mirror** — frustrated gets calming tones, not angry red
- Claude hooks communicate via Unix domain socket to the running Aura instance
- Color interpolation happens in OKLCH perceptual color space

## Mood Zones
Flow, Hyperfocus, Grinding, Frustrated, Debugging, Exploring, Fatigued, Triumphant

## Building
```bash
cargo build
cargo run
```

## Testing
```bash
cargo test
cargo clippy -- -D warnings
```

## Environment Variables
- `AURA_DEBUG=1` — Enable debug logging for mood engine
- `AURA_CONFIG=<path>` — Override default config location

## Conventions
- Use `thiserror` for error types, `anyhow` in main/bin code
- Prefer `log` + `env_logger` for logging
- Keep mood engine decoupled from rendering — communicate via channels
- All color math in OKLCH; convert to sRGB only at the GPU boundary
- Unix socket path: `$XDG_RUNTIME_DIR/aura.sock` or `/tmp/aura-$UID.sock`
