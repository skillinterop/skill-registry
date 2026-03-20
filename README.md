# skill-registry

Skill packages for the interop ecosystem.

## Overview

This repository is a **leaf registry** that stores skill packages only. It is NOT the hub — the hub is [`registry-hub`](https://github.com/skillinterop/registry-hub). Wrapper tools consume this registry's `manifest.json` to discover and install skills.

## Directory Structure

```
skill-registry/
├── manifest.json              # Registry manifest with all skill entries
├── schemas/
│   └── manifest.schema.json   # JSON Schema for manifest validation
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md           # Skill documentation
│       └── metadata.json      # Skill metadata
├── README.md
└── .gitignore
```

## Available Skills

| Canonical ID | Name | Version | Description |
|--------------|------|---------|-------------|
| `skill/org/[MASKED_EMAIL]` | workmux-router | 1.0.0 | Natural-language task launcher and router using workmux and git worktree |

## Manifest Format

The `manifest.json` file follows the LeafManifest structure:

```json
{
  "registryType": "skill",
  "namespace": "org",
  "version": "0.1.0",
  "channel": "experimental",
  "generatedAt": "2026-03-20T07:00:00Z",
  "items": [...]
}
```

## How to Add a New Skill

1. Create a directory under `skills/<skill-name>/`
2. Add `SKILL.md` with skill documentation
3. Add `metadata.json` with skill metadata
4. Update `manifest.json` to include the new skill entry
5. Open a PR with the changes

## Canonical ID Format

All skills use the canonical ID format:

```
skill/<namespace>/<name>@<version>
```

Example: `skill/org/[MASKED_EMAIL]`

## Related Repos

- [`registry-hub`](https://github.com/skillinterop/registry-hub) — Top-level hub that aggregates leaf registries
- [`cao-profile-registry`](https://github.com/skillinterop/cao-profile-registry) — CAO profile leaf registry
- [`reprogate-registry`](https://github.com/skillinterop/reprogate-registry) — ReproGate leaf registry

## License

MIT
