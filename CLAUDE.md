# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal knowledge vault managed with [Obsidian](https://obsidian.md) and versioned with Git. It contains learning notes and concept breakdowns organized by topic. There are no build, test, or lint commands — it is a documentation-only repository.

## Repository Structure

```
concepts/       # Concept notes, organized by topic (e.g., concepts/git/)
docs/           # Guidance for agents and repo navigation
.obsidian/      # Obsidian vault configuration (do not edit manually)
TOC.md          # Table of contents (currently a placeholder)
```

## Conventions

- Notes live under `concepts/<topic>/` as Markdown files
- `docs/` is intended for agent-readable guidance and repo navigation aids
- Obsidian workspace state is tracked in `.obsidian/workspace.json` — avoid editing it outside Obsidian

## Working with Notes

- New concept notes should follow the pattern `concepts/<topic>/<note-name>.md`
- Notes may include Obsidian-flavored Markdown (wikilinks `[[...]]`, callouts, embeds)
- Cross-links between notes use Obsidian wikilink syntax, not standard Markdown links
