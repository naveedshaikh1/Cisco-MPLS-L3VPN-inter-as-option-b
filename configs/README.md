# Complete router configurations

`R1.cfg` through `R10.cfg` contain the complete candidate configurations for the supplied tasks. They are newly reconstructed, not original historical configuration exports. They have not been router-tested.

Use an isolated clean lab with Cisco IOS-compatible MPLS/VPNv4, OSPF VRFs and classic EIGRP VRF support. Review [tasks and commands](../docs/tasks-and-commands.md) before applying. Verify Ethernet interface names, paste only the matching device file from privileged EXEC mode, and check every command for parser errors.

The files include `enable`, `configure terminal`, and `end`; they are paste-ready CLI sequences rather than emulator-native project files. They do not automatically save startup configuration. After successful verification, run `copy running-config startup-config`.

For historical evidence, place original sanitized exports in a separate `original-exports/` folder with their genuine dates and software versions. Never include credentials or proprietary router images.
