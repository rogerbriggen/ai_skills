# Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

## Purpose

This repository stores skills and configuration for Claude Code and GitHub Copilot. It is not a software project with a build system or tests — the primary artifacts are workflow/configuration files.
Try to build skills which works for both Claude Code and GitHub Copilot, but prioritize Claude Code compatibility if there is a conflict. If there is a conflict, document it clearly in the code comments and in the README.md.

Always make sure, that the README.md in the root and the README.md in each skills subdirectory (e.g. `skills/jira/README.md`) contain clear setup instructions for both Claude Code and GitHub Copilot users and is always updated with the latest information about the skills.

## GitHub Actions: Claude Code Integration

The workflow in [.github/workflows/claude.yml](.github/workflows/claude.yml) enables Claude Code to respond to `@claude` mentions in:

- Issue comments
- Pull request review comments
- Pull request reviews
- Issues (title or body)

Authentication uses `CLAUDE_CODE_OAUTH_TOKEN` (preferred) stored as a GitHub Actions secret. The `ANTHROPIC_API_KEY` alternative is commented out in the workflow.

### Customization points in the workflow

- `trigger_phrase` — change the mention keyword (default: `@claude`)
- `assignee_trigger` — trigger when a specific user is assigned to an issue
- `claude_args` — pass CLI flags (model, max-turns, allowed tools, system prompt)
- `settings` — inject environment variables (e.g. `NODE_ENV`)
