# Aura Design Plan

## 1. Overview

Aura is a GPU-accelerated terminal emulator that senses your coding mood from behavioral signals and dynamically shifts its visual appearance in real time. It integrates with Claude Code via hooks, creating a bidirectional feedback loop: the terminal tells Claude how the developer is doing, and Claude's behavior feeds back into the mood model.

## 2. Mood Detection Engine

### 2.1 Signal Sources

| Signal | Source | Update Rate |
|---|---|---|
| Typing velocity | Keystroke timestamps from PTY input | Per keystroke |
| Typing variance | Stddev of inter-key intervals | Per keystroke |
| Backspace ratio | Backspace count / total keystrokes (rolling window) | Per keystroke |
| Error rate | Pattern matching on PTY output (compile errors, stack traces) | Per PTY read |
| Exit codes | Process exit status from PTY | Per command |
| Git status | `git status --porcelain` polling | Every 30s |
| Time of day | System clock | Every 2s tick |
| Session duration | Time since Aura launch | Every 2s tick |

### 2.2 Mood Model

```rust
struct MoodState {
    valence: f32,    // -1.0 (negative) to 1.0 (positive)
    arousal: f32,    // 0.0 (calm) to 1.0 (intense)
    flow: f32,       // 0.0 (scattered) to 1.0 (deep focus)
    confidence: f32, // 0.0 to 1.0, how certain we are of the reading
}
```

### 2.3 Algorithm

Exponential Moving Average with alpha=0.15, ticked every 2 seconds:

```
new_value = alpha * raw_signal + (1 - alpha) * previous_value
```

Each signal contributes to valence and arousal with configured weights. Flow is derived from typing consistency (low variance + sustained velocity).

### 2.4 Mood Zones

Zones are regions in valence-arousal-flow space:

| Zone | Valence | Arousal | Flow | Description |
|---|---|---|---|---|
| Flow | 0.3–0.8 | 0.4–0.7 | >0.7 | Productive deep work |
| Hyperfocus | 0.2–0.6 | 0.7–1.0 | >0.8 | Intense concentration |
| Grinding | -0.1–0.3 | 0.4–0.7 | 0.3–0.7 | Working through it |
| Frustrated | <-0.2 | >0.5 | <0.4 | Hitting walls |
| Debugging | -0.1–0.3 | 0.3–0.6 | 0.4–0.7 | Methodical investigation |
| Exploring | 0.3–0.8 | 0.2–0.5 | <0.5 | Learning, reading |
| Fatigued | <0.1 | <0.3 | <0.3 | Running low |
| Triumphant | >0.7 | >0.6 | any | Just succeeded |

Zone classification uses nearest-centroid with soft boundaries for smooth transitions.

## 3. Reactive Theming

### 3.1 Theme Parameters

```rust
struct ThemeParams {
    // Colors (OKLCH)
    bg_color: Oklch,
    fg_color: Oklch,
    ansi_palette: [Oklch; 16],
    cursor_color: Oklch,
    selection_color: Oklch,

    // Atmosphere
    bg_opacity: f32,
    vignette_strength: f32,
    grain_amount: f32,

    // Typography
    font_weight: f32,
    letter_spacing: f32,
    line_height: f32,

    // Effects
    cursor_style: CursorStyle,
    glow_radius: f32,
    particle_density: f32,
}
```

### 3.2 Design Philosophy: Supportive, Not Mirror

The theme system **does not** mirror mood — it **supports** the developer:

- **Frustrated** → Calm deep blues, reduced contrast, slower animations. Soothe, don't amplify.
- **Fatigued** → Warm amber tones like a desk lamp, slightly increased font size. Comfort.
- **Flow** → Minimal, clean, high-contrast. Stay out of the way.
- **Hyperfocus** → Dark, focused, slight vignette. Tunnel vision support.
- **Triumphant** → Brief gold flash, then return to current zone. Celebrate, don't distract.
- **Debugging** → Slightly cooled tones, crisp fonts. Clinical clarity.
- **Exploring** → Soft greens, open feel. Curiosity-friendly.
- **Grinding** → Neutral warm tones, steady. Companionship.

### 3.3 Interpolation

