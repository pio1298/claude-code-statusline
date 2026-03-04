# claude-code-statusline

A real-time statusline for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that tracks context usage, costs, and optionally logs sessions to Obsidian.

```
@your_username | Claude 4 Opus | ▓▓▓▓░░░░░░ 42% | $0.0031/1k | Session: $0.1847 | API: $12.50 | ✅ healthy
```

## Features

- **Context rot tracking** — visual progress bar + health warnings at 70% and 85%
- **Real-time cost** — per-1k-token rate and session total
- **API spend** — month-to-date billing via Anthropic Admin API (optional)
- **GitHub identity** — shows your `@username` from `gh` CLI
- **Obsidian logging** — auto-generates daily session tables (optional)

## Quick Install

```bash
git clone https://github.com/blushdas/claude-code-statusline.git
cd claude-code-statusline
bash install.sh
```

The installer will:
1. Copy `statusline.sh` to `~/.claude/statusline.sh`
2. Add the `statusLine` config to `~/.claude/settings.json` (preserves existing settings)
3. Optionally prompt for environment variables

## Manual Setup

### 1. Copy the script

```bash
cp statusline.sh ~/.claude/statusline.sh
chmod +x ~/.claude/statusline.sh
```

### 2. Add to settings

Add this to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline.sh"
  }
}
```

### 3. Restart Claude Code

The statusline appears at the bottom of your terminal.

## Configuration

All configuration is via environment variables. Add these to your `.zshrc` / `.bashrc`:

| Variable | Required | Description |
|----------|----------|-------------|
| `OBSIDIAN_VAULT` | No | Path to your Obsidian vault for session logging |
| `ANTHROPIC_ADMIN_API_KEY` | No | Admin API key for month-to-date spend tracking |

### Example `.zshrc`

```bash
# Claude Code Statusline
export OBSIDIAN_VAULT="$HOME/Documents/MyVault"
export ANTHROPIC_ADMIN_API_KEY="sk-ant-admin01-..."
```

## Dependencies

| Tool | Required | Install |
|------|----------|---------|
| `jq` | Yes | `brew install jq` / `apt install jq` |
| `gh` | Yes | `brew install gh` / `apt install gh` |
| `bc` | Yes | Pre-installed on most systems |
| `curl` | For API spend | Pre-installed on most systems |

## Context Rot Thresholds

The statusline warns you as context fills up:

| Threshold | Display | Meaning |
|-----------|---------|---------|
| < 70% | `✅ healthy` | Normal operation |
| 70–84% | `⚠️  wrap up soon` | Start wrapping up or compacting |
| 85%+ | `🔴 ROT — start new session` | Context is degraded, start fresh |

## Obsidian Integration

When `OBSIDIAN_VAULT` is set, the statusline creates daily notes at:

```
{OBSIDIAN_VAULT}/Claude Sessions/Claude Sessions — 2025-03-15.md
```

Each note contains a live-updating table:

| Time | Model | Context% | $/1k tokens | Session $ | Tokens | Git Branch | Status |
|------|-------|----------|-------------|-----------|--------|------------|--------|
| 14:22:01 | Claude 4 Opus | 23% | $0.0029 | $0.0412 | ~14k | main | ✅ healthy |
| 14:35:18 | Claude 4 Opus | 45% | $0.0031 | $0.1203 | ~39k | feat/auth | ✅ healthy |

Plus a footer with month-to-date API spend.

See [`examples/obsidian-sample.md`](examples/obsidian-sample.md) for a full example.

## API Spend Tracking

To track your total Anthropic API spend:

1. Go to [console.anthropic.com/settings/admin-keys](https://console.anthropic.com/settings/admin-keys)
2. Create an Admin API key
3. Set `ANTHROPIC_ADMIN_API_KEY` in your shell profile

The API cost is cached for 5 minutes to avoid excessive requests. Without this key, the statusline tracks session costs locally.

> **Note:** The Admin API returns costs in cents. The script divides by 100 to display dollars correctly.

## How It Works

Claude Code pipes a JSON blob to the statusline command on each update. The script:

1. Parses the JSON with `jq` for model, tokens, cost, and context window data
2. Calculates real-time cost-per-1k-tokens
3. Fetches your GitHub username (cached 60 min)
4. Optionally queries the Anthropic Admin API for month-to-date spend (cached 5 min)
5. Builds a visual progress bar and context health warning
6. Outputs the formatted statusline
7. Optionally appends a row to the Obsidian daily note

## Troubleshooting

**Statusline not showing?**
- Make sure `~/.claude/settings.json` has the `statusLine` config
- Restart Claude Code after making changes

**`jq: command not found`**
- Install jq: `brew install jq` (macOS) or `apt install jq` (Linux)

**API cost shows $0.00?**
- Check that `ANTHROPIC_ADMIN_API_KEY` is set: `echo $ANTHROPIC_ADMIN_API_KEY`
- The key needs Admin permissions, not just API access
- Cost data refreshes every 5 minutes (check `~/.claude/.api_cost_cache`)

**GitHub username not showing?**
- Make sure you're logged in: `gh auth status`
- Cache refreshes every 60 minutes (check `~/.claude/.gh_user_cache`)

**Obsidian notes not appearing?**
- Verify `OBSIDIAN_VAULT` points to a valid directory: `ls $OBSIDIAN_VAULT`
- Notes are created in `$OBSIDIAN_VAULT/Claude Sessions/`

## License

MIT — see [LICENSE](LICENSE)
