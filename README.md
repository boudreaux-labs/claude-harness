# claude-harness

Claude Code configuration for the [boudreaux-labs](https://github.com/boudreaux-labs) GitHub org.

This repo lives at the root of the local boudreaux-labs workspace so that Claude Code discovers it as a parent-directory config, making agents and skills available across all repos in the org.

## Structure

```
.claude/
  agents/
    security-auditor.md   # Pre-push and on-demand security audit agent
  skills/
    ui-stylist/
      SKILL.md            # Color theme authoring for boudreaux-labs front-end apps
```

## Agents

### security-auditor

Invoked automatically on `git push` (via pre-push hook in `settings.json`) and on-demand via the Agent tool. Covers:

- AWS IAM and cloud posture
- Kubernetes manifest review (RBAC, container hardening, resource limits)
- Dockerfile and nginx config review
- GitHub Actions permission scope
- Secrets detection
- Version currency

### Usage

The security auditor runs automatically before any `git push`. It can also be invoked directly by asking Claude to audit a file, manifest, or diff.

## Skills

### ui-stylist (`/ui-stylist`)

Manages color themes for boudreaux-labs front-end apps. Themes are defined as structured objects in `themes.js` with a full Material Design 3 token set, accent values, and font assignments.

Accepts color input in any format — hex values, natural language, screenshots, Tailwind config snippets — and derives a complete theme.

### Usage

```
/ui-stylist add a new theme — dark forest green with copper accents
```

## Setup

Clone into the root of your local boudreaux-labs workspace:

```bash
git clone https://github.com/boudreaux-labs/claude-harness C:/Users/<you>/Code/boudreaux-labs
```

`settings.json` and `settings.local.json` are gitignored — configure locally after cloning.
