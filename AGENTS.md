# CLAUDE.md - External Agent Instructions

This is Jake's (kikileroy) OpenClaw fork running Otto, a personal cognitive extension agent.

## Architecture

- ~/openclaw-new/ = OpenClaw runtime (this repo). Humans + external agents modify.
- ~/.openclaw-new/ = Otto's state directory (config, workspace, sessions). Otto primarily modifies, humans for config.
- ~/.openclaw-new/openclaw.json = Gateway + model config. Humans only.
- ~/.openclaw-new/workspace/ = Otto's mind (AGENTS.md, SOUL.md, MEMORY.md, daily logs). Otto primarily.
- ~/.openclaw-new/agents/ = Session transcripts (ephemeral). Otto automatic.

## State repo

~/.openclaw-new/ is tracked in git: https://github.com/kikileroy/otto-state (private)

## Building

From ~/openclaw-new:
  npm install
  npm run build

After building, restart the gateway:
  OPENCLAW_STATE_DIR=/home/otto/.openclaw-new node dist/entry.js gateway restart

Or via systemd:
  systemctl --user restart openclaw-gateway.service

## Gateway

- Systemd service: openclaw-gateway.service (user-level)
- Logs: journalctl --user -u openclaw-gateway.service -n 200 --no-pager
- Log file: /tmp/openclaw/openclaw-YYYY-MM-DD.log
- Entry point: ~/openclaw-new/dist/entry.js
- Binds to 127.0.0.1:18789 (loopback only, accessed via SSH tunnel)

## Working with Otto's workspace

Otto's workspace files are loaded into his context at session start. Changes take effect on his next session/turn.

Safe to edit externally: AGENTS.md, SOUL.md, USER.md, TOOLS.md, openclaw.json
Edit with care: MEMORY.md, memory/*.md (Otto writes these -- check git diff first)
Never edit: agents/*/sessions/*.jsonl (active session transcripts)

After editing workspace files, commit from ~/.openclaw-new/:
  cd ~/.openclaw-new && git add -A && git diff --staged --quiet || git commit -m "Human: [description]" && git push

## Security rules

- Never expose API keys, tokens, or secrets in code, logs, or commits
- Keys live in openclaw.json (mode 600) and GH_TOKEN in .bashrc
- Never commit .env files or token values
- The gateway token in the systemd service file is sensitive

## SSH access

  ssh -i /Users/jake/.ssh/factory_agent_vps otto@178.156.211.134

SSH tunnel for dashboard:
  ssh -i /Users/jake/.ssh/factory_agent_vps -N -L 18789:127.0.0.1:18789 otto@178.156.211.134

## Upstream sync

From ~/openclaw-new:
  git fetch upstream
  git merge upstream/main
  npm install && npm run build
  systemctl --user restart openclaw-gateway.service
