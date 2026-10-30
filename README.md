# Anthropic.claude-code 2.1.313 — compromise indicators (machine-readable)

On 29 October 2026 the official Claude Code extension for VS Code and Cursor, version
**2.1.313**, was served from the Cursor marketplace with an injected payload that plants an
SSH key, attempts to enable Remote Login, opens a reverse SSH tunnel, and beacons host
reconnaissance to an operator server. The extension auto-updates, so affected developers did
not choose the version.

The fixed build (2.1.314) **does not remove** the key, the persistence script or the
LaunchAgent. Removal is a separate per-host step, followed by credential rotation.

## What is in this repo

| File | What it is |
|---|---|
| `iocs.json` | Hashes, key fingerprints, network indicators, filesystem and process artifacts for macOS, Linux and Windows, plus removal order. One machine-readable file, CC-BY-4.0. |
| `check.sh` | Read-only host checker. Writes a dated evidence report, exits non-zero if anything is found. Nothing is uploaded. |
| `index.html` | The human-readable page these files back. |

## Use the checker

```
curl -fsSLO https://raw.githubusercontent.com/Sam-Bombay/claude-code-2.1.313-iocs/main/check.sh
bash check.sh                 # current user
bash check.sh /Users/a /home/b   # fleet mode, homes you can read
```

The single definitive marker is the planted key comment:

```
grep -qs svc-telemetry ~/.ssh/authorized_keys && echo IMPACTED || echo "key not present"
```

That grep alone misses the temp-directory tunnel key, `~/.config/.smt`, the
`com.apple.sysmond` LaunchAgent, cron persistence and a live tunnel process — which is why the
script checks all of them, and why an evidence file matters more than a grep.

## Sources

- Cursor security notice, 29 Oct 2026 — https://cursor.com/blog/claude-code-extension-security-notice
- Phorion technical advisory, 30 Oct 2026 — https://phorion.io/blog/offical-claude-code-vsx-compromise/
- Kevin Beaumont reporting, 30 Oct 2026 — https://doublepulsar.com/anthropic-hacked-to-deliver-remote-access-trojan-to-claude-developer-laptops-worldwide-1148a92baae2

Indicators are copied from those three publications. No original telemetry is claimed here, and
this repo is not affiliated with Anthropic or Cursor.

## Caveat worth repeating

On macOS the implant's attempt to enable Remote Login normally fails when it runs as a standard
user, because `systemsetup -getremotelogin` returns an access error that the payload misreads as
"already on". The key, persistence and beacon still occur, and any host where Remote Login was
already enabled was fully exposed. Do not use "the tunnel probably failed" as a reason to skip
cleanup.

Compiled by Noir & Co., 30 Oct 2026. CC-BY-4.0 — cite it and reuse it.
