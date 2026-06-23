# intel-mbp-setup

Setup, configuration, and node state for Intel MacBook Pro (Linux Mint 22.3).

**Node:** intel-mbp · **Tailscale:** 100.118.98.5 · **User:** lg  
**Role:** gbrain client + enrichment consumer. Stalwart migration target.

## Files

- `STATUS.md` — living state doc (versions, services, gaps)
- `docs/` — extended notes, setup procedures
- `scripts/mbp-update` — manual APT/DKMS update helper installed as `/usr/local/bin/mbp-update`

## Manual Update Shortcut

Until Ubuntu/Mint ships the Broadcom STA Linux 6.17 fix through the normal update path, update this node with:

```bash
mbp-update
```

Useful non-mutating checks:

```bash
mbp-update --dry-run
mbp-update --check
```

The command refreshes APT metadata, preserves/reapplies the local Broadcom STA DKMS Linux 6.17 patch, runs the package upgrade, then runs DKMS and dpkg repair steps. It logs to `/var/log/mbp-update.log`.

## Mesh

Part of FUSED-ID Tailscale mesh. Connects to M4 (100.77.128.17) for gbrain DB.

## Repo

GitHub: [FUSED-ID/intel-mbp-setup](https://github.com/FUSED-ID/intel-mbp-setup)  
Deploy key: `github_intel_mbp_ed25519` (registered 2026-06-11)
