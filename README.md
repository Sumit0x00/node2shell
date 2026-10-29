# node2shell

This repository contains a proof of concept (PoC) for a claimed Brotli
decompression vulnerability. The script labels the issue `CVE-2026-111497`;
that identifier and the vulnerability details have not been independently
verified here. Confirm affected versions and fixes with the relevant vendor
advisory before making security decisions.

## What the PoC claims

The PoC describes an invalid Brotli decoder state that can lead to a
controlled out-of-bounds write in a Node.js HTTP request-processing path. It
claims that, for a specific Node.js binary and server configuration, the write
can be used to corrupt control flow and execute a local program as the Node.js
process user. The effect could therefore range from a worker crash or denial
of service to code execution, depending on the exact build and runtime
conditions.

The included gadget offsets are tied to the particular Node.js build described
in `poc.py`; they should not be assumed to apply to other versions, builds, or
architectures. The PoC's own output and comments are not proof that a system is
vulnerable or that the claimed issue has been confirmed by a vendor.

## Safety and authorization

This is an exploit PoC, not a general-purpose diagnostic or production
scanner. Its normal execution path sends crafted HTTP request data and can
crash a service or attempt code execution. Do not run it against production,
public services, or systems you do not own or have explicit permission to
test. If you need to evaluate it, use a disposable, isolated lab with no
sensitive data and no route to other systems.

Although the script is intended to restrict targets to loopback addresses,
the current host-validation logic does not enforce that restriction correctly.
Treat it as capable of connecting to a non-loopback host if one is supplied.
Do not rely on the script itself as a safety boundary; use network isolation
and only a local lab target.

## Requirements

- Python 3.10 or later (the script uses built-in generic and union type syntax)
- No third-party packages for the script's standard mode
- `pwntools` is listed in `requirements.txt` for the optional `--autoprofile`
  feature. That feature also imports `build_profile`, which is not included in
  this repository snapshot.

## Safe command examples

These commands inspect the script or check its Python syntax; they do not send
requests to a target:

```bash
python3 --version
python3 poc.py --help
python3 -c 'import ast; ast.parse(open("poc.py", encoding="utf-8").read())'
```

`--help` lists the available command-line arguments. Avoid invoking the script
with target or execution options unless you are working in an authorized,
isolated lab and have reviewed the code and its effects. This README
intentionally does not provide a ready-to-run exploitation command.

## Defensive guidance

- Check the Node.js and framework/vendor advisories for the affected component
  and install a fixed release as soon as one is available.
- Until patched, consider disabling Brotli request-body decompression on
  affected routes, or rejecting Brotli-encoded request bodies where the
  application does not need them.
- Enforce request-body and decompression limits, and keep the service process
  restricted to the minimum privileges and filesystem/network access it needs.
- Monitor for unexpected worker restarts, crashes during Brotli request
  handling, and suspicious child-process execution.
- Validate the fix in a disposable test environment using vendor-supported
  checks; do not use this PoC against live services.

## Repository contents

- `poc.py` — the proof-of-concept script
- `requirements.txt` — optional dependency declaration for auto-profiling
- `LICENSE` — repository license
