# Fierce Plugins

Internal plugin marketplace for Fierce Staffing Services and Consulting LLC.

## Setup (one time)

In Cowork: Customize → Plugins → Add marketplace → enter `https://github.com/rcarth/fierce-plugins`.

## Installing a plugin

Customize → Plugins → Browse → Personal tab → find the "fierce-plugins" group → click "+" on `fierce-coo-orchestrator`.

## Getting updates

Confirmed 2026-08-19: Cowork desktop's per-plugin "Update" button does not work for personal git marketplaces (stays disabled even after a version bump, a marketplace sync, and a full app restart). This matches Anthropic issue [#65426](https://github.com/anthropics/claude-code/issues/65426), closed "not planned."

When this repo has a new version, update by removing and reinstalling instead of clicking Update:

1. Customize → Plugins → find "Fierce COO Orchestrator" → "⋯" menu → Remove.
2. Customize → Plugins → Browse → Personal → fierce-plugins → click "+" on `fierce-coo-orchestrator` to reinstall.

This pulls the current version from this repo (confirmed working), it just isn't an in-place update.

## Plugins

- `plugins/fierce-coo-orchestrator` — Fierce COO Orchestrator. See its own README for details and changelog.
