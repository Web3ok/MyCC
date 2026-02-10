# MyCC (My Claude Code)

Quota monitor + countdown + auto-resume for Claude Code. One script, zero hassle.

```
  ┌────────────────────────────────────────────────────┐
  │  MyCC v1.0.0                                       │
  ├────────────────────────────────────────────────────┤
  │  5-Hour  [████████████████░░░░]  82%  Feb 10 3PM   │
  │  7-Day   [██████░░░░░░░░░░░░░░]  31%  Feb 14      │
  │  Opus    [████████████████████] 100%  Feb 12       │
  │  Sonnet  [████░░░░░░░░░░░░░░░░]  22%  Feb 15      │
  │                                                    │
  │  Status: RATE LIMITED (Opus 7-day)                 │
  │  Resets in 01:18:22                                │
  │  Action: Auto-resume (continue)                    │
  └────────────────────────────────────────────────────┘
```

[English](#features) | [中文](README_zh.md)

## Why?

Claude Code (Pro/Max) users hit rate limits constantly. When it happens:

1. You don't know **how much quota** you've used across each dimension
2. You don't know **exactly when** the limit resets
3. You have to **manually come back** and resume your work

MyCC solves all three: **monitor → countdown → auto-resume**.

## Features

- **Real-time quota monitoring** — See 5-hour, 7-day, Opus, and Sonnet usage at a glance
- **Precise countdown** — Know exactly when your limit resets, down to the second
- **Auto-resume** — Automatically continues your Claude Code session when the limit lifts
- **tmux integration** — `--launch` starts Claude Code with a live status bar at the bottom
- **Dual mode** — Basic (zero-config CLI detection) or Detailed (claude.ai API with full % breakdown)
- **Single file** — One shell script, ~900 lines, zero dependencies for basic mode
- **macOS + Linux** — Works on both (auto-detects BSD/GNU tools)

## Install

```bash
# Download
curl -fsSL https://raw.githubusercontent.com/Web3ok/MyCC/main/mycc -o ~/.local/bin/mycc
chmod +x ~/.local/bin/mycc

# Or clone
git clone https://github.com/Web3ok/MyCC.git
cp MyCC/mycc ~/.local/bin/mycc
chmod +x ~/.local/bin/mycc
```

Requirements:
- **Basic mode**: No dependencies (bash, curl, grep, perl — all pre-installed on macOS/Linux)
- **Detailed mode**: `jq` (`brew install jq` / `apt install jq`)
- **Launch mode**: `tmux` (`brew install tmux` / `apt install tmux`)

## Quick Start

### One-command launch (recommended)

```bash
# Start Claude Code with live status bar at bottom
mycc --launch

# Continue previous session + status bar
mycc --launch "-c"
```

This opens a tmux split — Claude Code on top, MyCC status bar on bottom:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Claude Code (interactive session)                              │
│                                                                 │
│  > Working on your code...                                      │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ [17:01:10] 5h ████████ 100%  7d ███░░░░░ 45%  LIMITED 02:14:32 │
└─────────────────────────────────────────────────────────────────┘
```

### Manual usage

```bash
mycc --check              # One-shot status check
mycc                      # Wait for limit to lift, then auto-resume
mycc -n                   # Same, but start a new session
mycc -p "run tests"       # Auto-resume with custom prompt
mycc --no-resume          # Monitor only
mycc -d --check           # Detailed mode: see all quota percentages
```

## Two Modes

### Basic Mode (default)

Detects rate limits via the `claude` CLI. Zero configuration needed.

```bash
mycc --check
```

```
  ┌────────────────────────────────────────────────────┐
  │  MyCC v1.0.0                                       │
  ├────────────────────────────────────────────────────┤
  │  Status:  RATE LIMITED                             │
  │  Limit:   5-hour limit reached                     │
  │  Resets:  2026-02-10 15:00 (in 02:14:32)           │
  │  Action:  Auto-resume (continue)                   │
  └────────────────────────────────────────────────────┘
```

### Detailed Mode (`-d`)

Calls the claude.ai API for precise utilization percentages across all dimensions.

```bash
mycc -d --check
```

```
  ┌────────────────────────────────────────────────────┐
  │  MyCC v1.0.0                                       │
  ├────────────────────────────────────────────────────┤
  │  5-Hour  [████████████████░░░░]  82%  Feb 10 3PM   │
  │  7-Day   [██████░░░░░░░░░░░░░░]  31%  Feb 14      │
  │  Opus    [████████████████████] 100%  Feb 12       │
  │  Sonnet  [████░░░░░░░░░░░░░░░░]  22%  Feb 15      │
  │                                                    │
  │  Status: RATE LIMITED (Opus 7-day)                 │
  │  Resets in 01:18:22                                │
  └────────────────────────────────────────────────────┘
```

Requires a one-time setup:

```bash
mycc --setup
```

This will guide you to copy your `sessionKey` from the browser (claude.ai cookies).

## Usage Reference

```
Usage: mycc [OPTIONS]

Modes:
  (no flag)            Basic mode - detect limits via claude CLI
  -d, --detailed       Detailed mode - Claude.ai API for utilization %

Auto-resume:
  -c, --continue       Continue previous conversation (default)
  -n, --new            Start new session when resuming
  -p, --prompt TEXT    Custom resume prompt (default: "continue")
  --no-resume          Monitor only, don't auto-resume
  -w, --watch          Keep watching after resume (loop mode)

Launch:
  --launch             Start tmux: Claude Code (top) + status bar (bottom)
  --launch "ARGS"      Same, but pass ARGS to claude (e.g. --launch "-c")

Configuration:
  --setup              Interactive credential setup wizard
  --session-key KEY    Override session key (one-time)
  --org-id ID          Override organization ID (one-time)

Display:
  --compact            Single-line status bar (for tmux pane)
  --no-color           Disable ANSI colors
  --json               Output status as JSON
  --check              One-shot status check, then exit

Other:
  --test-mode SECS     Simulate rate limit (for testing)
  -h, --help           Show this help
  -v, --version        Show version
```

## Configuration

Config file: `~/.mycc.conf` (created by `mycc --setup`, permissions `600`)

| Field | Description | Default |
|-------|-------------|---------|
| SESSION_KEY | claude.ai browser session cookie | (empty, required for `-d`) |
| ORG_ID | Organization UUID (auto-discovered) | (empty, auto) |
| RESUME_MODE | `continue` or `new` | `continue` |
| RESUME_PROMPT | Prompt sent when resuming | `continue` |
| REFRESH_INTERVAL | API refresh interval in seconds | `120` |
| NOTIFY | macOS desktop notification on resume | `true` |

## JSON Output

For scripting and tool integration:

```bash
mycc -d --check --json | jq .
```

```json
{
  "status": "limited",
  "limit_name": "5-hour",
  "five_hour":  { "pct": 100, "resets_at": "2026-02-10T09:00:00Z" },
  "seven_day":  { "pct": 31,  "resets_at": "2026-02-14T00:00:00Z" },
  "opus":       { "pct": 100, "resets_at": "2026-02-12T09:00:00Z" },
  "sonnet":     { "pct": 22,  "resets_at": "2026-02-15T00:00:00Z" }
}
```

## Testing

```bash
mycc --test-mode 10              # Simulate 10s rate limit (basic)
mycc --test-mode 10 -d --check   # Simulate with detailed view
mycc --test-mode 30 -d --compact # Simulate compact status bar
mycc --test-mode 5 --json        # Simulate JSON output
```

## Security

- Config file is `chmod 600` (owner-only read/write)
- sessionKey is stored locally, never transmitted except to claude.ai
- Auto-resume uses `--dangerously-skip-permissions` — only use in trusted environments with trusted prompts
- No telemetry, no analytics, no network calls except to claude.ai API

## Inspired By

MyCC combines the best ideas from these open-source projects into a single, portable shell script:

| Project | Stars | What we took |
|---------|-------|--------------|
| [Claude-Code-Usage-Monitor](https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor) | 6.4k | Real-time token monitoring concept |
| [claude-auto-resume](https://github.com/terryso/claude-auto-resume) | 662 | Rate limit detection + auto-resume flow |
| [Usage4Claude](https://github.com/f-is-h/Usage4Claude) | 167 | claude.ai API integration for quota data |

## License

MIT
