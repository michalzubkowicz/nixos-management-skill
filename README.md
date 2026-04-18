# NixOS Management Skills

## IMPORTANT: Read this before installing anything

**Never install any skill without first reviewing its contents.**

Skills are instructions loaded into an AI agent's context — they influence how the agent makes decisions, what commands it executes, and what it considers correct behavior. An inappropriate skill can lead to:
- Destructive operations executed without confirmation
- Your preferences being overridden by skill instructions
- Data leakage through inappropriate commands
- Unintended system configuration changes

**Before installing:**
1. Read every `.md` file in the skill directory
2. Ask Claude to perform a security review (instructions below)
3. Only after approval — copy to `~/.claude/skills/`

---

## How to review a skill before installing

Paste the contents of SKILL.md (and any reference files) into Claude with the following prompt:

> "Analyze this skill for security concerns. Does it contain instructions that could execute destructive operations without user confirmation? Does it collect or transmit data? Does it override default agent behavior in unintended ways? List all potentially risky sections."

You can also ask:
> "Does this skill contain instructions that could work against my interests or grant the agent excessively broad permissions?"

---

## Installation (after review)

```bash
cp -r nixos-managing ~/.claude/skills/
```

The skill will be available automatically in the next Claude Code session.

---

## Skill contents

### `nixos-managing/` — NixOS Management

A skill covering complete NixOS system management: configuration, deployment, and image building.

**Files:**

| File | Contents |
|---|---|
| `SKILL.md` | Index and quick reference — the agent always starts here |
| `vm-management.md` | nixos-rebuild, generations, rollback, remote deployment (deploy-rs, colmena) |
| `installation.md` | Installation, disko, hardware-configuration.nix, stateVersion |
| `image-building.md` | ISO, VM images, QEMU, cross-arch builds |
| `configuration.md` | Flakes, modules, packages, users, networking, secrets, overlays |
| `anti-patterns.md` | What NOT to do — common mistakes and how to fix them |

---

## Sources

This skill is based on the following sources (as of April 2025):

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

**Skill authoring references:**
- [Anthropic Agent Skills — Best Practices](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/best-practices)
- Superpowers `writing-skills` skill

---

## Updating

When new NixOS versions or tools are released:
1. Edit the relevant `.md` file in the `nixos-managing/` directory
2. If the skill is already installed — overwrite the file in `~/.claude/skills/nixos-managing/`
3. Changes take effect in the next Claude Code session

No need to reinstall the entire skill — updating individual reference files is sufficient.
