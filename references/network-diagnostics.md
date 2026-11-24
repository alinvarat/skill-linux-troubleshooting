# Network diagnostics

Check each command exists before using it; some probes require optional packages or privileges. Substitute the intended interface/host/port before running. Preserve the symptom, evidence and authorized rollback scope.

## Read-Only OSI-Layer Network Diagnostic Workflow

Adapted from ECC's `agents/network-troubleshooter.md` (imported 2026-09-20).
That file is an *agent* definition; this engine has no `agents/` directory
and does not use agent files as a distribution mechanism, so the pattern is
adopted here as a workflow within this skill instead — the substance
(layer-by-layer diagnosis, read-only until a fix is explicitly approved) is
what mattered, not the agent packaging.

Use this workflow for any network-path symptom — Branch 11 ("Can't reach the
server") and the `tcpdump` half of Branch 12 in
[`references/diagnosis-tree.md`](diagnosis-tree.md) are instances
of it. It supersedes ad hoc "just ping it" diagnosis: **work down (or up)
the OSI stack in order, collect evidence at each layer before concluding,
and never apply a mutating fix while still diagnosing.**

### Rule: read-only until the cause is confirmed

Every command in this workflow is observation, not remediation. If a command
would change state (restarting a service, flushing a route, disabling an
ACL, deleting a firewall rule "to test"), it is a **remediation step** and
must be labelled as such, proposed separately, and approved before running —
never folded into the diagnostic pass. This mirrors the standing rule this
skill already applies to config changes generally (state → propose → apply →
verify), applied specifically to the diagnostic phase of a network incident.

### Layer 1/2 — Physical and data link

```bash
ip -c link                                    # link state, MAC, MTU per interface
ethtool <iface> 2>/dev/null | grep -E 'Link detected|Speed|Duplex'
dmesg -T | grep -iE 'link (up|down)|nic' | tail -20
```

If a switch or router is in the path, the equivalent read-only layer-1/2
evidence is `show interfaces <interface> status`, `show vlan brief`, and
`show spanning-tree` — see
`cisco-ios-patterns` (optional external specialist; not installed here).

### Layer 3 — Network (addressing and routing)

```bash
ip -c addr show dev <iface>                   # address assigned?
ip -c route                                   # routing table
ip route get <destination>                    # which route/iface will this use?
ping -c 3 $(ip route | awk '/default/{print $3; exit}')   # gateway reachable?
ping -c 3 1.1.1.1                             # external reachable by IP (skips DNS)
mtr -c 10 --report <destination>              # where does the path actually stop?
```

Router/switch equivalent: `show ip interface brief`, `show ip route
<prefix>`, `show ip protocols` — same skill reference as above.

### Layer 4 — Transport (port reachability)

```bash
ss -tulnp | grep ':<port>'                    # is anything local listening?
timeout 3 bash -c 'echo > /dev/tcp/"$1"/"$2"' _ HOST PORT 2>/dev/null && echo OK || echo FAIL
nc -u -z -v <host> <port>                     # probe only; no reply does not prove UDP reachability
mtr -c 10 -T -P <port> --report <host>        # TCP-mode mtr to the exact port
```

### DNS (when IP connectivity works but names fail)

```bash
resolvectl query <name>                       # what the host actually sees
dig @1.1.1.1 <name> +short                    # bypass local resolver
dig @8.8.8.8 <name> +short
```

If local and public resolvers disagree, the fault is the local resolver path
— see `linux-network-admin` (optional external specialist; not installed here).
If the DNS server in the path is Pi-hole specifically, see
`linux-dns-server`'s Pi-hole reference (optional external specialist; not installed here).

### Application/policy layer (firewalls, ACLs)

```bash
sudo iptables -L -n -v | head -40             # or nft list ruleset
sudo journalctl -k --since "10 min ago" | grep -i 'DROP\|REJECT'
```

Router/switch ACL evidence is read-only counters and logs, never removing
the policy to test:

```text
show ip access-lists <name>
show logging | include <interface>|ACL|DENY|DROP
```

Full detail and the "why not to test by disabling a rule" rule:
`cisco-ios-patterns` § ACL Placement Review (optional external specialist; not installed here)
and
`network-config-validation` (optional external specialist; not installed here)
for validating a candidate ACL/config change before it is even proposed.

### Output format

Close every pass through this workflow with an evidence-backed summary, not
just a fix:

```text
## Diagnosis: <one-line likely root cause>

Symptom: <reported failure>
Layer: <where the fault was found>

Evidence:
- `<command>` -> <what it proved>
- `<command>` -> <what it ruled out>

Root cause: <specific explanation>

Recommended fix (remediation, not diagnosis):
1. <safe action to schedule, with rollback>

Verification:
- `<command>` should show <expected result>
```

---

