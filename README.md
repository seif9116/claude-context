# Claude Code Context Statusline

A custom statusline for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that displays real-time context window usage with a color-coded progress bar.

![bash](https://img.shields.io/badge/bash-script-4EAA25?logo=gnubash&logoColor=white)
## What it does

Replaces the default Claude Code status bar with a richer display showing:

- **Model name** and **session cost**
- **Context usage bar** with token count and percentage
- **Color-coded warnings** as context fills up:
  - Green (< 50%) — Yellow (>= 50%) — Red (>= 80%)
  - **Bright red warning at 200k tokens** regardless of window size, based on research showing LLM quality degrades well before the context limit

## Why 200k?

Research shows LLM performance degrades substantially as input context grows — even when the model can perfectly retrieve all relevant information:

- [Lost in the Middle](https://arxiv.org/abs/2307.03172) (Liu et al., 2024) — 30%+ accuracy drops when relevant info sits in the middle of long contexts
- [Context Length Alone Hurts LLM Performance](https://arxiv.org/abs/2510.05381) (Du et al., 2025) — 13.9–85% degradation as input length increases, even with 100% retrieval accuracy
- [Context Rot](https://research.trychroma.com/context-rot) (Hong et al., 2025) — all 18 tested frontier models degrade as context grows

200k is a practical threshold: beyond it, quality noticeably drops. When you see the red warning, run `/compact` or `/clear`.

## Install

**Requirements:** `jq`, `bc` (most systems have `bc` pre-installed)

```bash
# 1. Copy the script
mkdir -p ~/.claude/scripts
cp statusline.sh ~/.claude/scripts/statusline.sh
chmod +x ~/.claude/scripts/statusline.sh

# 2. Add to ~/.claude/settings.json (merge with existing settings if needed)
# {
#   "statusLine": {
#     "type": "command",
#     "command": "~/.claude/scripts/statusline.sh"
#   }
# }

# 3. Restart Claude Code
```

A `settings-snippet.json` is included for reference — copy the `statusLine` block into your existing `~/.claude/settings.json`.

## Screenshot

```
Claude Opus 4.6 │ $0.42
████░░░░░░░░░░░░░░░░ 21% (42.3k/200.0k)
```

At 200k+ tokens:
```
██████████████░░░░░░░ 68% (210.5k/308.0k) ⚠ CONTEXT ROT — quality degrades past 200k (run /compact)
Research: Context Length Alone Hurts LLM Performance (Du et al. 2025)
```

