# Intel MBP — Status

## 2026-06-23 — APT/DKMS repair for Linux 6.17 HWE

`apt upgrade` was blocked by `broadcom-sta-dkms` failing to build `wl.ko` for `6.17.0-29-generic` and `6.17.0-35-generic`. Patched the local DKMS source for Linux 6.17 kbuild/timer/cfg80211 changes, installed `wl.ko` for both kernels, and completed `dpkg --configure -a` / `apt-get -f install` / `apt-get upgrade`.

Current package state is clean: `apt update` exits 0, `dpkg --audit` is empty, and simulated upgrade only shows phased `kpartx` deferrals. See `docs/apt-broadcom-sta-dkms-linux-6.17-2026-06-23.md`.

## 2026-06-11 — Initial doc (post-Mint-reinstall)

**OS:** Linux Mint 22.3 · **Hostname:** intel-mbp · **User:** lg  
**Tailscale:** 100.118.98.5 · **Disk:** 50G/160G (33%) · **RAM:** 15GB

### Versions
| Package | Version | Notes |
|---------|---------|-------|
| Node.js | v18.19.1 | ⚠️ needs upgrade to v22 |
| Python | 3.12.3 | ✅ |
| bun | 1.3.14 | ✅ |
| rclone | 1.74.3 | ✅ ~/bin/rclone |
| Ollama | 0.24.0 | ⚠️ outdated |
| git | 2.43.0 | ✅ |
| Codex CLI | not installed | ❌ |

### Services (systemd --user)
| Service | Status |
|---------|--------|
| stalwart.service | installed, not active (user key seeding = blocker) |
| qbittorrent-nox.service | running |

### SSH keys
| File | Purpose |
|------|---------|
| `~/.ssh/id_ed25519` | Mesh (mbp-mesh-20260605) |
| `~/.ssh/github_intel_mbp_ed25519` | GitHub deploy key for intel-mbp-setup (write) |
| `~/.ssh/github_catalina_ed25519` | Stale Catalina-era key — keep for now |

### gbrain
Config: `~/.gbrain/config.json` → `postgresql://lg@100.77.128.17:5432/gbrain`  
Consumer-only. No autopilot, no worker. Self-upgrade: off.

### Pending / TODO
- [ ] Upgrade Node v18 → v22 (nvm or nodesource)
- [ ] Install Codex CLI (`npm i -g @openai/codex@latest` post-Node upgrade)
- [ ] Set git global user.name / user.email
- [ ] Upgrade Ollama to current
- [ ] Staltawrt: seed user keys (LGV manual step)
- [ ] Clean up stale `~/repos/intel-mb-pro-setup`
