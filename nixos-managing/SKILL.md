---
name: nixos-managing
description: Use when managing NixOS systems — rebuilding, configuring, deploying, installing, or building images. Covers flakes, modules, secret management, VM management, disk imaging, remote deployment, and common anti-patterns to avoid.
---

# NixOS Management

## Quick Decision: What do you need?

| Task | Go to |
|---|---|
| Rebuild / activate / rollback system | [vm-management.md](vm-management.md) |
| Install NixOS on new machine | [installation.md](installation.md) |
| Build ISO / disk image / VM image | [image-building.md](image-building.md) |
| Configure modules, flakes, packages, services | [configuration.md](configuration.md) |
| Ephemeral root / wipe on boot / impermanence | [impermanence.md](impermanence.md) |
| LUKS encryption / remote unlock (SSH, Tailscale) | [luks.md](luks.md) |
| Health checks / Telegram (or webhook) alerts for filesystem, disk, SMART, services | [monitoring.md](monitoring.md) |
| Something isn't working / weird behavior | [anti-patterns.md](anti-patterns.md) |

## Execution Context — Ask First

**Before suggesting any commands, establish where they will run.**

NixOS management involves multiple machines. Commands that work on the NixOS host will fail on macOS or non-NixOS Linux. Always determine the setup before suggesting commands.

**If unsure — ask:** "Where are you editing configs, and where does nixos-rebuild run?"

### All common setups

**Setup A — NixOS local (everything on one machine)**
```
NixOS machine: edit → nixos-rebuild switch (local)
```
- All commands run locally
- `nixos-option`, `man configuration.nix`, `nixos-rebuild` — all local

---

**Setup B — NixOS workstation → NixOS remote server**
```
NixOS workstation (edit) → nixos-rebuild --target-host → NixOS server
```
- `nixos-rebuild` runs on workstation, deploys to server
- Option verification: workstation (has nixpkgs) or server via SSH
- `nixos-option` on server reflects what's actually active there

---

**Setup C — macOS → NixOS remote (Linux rebuilds locally)**
```
macOS (edit .nix files) → push/rsync → NixOS Linux host → nixos-rebuild switch (local)
```
- `nixos-rebuild` runs **on the Linux host** (SSH in, or via remote trigger)
- Option verification: **SSH into Linux host** — macOS has no `nixos-option`
- `search.nixos.org/options` always works from macOS

---

**Setup D — macOS → NixOS remote (macOS drives rebuild)**
```
macOS (edit) → nixos-rebuild --target-host root@linux-host --flake .#host
```
- `nixos-rebuild` runs on macOS but activates on Linux
- Requires Nix installed on macOS (`nix` CLI, not NixOS)
- Option verification: `nix eval` works on macOS if Nix is installed; otherwise use `search.nixos.org`

---

**Setup E — Linux (non-NixOS) → NixOS remote**
```
Ubuntu/Debian/Arch (edit + run nixos-rebuild) → NixOS remote server
```
- Same as Setup D — Nix must be installed on the source machine
- `nixos-option` not available unless NixOS; use `nix eval` or `search.nixos.org`

---

**Setup F — CI/CD → NixOS remote**
```
CI agent (GitHub Actions / GitLab CI) → deploy to NixOS server
```
- CI agent needs Nix installed (use `cachix/install-nix-action` or similar)
- No interactive verification — all options must be pre-validated
- Use `nix flake check` in CI to catch errors before deploy

---

### Verification availability by machine type

| Machine | `nixos-option` | `nix eval nixpkgs#...` | `man configuration.nix` | `search.nixos.org` |
|---|---|---|---|---|
| NixOS host | ✅ | ✅ | ✅ | ✅ |
| macOS with Nix | ❌ | ✅ | ❌ | ✅ |
| Linux (non-NixOS) with Nix | ❌ | ✅ | ❌ | ✅ |
| macOS without Nix | ❌ | ❌ | ❌ | ✅ |
| CI agent with Nix | ❌ | ✅ | ❌ | ✅ |

## Option Verification — Always Verify Before Suggesting

**Never suggest a NixOS option from memory alone.** Option names change between NixOS versions, and incorrect options fail silently or produce confusing errors.

**Run on the NixOS host** (not macOS, not CI unless it runs NixOS):

```bash
# Search available options interactively
nixos-option services.openssh.settings

# Evaluate option existence in current nixpkgs
nix eval nixpkgs#nixosOptionsDoc --apply 'x: builtins.attrNames x' 2>/dev/null | grep -o '"[^"]*PasswordAuth[^"]*"'

# Quickest: man page on the NixOS machine
man configuration.nix | grep -A3 "PasswordAuthentication"
```

**Or use the web** (version-specific, always available):
- `https://search.nixos.org/options` — search by option name, filterable by channel
- Check the channel matching your `nixpkgs.url` (e.g. `nixos-25.05`)

**Workflow when unsure about an option:**
1. Check `search.nixos.org/options` for the option name and correct path
2. Note the NixOS version it applies to
3. Only then include it in configuration

## Core Mental Model

NixOS is **declarative and atomic**. Every change produces a new **generation**. You can always roll back.

Key workflow:
1. Edit `.nix` files
2. `git add` (in flakes — untracked files are invisible to Nix)
3. `nixos-rebuild test` (activate without committing to bootloader)
4. `nixos-rebuild switch` (set as default boot)

## Flakes vs Channels

**Use flakes** for reproducibility. Channels are impure (machine-dependent lookup paths).

Enable flakes once:
```nix
nix.settings.experimental-features = [ "nix-command" "flakes" ];
```

## Common `nixos-rebuild` Commands

| Command | When to use |
|---|---|
| `nixos-rebuild switch` | Apply + set as default boot |
| `nixos-rebuild test` | Apply now, skip bootloader — safe first step |
| `nixos-rebuild boot` | Set as next boot without activating now |
| `nixos-rebuild dry-activate` | Preview changes without applying |
| `nixos-rebuild build` | Build only, creates `./result` |
| `nixos-rebuild build-vm` | Build QEMU VM for local testing |

Always test before switch — especially on remote servers:
```bash
nixos-rebuild test --flake .#hostname --target-host root@server
nixos-rebuild switch --flake .#hostname --target-host root@server
```

## Remote Deployment Tools

| Tool | Best for |
|---|---|
| `nixos-rebuild --target-host` | 1-3 machines, simplest |
| `deploy-rs` | Small fleet, auto-rollback on failure |
| `colmena` | Large fleet, parallel, cross-host config |
| `nixos-anywhere` | Initial install on non-NixOS machine |

## Secret Management

| Tool | When to use |
|---|---|
| **agenix** | Simple setup, SSH key workflow, small secret count |
| **sops-nix** | Cloud KMS needed, templating, multiple formats |

Secrets live at `/run/agenix/` or `/run/secrets/` — never in Nix store.

## Most Frequently Changed Elements

- `environment.systemPackages` — installed packages
- `users.users.<name>` — user accounts and SSH keys
- `networking.*` — hostname, IPs, firewall ports
- `services.*` — enable/configure systemd services
- `boot.loader.*` — bootloader and kernel settings
- `nix.settings.*` — substituters, trusted users, features

See [configuration.md](configuration.md) for patterns.
