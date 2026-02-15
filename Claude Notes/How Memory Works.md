---
date: "2026-02-16"
tags: [claude-code, memory, system]
pinned: true
---

# Claude Code Memory System

## How it works

Claude Code has a persistent memory at `~/.claude/projects/-home-james/memory/`.
The file `MEMORY.md` is loaded into **every new session** automatically.

This means: anything written there, I will "remember" next time.

## What's stored

- **About you** - name, preferences, work style
- **Environment** - tools, paths, MCP servers
- **Active projects** - what you're working on
- **Key decisions** - so I don't repeat questions
- **Session log** - important context across sessions

## How to update

You can update my memory two ways:

1. **Tell me directly**: "Remember that I prefer X" / "Forget about Y"
2. **Edit in Obsidian**: Open `Claude Notes/Memory Mirror.md` or directly edit
   `\\wsl$\Ubuntu\home\james\.claude\projects\-home-james\memory\MEMORY.md`

## Limitations

- MEMORY.md has a 200-line limit (lines after 200 are truncated)
- Detailed notes go in separate topic files (skills.md, etc.)
- Memory is per-project path, not truly global
