# Aura

**A mood-reactive terminal emulator.**

Aura is a GPU-accelerated terminal emulator that detects your coding mood from behavioral signals and dynamically shifts its appearance in real time. Built-in Claude Code integration creates a feedback loop where the AI adapts its communication style to your current state.

> Frustrated? The terminal shifts to calm deep blues. In flow? It gets out of your way. Triumphant? A brief gold flash celebrates with you.

## How It Works

Aura observes signals from your coding session — typing speed, error rates, backspace frequency, exit codes, git status — and feeds them into a 2D mood model (valence × arousal) with a flow coefficient. The mood engine uses exponential moving averages to smooth transitions between eight mood zones:

| Zone | What It Looks Like |
|---|---|
| **Flow** | Minimal, clean, high-contrast. Stays out of your way. |
| **Hyperfocus** | Dark, focused, slight vignette. Tunnel vision support. |
| **Grinding** | Neutral warm tones. Steady companionship. |
| **Frustrated** | Calm deep blues, reduced contrast, slower animations. |
| **Debugging** | Cooled tones, crisp fonts. Clinical clarity. |
| **Exploring** | Soft greens, open feel. Curiosity-friendly. |
| **Fatigued** | Warm amber like a desk lamp. Comfort. |
| **Triumphant** | Brief gold flash, then back to business. |

The design philosophy is **supportive, not mirror** — the terminal doesn't amplify your state, it counterbalances it.

## Features

- **GPU-accelerated rendering** via wgpu (Metal / Vulkan / DX12)
- **Real-time mood detection** from typing dynamics, errors, and session patterns
- **Reactive theming** with smooth OKLCH color interpolation
- **Claude Code hooks** that inject developer mood context into AI conversations
- **AI awareness panel** — a webview sidebar with mood orb visualization, session stats, and timeline
- **Configurable** via TOML with built-in theme profiles

## Requirements

- **Rust** 1.75+ (stable)
- **macOS** (Metal), **Linux** (Vulkan), or **Windows** (DX12/Vulkan)
- **Linux system dependencies**:
  ```bash
  sudo apt-get install -y \
    libxkbcommon-dev libwayland-dev \
    libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev \
    libgtk-3-dev libwebkit2gtk-4.1-dev \
    libsoup-3.0-dev libjavascriptcoregtk-4.1-dev
  ```

## Building

```bash
git clone https://github.com/cjvana/aura.git
cd aura
cargo build --release
```

## Usage

```bash
# Run Aura
cargo run --release

# With debug logging for the mood engine
AURA_DEBUG=1 cargo run --release

# With a custom config file
AURA_CONFIG=~/.config/aura/my-config.toml cargo run --release
```

## Configuration

Aura is configured via TOML at `~/.config/aura/config.toml`:

```toml
[mood]
sensitivity = 0.5          # 0.0-1.0, how reactive the mood engine is
ema_alpha = 0.15           # EMA smoothing factor
tick_interval_ms = 2000    # Mood update interval

[theme]
profile = "ocean"          # Built-in: ocean, ember, arctic, neon, minimal
transition_duration_ms = 2000

[panel]
enabled = true
position = "right"         # right, left
width = 280

[claude]
hooks_enabled = true
mood_injection = true
injection_style = "subtle" # subtle, detailed, minimal

[terminal]
font_family = "JetBrains Mono"
font_size = 14.0
scrollback_lines = 10000
```

## Claude Code Integration

Aura registers hooks with Claude Code to create a bidirectional feedback loop:

1. **Mood injection** — On session start, Aura tells Claude the developer's current state, so Claude adapts its tone (concise when you're in flow, careful when you're frustrated)
2. **Event tracking** — Tool use results feed back into the mood engine
3. **Claude mood** — Aura derives a secondary mood from Claude's behavior (tool call frequency, error rates) and visualizes it alongside yours

Communication happens via Unix domain socket at `$XDG_RUNTIME_DIR/aura.sock`.

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

See [docs/design-plan.md](docs/design-plan.md) for the full architecture and [docs/project-plan.md](docs/project-plan.md) for the implementation roadmap.

## Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Security

To report vulnerabilities, see [SECURITY.md](SECURITY.md).

## License

MIT