All color transitions happen in OKLCH perceptual color space for smooth, natural-looking shifts. Theme parameters are interpolated over ~2 seconds using ease-in-out curves synchronized with the mood engine tick.

## 4. Claude Code Integration

### 4.1 Hook Architecture

Aura registers Claude Code hooks for these events:
- `SessionStart` — Inject developer mood into Claude's system context
- `PreToolUse` — (reserved for future gate logic)
- `PostToolUse` — Update Claude's derived mood based on tool results
- `Stop` — Log session end, update session stats
- `Notification` — Display in panel
- `PreCompact` — Save mood snapshot before context compaction

### 4.2 Communication Flow

```
Claude Code → runs `aura hook <event>` → reads JSON from stdin
  → sends to running Aura via Unix domain socket
  → Aura processes event, updates mood model
  → returns JSON response to stdout → Claude Code reads it
```

### 4.3 Mood Injection

On `SessionStart`, Aura generates a `systemMessage` addition:

```json
{
  "systemMessage": "Developer mood context: currently in Flow state (valence=0.6, arousal=0.5, flow=0.8). Session duration: 45min, energy level: good. Adapt your communication style: be concise and direct, the developer is in a productive groove."
}
```

The injection adapts:
- **Frustrated**: "Be more careful and explanatory. Double-check assumptions. Offer alternatives."
- **Flow**: "Be concise. Don't over-explain. Match the developer's pace."
- **Fatigued**: "Keep responses shorter. Suggest breaks if appropriate. Be encouraging."
- **Debugging**: "Be methodical. Show your reasoning. Suggest diagnostic steps."

### 4.4 Claude's Derived Mood

Aura tracks Claude's behavior to derive a secondary mood:
- Tool call frequency → arousal
- Error rate in tool results → frustration indicators
- Context usage (tokens used / limit) → pressure
- Successive failures → confidence drop

This feeds the Claude orb visualization in the panel.

## 5. AI Awareness Panel

### 5.1 Layout

The panel is a webview sidebar (wry) rendered alongside the terminal grid. Default position: right side, 280px wide. Collapsible.

### 5.2 Components

**Mood Orb** (top): Generative WebGL visualization
- Hue = valence (blue=negative, green=neutral, warm=positive)
- Animation speed = arousal
- Surface coherence = flow (smooth when focused, turbulent when scattered)
- Size pulses gently with confidence

**Claude Orb** (below mood orb): Same visualization for Claude's derived mood
- When developer and Claude moods align, orbs visually "harmonize" (similar colors, synchronized pulse)
- When misaligned, visual contrast is apparent

**Session Stats** (middle):
- Session duration
- Commands run
- Error count
- Git status summary
- Claude context usage (tokens, compactions)

**Timeline Ribbon** (bottom): Horizontal color strip showing mood history
- Each pixel column = ~30 seconds of session time
- Color = mood zone color
- Hoverable for details

### 5.3 Communication

Rust → JS communication via wry's IPC:
- Rust pushes mood updates to JS via `window.ipc.postMessage()`
- JS can request data via custom protocol handler
- Update rate: mood orb at 60fps (interpolated), stats at 2s intervals

## 6. Terminal Rendering

### 6.1 GPU Pipeline (wgpu)

The terminal grid is rendered using wgpu with a simple pipeline:
1. **Glyph atlas**: cosmic-text shapes text → rasterize to atlas texture
2. **Cell shader**: Vertex shader positions quads per cell, fragment shader samples glyph atlas + applies colors
3. **Post-processing**: Optional vignette, grain, glow as mood-driven post-process passes
4. **Composition**: Terminal layer + effects composited to swapchain

### 6.2 PTY Management

Using `alacritty_terminal` for the heavy lifting:
- PTY spawning (fork/exec with proper terminal setup)
- VTE parsing (escape sequences, cursor movement, scrollback)
- Grid management (cells, scrollback buffer, selection)

Aura's renderer reads the grid state and draws it via wgpu.

## 7. Configuration

TOML-based configuration at `~/.config/aura/config.toml`:

```toml
[mood]
sensitivity = 0.5          # 0.0–1.0, how reactive the mood engine is
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
