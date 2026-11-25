---
name: linux-troubleshooting
description: Diagnose Linux incidents in CPU, memory, disk, services, networking, databases, TLS, backups or deployments; also check the real unit and recovery before a routine service restart/reload.
license: MIT
metadata:
  portable: true
  compatible_with:
  - claude-code
  - codex
  author: Peter Bamuhigire
  author_url: techguypeter.com
  author_contact: '+256784464178'
---

# Linux troubleshooting

1. Establish the target host/service, symptom, onset, recent changes and access. Reuse known context. Without access, provide a collection plan and label results unverified.
2. Take the smallest read-only snapshot that distinguishes host pressure from a service failure: `uptime`, `free -h`, `df -h`, `systemctl --failed`, relevant `journalctl` logs. Check installed tools/permissions; avoid unrelated or secret-bearing dumps.
3. Read the matching [diagnosis-tree](references/diagnosis-tree.md) branch: CPU=1, memory=2, disk=3, service=4, proxy 502/504=5, slow site=6, database=7, TLS=8, backup=9, deployment=10, unreachable=11, hung/locks/tracing=12. Network paths use [network diagnostics](references/network-diagnostics.md); packet/process traces use [capture and tracing](references/packet-capture-and-tracing.md).
4. Test one evidence-backed cause. Preserve before state and a rollback path. Apply only the authorized minimal fix. Do not batch candidate remedies from a reference. Restarts, forced renewal and log/cache deletion are not automatic first aid.
5. Repeat the user-visible failure check and relevant subsystem checks. If it fails, restore where possible and reassess. Report evidence, attempted fix, confirmed recovery and gaps. Task memory follows `memory` save triggers.

Debian units commonly use `apache2`/`ssh`; RHEL uses `httpd`/`sshd`. Verify actual units/config before restart. For correct Unix modes but denied access on RHEL, inspect SELinux mode/AVCs/labels; do not disable it. Distinguish DNS, routing, listener and firewall failures; UDP silence proves no open port.

No `install-skills-bin`, `common.sh` or `sk-capture` wrapper is installed with this skill. Use available system commands; report missing privileges/tools instead of inventing results.
