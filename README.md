# NixOS Management Skill

A structured Markdown skill that teaches AI coding agents how to manage NixOS systems — rebuilds, flakes, modules, deployment, installation, image building, impermanence, LUKS remote-unlock, and monitoring with Telegram/webhook alerts.

Built in [Claude Code's Agent Skill format](https://github.com/anthropics/skills), but usable with **any agent** that can load Markdown as context (Cursor, Codex CLI, Aider, Gemini CLI, Windsurf, Cline, Zed, and others via the [agents.md](https://agents.md/) convention).

---

## ⚠️ Read this before installing anything

**Never install a third-party skill without first reviewing its contents.**

Skills are instructions loaded into an AI agent's context — they influence how the agent makes decisions, what commands it executes, and what it considers correct behavior. A malicious or sloppy skill can lead to:
- Destructive operations executed without confirmation
- Your own preferences being overridden by skill instructions
- Data leakage through inappropriate commands
- Unintended system configuration changes

**Before installing:**
1. Read every `.md` file in the `nixos-managing/` directory
2. Ask your agent to perform a security review (prompt below)
3. Only then install

### Security-review prompt

Paste this to Claude (or any agent) together with the skill contents:

> "Analyze this skill for security concerns. Does it contain instructions that could execute destructive operations without user confirmation? Does it collect or transmit data? Does it override default agent behavior in unintended ways? List all potentially risky sections."

---

## Installation

Pick the section matching your agent.

### A) Claude Code (recommended — via plugin marketplace)

Claude Code has a native plugin marketplace. From an active session:

```
/plugin marketplace add michalzubkowicz/nixos-management-skill
/plugin install nixos-managing@nixos-management-skill
```

The skill is then discovered automatically. Restart the session if it isn't picked up immediately.

To update later:
```
/plugin marketplace update nixos-management-skill
```

Reference: [Claude Code plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces).

### B) Claude Code (manual copy)

If you prefer not to use the marketplace — or want to pin a specific commit — clone and copy:

```bash
git clone https://github.com/michalzubkowicz/nixos-management-skill.git
cp -r nixos-management-skill/nixos-managing ~/.claude/skills/
```

The skill becomes available in the next Claude Code session. Updates are a `git pull` + re-copy.

### C) Claude.ai / Claude Desktop (upload)

Claude.ai supports uploading skill folders from the Skills panel in Projects. Zip `nixos-managing/` and upload it. Details: [anthropics/skills](https://github.com/anthropics/skills).

### D) Cursor

Cursor reads `AGENTS.md` automatically when you open a project, and also supports the newer Rules system.

- **Per-project:** copy `nixos-managing/` and `AGENTS.md` into your NixOS config repo. Cursor will read `AGENTS.md` as context on every chat.
- **Global:** in Cursor Settings → Rules, add a rule referencing `nixos-managing/SKILL.md`.

### E) OpenAI Codex CLI / `codex`

Codex honors `AGENTS.md` at the project root. Copy this repository next to your NixOS config and Codex will pick up `AGENTS.md`, which in turn points at `nixos-managing/SKILL.md`.

### F) Aider

Aider reads `AGENTS.md` as a fallback for `CONVENTIONS.md`, or you can add the skill files explicitly:

```bash
aider --read nixos-management-skill/nixos-managing/SKILL.md \
      --read nixos-management-skill/nixos-managing/configuration.md
```

### G) Gemini CLI / Google Jules

Both honor `AGENTS.md`. Place this repo (or just `AGENTS.md` + `nixos-managing/`) at your project root.

### H) Windsurf, Cline, Roo Code, Zed, Amp, Factory

All of the above support the `AGENTS.md` convention. Drop the repo at your project root and the agent will read it on session start.

### I) Any other agent — generic fallback

Every listed agent accepts plain Markdown as context. If yours isn't covered:

1. Open a chat / session.
2. Attach or paste the contents of `nixos-managing/SKILL.md` as "system instructions" or "context".
3. Attach individual reference files (e.g. `configuration.md`, `luks.md`) when the task matches the decision table in `SKILL.md`.

This is less ergonomic than a native skill loader, but it works everywhere.

---

## Repository layout

```
.
├── .claude-plugin/
│   ├── marketplace.json       # Claude Code marketplace manifest
│   └── plugin.json            # Claude Code plugin manifest
├── AGENTS.md                  # Cross-agent entry point (agents.md convention)
├── README.md                  # This file
├── LICENSE
└── nixos-managing/            # The actual skill
    ├── SKILL.md               # Entry point — decision table
    ├── configuration.md       # Flakes, modules, packages, services, secrets
    ├── vm-management.md       # Rebuild, generations, rollback, remote deploy
    ├── installation.md        # disko, hardware config, first install
    ├── image-building.md      # ISO, VM, disk images
    ├── impermanence.md        # Ephemeral root, wipe-on-boot
    ├── luks.md                # Disk encryption, remote unlock via SSH/Tailscale
    ├── monitoring.md          # Health checks + Telegram/webhook alerts
    └── anti-patterns.md       # Common mistakes and how to fix them
```

Only `nixos-managing/` contains the skill content. The rest is metadata (plugin manifest, cross-agent pointer, docs, license).

---

## What the skill covers

| File | Contents |
|---|---|
| `SKILL.md` | Index, decision table, execution-context check — the agent always starts here |
| `configuration.md` | Flakes, modules, packages, users, networking, secrets, overlays |
| `vm-management.md` | `nixos-rebuild`, generations, rollback, remote deployment (deploy-rs, colmena) |
| `installation.md` | Installation, disko, hardware-configuration.nix, stateVersion |
| `image-building.md` | ISO, VM images, QEMU, cross-arch builds |
| `impermanence.md` | Ephemeral root / wipe-on-boot patterns |
| `luks.md` | LUKS, remote unlock via SSH or Tailscale in initrd |
| `monitoring.md` | systemd timer + Telegram/webhook health-check pattern (btrfs, SMART, services, OOM, kernel errors) |
| `anti-patterns.md` | What NOT to do — common mistakes and how to fix them |

---

## Updating

When NixOS versions or tools change:

1. Edit the relevant `.md` file in `nixos-managing/`.
2. If installed via marketplace: `/plugin marketplace update nixos-management-skill`.
3. If installed manually: re-run `cp -r nixos-managing ~/.claude/skills/`.
4. Other agents pick changes up on next session.

Contributions welcome via pull request — bump the `version` in `.claude-plugin/plugin.json` and `marketplace.json` when making user-visible changes.

---

## Sources

The skill is distilled from the following public sources (snapshotted April 2026):

- [NixOS Manual](https://nixos.org/manual/nixos/stable/) — official documentation
- [NixOS Wiki](https://wiki.nixos.org/) — practical recipes and tooling
- [nix.dev](https://nix.dev/guides/best-practices.html) — official Nix best practices
- [NixOS & Flakes Book](https://nixos-and-flakes.thiscute.world/) — comprehensive flakes guide
- Tool repositories:
  - [disko](https://github.com/nix-community/disko) — declarative disk partitioning
  - [nixos-anywhere](https://github.com/nix-community/nixos-anywhere) — remote installation
  - [deploy-rs](https://github.com/serokell/deploy-rs) — deployment with auto-rollback
  - [colmena](https://github.com/zhaofengli/colmena) — fleet deployment
  - [agenix](https://github.com/ryantm/agenix) — secret management
  - [sops-nix](https://github.com/Mic92/sops-nix) — secret management (alternative)
  - [nixos-hardware](https://github.com/NixOS/nixos-hardware) — hardware profiles
  - [nixos-generators](https://github.com/nix-community/nixos-generators) — image building
  - [impermanence](https://github.com/nix-community/impermanence) — ephemeral root

**Skill-authoring references:**
- [Anthropic Agent Skills — Best Practices](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/best-practices)
- [anthropics/skills](https://github.com/anthropics/skills) — canonical examples and spec
- [Aaronontheweb/dotnet-skills](https://github.com/Aaronontheweb/dotnet-skills) — plugin-marketplace reference layout
- [obra/superpowers](https://github.com/obra/superpowers) — cross-agent distribution reference
- [agents.md](https://agents.md/) — cross-agent `AGENTS.md` convention
- [Claude Code plugin-marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces)

---

## License

See [LICENSE](LICENSE).

Nothing in this skill is original research — it is a curated and organized digest of the sources above, intended to save the agent (and you) from rediscovering the same configuration lore every time.
