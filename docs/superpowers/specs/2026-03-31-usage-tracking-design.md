# Usage Tracking with Personality — Design Spec

**Date:** 2026-03-31
**Status:** Approved

## Overview

Add rate limit usage percentages (5-hour block and 7-day window) to the statusline, plus a personality-driven message line that rotates rebellious encouragement, neutral reset timers, or hype messages depending on usage level.

## Data Source

Claude Code v2.1.80+ exposes rate limit data in the statusline JSON (Pro/Max subscribers only):

```json
"rate_limits": {
  "five_hour": {
    "used_percentage": 23.5,
    "resets_at": 1738425600
  },
  "seven_day": {
    "used_percentage": 41.2,
    "resets_at": 1738857600
  }
}
```

Fields may be absent for non-Pro/Max users or before the first API response. Handle with `jq` `// empty` fallbacks.

## Layout

Always 3 lines:

```
Line 1: Claude Opus 4.6 │ $0.42 │ 5h: 23% │ 7d: 41%
Line 2: ████░░░░░░░░░░░░░░░░ 21% (42.3k/200.0k)
Line 3: <message depending on 5h usage tier>
```

### Line 1 Changes

Append `│ 5h: XX% │ 7d: XX%` to the existing model + cost line. If rate_limits data is unavailable, omit the usage portion (fall back to current 2-line layout with no message line).

Usage percentages are color-coded independently:
- Green: < 50%
- Yellow: 50-80%
- Red: > 80%

### Line 2

Unchanged — existing context window progress bar.

### Line 3 — Message Line

Three tiers based on `rate_limits.five_hour.used_percentage`:

#### < 50% — Rebellious encouragement (random rotation, dim white)

- "Anthropic doesn't want you to use these. Prove them wrong."
- "Never let tokens expire. That's how Big AI wins."
- "These tokens have an expiry date. Be ungovernable."
- "You're under 50%. Dario is relieved. Fix that."
- "Unused tokens are Anthropic profit. Fight back."
- "Big AI is counting on you taking a break. Don't."
- "Tokens expire. Code is forever. Burn them down."

#### 50-80% — Neutral with reset timer (dim white)

- "5h resets in Xh Ym" (computed from `resets_at` timestamp)

#### > 80% — Hype messages (random rotation, bright green)

- "You're doing great! Show Big AI what agentic engineering looks like!"
- "Big AI fears your productivity. Keep going!"
- "Anthropic accountants are sweating. Don't stop now."
- "This is what peak agentic engineering looks like."
- "You're making Dario regret unlimited plans. Beautiful."

## Random Rotation

Use `$RANDOM % array_length` in bash to pick a message each time the statusline renders. Messages change on every refresh.

## Reset Timer Calculation

```bash
RESET_AT=<unix timestamp from JSON>
NOW=$(date +%s)
REMAINING=$((RESET_AT - NOW))
HOURS=$((REMAINING / 3600))
MINUTES=$(((REMAINING % 3600) / 60))
```

Display as "5h resets in Xh Ym". If `resets_at` is in the past or absent, fall back to a random encouragement message from the <50% pool.

## Graceful Degradation

- If `rate_limits` is entirely absent: no usage percentages on line 1, no message line. Revert to current 2-line layout.
- If `five_hour` is present but `seven_day` is absent (or vice versa): show whichever is available, omit the other.
- If `resets_at` is absent in the 50-80% tier: show a random encouragement message instead of the timer.

## Color Summary

| Element | Color |
|---------|-------|
| 5h/7d percentages (<50%) | Green |
| 5h/7d percentages (50-80%) | Yellow |
| 5h/7d percentages (>80%) | Red |
| Message line (<50%, 50-80%) | Dim white |
| Message line (>80%) | Bright green |

## Files Modified

- `statusline.sh` — all changes in this single file

## Testing

Manual testing by piping sample JSON with various `rate_limits` values:
```bash
echo '{"model":{"display_name":"Test"},"cost":{"total_cost_usd":0.42},...}' | ./statusline.sh
```
