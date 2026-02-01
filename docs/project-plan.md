# Aura Project Plan

## Phased Implementation Roadmap

### Phase 0: Project Foundation ✅
- [x] Create repo at `/Users/cjvana/Documents/GitHub/aura`
- [x] Create `CLAUDE.md` with project context
- [x] Create `docs/design-plan.md` with full architecture
- [x] Create `docs/project-plan.md` (this file)
- [x] Set up directory structure
- [x] Initialize git repo, initial commit

### Phase 1: Scaffold + Basic Terminal
- [ ] Set up `Cargo.toml` with dependencies (alacritty_terminal, winit, wgpu, wry, cosmic-text)
- [ ] Implement PTY spawning and management (`src/terminal/pty.rs`)
- [ ] Implement GPU-rendered terminal grid (`src/terminal/grid.rs`)
- [ ] Implement keyboard/mouse input forwarding (`src/terminal/input.rs`)
- [ ] Wire up winit window with wgpu surface
- [ ] Verify: launch Aura, run shell commands, see output

**Verification**: Launch aura, type commands, verify shell works correctly.

### Phase 2: Mood Engine
- [ ] Implement signal collectors (`src/mood/signals.rs`)
  - Keystroke dynamics (velocity, variance, backspace ratio)
  - Error pattern matching on PTY output
  - Exit code tracking
  - Git status polling
- [ ] Implement MoodState model (`src/mood/model.rs`)
- [ ] Implement EMA computation loop (`src/mood/engine.rs`)
- [ ] Implement zone classification (`src/mood/zones.rs`)
- [ ] Add debug logging (gated behind `AURA_DEBUG=1`)

**Verification**: Run `AURA_DEBUG=1 aura`, verify mood readings change with typing patterns.

### Phase 3: Reactive Theming
- [ ] Define `ThemeParams` struct and mood-zone-to-theme mappings (`src/theme/palette.rs`)
- [ ] Implement OKLCH color interpolation (`src/theme/interpolation.rs`)
- [ ] Implement theme manager with smooth transitions (`src/theme/manager.rs`)
- [ ] Wire mood engine → theme manager → GPU renderer
- [ ] Implement "Triumphant" flash event (gold glow on test pass / build success)

**Verification**: Visually confirm theme shifts. Run `exit 1` repeatedly → frustrated zone. Run `echo "test passed"` → triumphant flash.

### Phase 4: Claude Code Hooks
- [ ] Implement `aura hook <event>` CLI subcommand (`src/claude/hooks.rs`)
- [ ] Implement Unix domain socket IPC (`src/claude/socket.rs`)
- [ ] Implement Claude session tracker (`src/claude/tracker.rs`)
- [ ] Implement mood injection / systemMessage generation (`src/claude/injection.rs`)
- [ ] Auto-install hooks into `~/.claude/settings.json`

**Verification**: Run `claude` inside Aura, verify hooks fire (check logs), verify systemMessage in context.

### Phase 5: AI Awareness Panel
- [ ] Set up wry webview sidebar alongside terminal grid
- [ ] Build mood orb WebGL shader (`panel/orb.js`)
- [ ] Build Claude orb + sync visualization
- [ ] Build session stats display (`panel/stats.js`)
- [ ] Build timeline ribbon (`panel/timeline.js`)
- [ ] Implement Rust ↔ JS IPC bridge

**Verification**: Panel renders, orbs animate, stats update in real time.

### Phase 6: Config + Polish
- [ ] TOML configuration system (`src/config/settings.rs`)
- [ ] Built-in theme profiles (ocean, ember, arctic, neon, minimal)
- [ ] Break suggestions when Fatigued >15min
- [ ] README with usage docs
- [ ] Final testing and polish

**Verification**: End-to-end — code inside Aura with Claude Code for 30 minutes, observe full feedback loop.

## Dependency Graph

```
Phase 0 (Foundation)
  └→ Phase 1 (Terminal)
       ├→ Phase 2 (Mood Engine)
       │    └→ Phase 3 (Theming) ← depends on mood output
       │         └→ Phase 6 (Config) ← depends on theme system
       ├→ Phase 4 (Claude Hooks) ← depends on mood engine (Phase 2)
       └→ Phase 5 (Panel) ← depends on mood engine (Phase 2)
```

Phases 3, 4, and 5 can be worked in parallel once Phase 2 is complete.

## Key Crates

| Crate | Purpose | Version |
|---|---|---|
| `alacritty_terminal` | VTE parsing, grid, PTY | latest |
| `winit` | Cross-platform windowing | 0.30+ |
| `wgpu` | GPU rendering | 23+ |
| `wry` | Webview for side panel | 0.46+ |
| `cosmic-text` | Font shaping and rasterization | latest |
| `serde` + `toml` | Configuration | latest |
| `tokio` | Async runtime (socket, PTY I/O) | 1.x |
| `log` + `env_logger` | Logging | latest |
| `thiserror` / `anyhow` | Error handling | latest |
| `clap` | CLI argument parsing | 4.x |
