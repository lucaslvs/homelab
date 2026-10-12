# bootstrap

Takes the [host](../docs/glossary.md#host) from bare metal to reachable over SSH with key-based access,
so that [`configuration/`](../configuration/) can take over.

* **Current implementation:** this runbook, followed manually.
* **Planned:** Ubuntu autoinstall files in `autoinstall/`, replacing manual steps (future ADR).

> **Status:** not written yet.

## Runbook

1. Hardware inventory
2. Firmware (BIOS/UEFI) settings
3. Ubuntu Server installation
4. Network
5. First SSH key
6. Verification from the workstation
