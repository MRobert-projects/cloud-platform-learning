# Lab 01 Observations

Date: 2026-08-13 to 2026-08-14

## Identity and platform

- User and groups: `mrobert` (`uid=1000`, `gid=1000`), with standard desktop groups and administrative access through `sudo`.
- Environment type: WSL 2; `hostnamectl` reports WSL virtualization and a container-style chassis.
- Distribution: Ubuntu 24.04.4 LTS (Noble Numbat), derived from Debian.
- Kernel: Linux 6.18.33.2-microsoft-standard-WSL2 on x86-64.
- Key distinction between distribution and kernel: Ubuntu identifies the user-space operating-system release and package ecosystem; the kernel is the Microsoft-maintained WSL 2 Linux kernel underneath it. Their version numbers and release lifecycles are independent.

## CPU and memory

- Architecture: x86-64, little-endian, with both 32-bit and 64-bit CPU operating modes.
- Logical CPUs: 12 online CPUs from 6 physical cores with 2 threads per core; 1 socket and 1 NUMA node.
- Total memory: Approximately 7.7 GiB is currently exposed to WSL, with 2.0 GiB of swap configured.
- Available memory: Approximately 6.9 GiB; swap usage was 0 B at observation time.
- Notes: The Intel i7-9750H is exposed through the Microsoft hypervisor with VT-x/full virtualization. `MemAvailable` is more operationally useful than `MemFree` because it includes reclaimable cache. CPU vulnerability entries mostly report active mitigations or hypervisor-dependent visibility, not active security incidents.

## Filesystems and storage

- Root filesystem: `/dev/sdd`, formatted as ext4 and mounted at `/`; approximately 1007 GiB virtual capacity with about 954 GiB available at observation time.
- Windows mount: `C:` is mounted at `/mnt/c` through WSL DrvFs over the 9P (`v9fs`) protocol; `D:` is similarly mounted at `/mnt/d`.
- Notes: Linux projects remain under `/home/mrobert` on ext4 for native Linux permissions, symbolic links, case behavior, and better metadata-heavy I/O. Windows-mounted files cross the WSL/Windows filesystem boundary. The reported Linux virtual-disk capacity is not the same as immediately preallocated physical space. Windows C: and D: were approximately 82% and 83% utilized. `stat` labels ext4 as the broader `ext2/ext3` filesystem family.

## Processes and services

- PID 1: `/sbin/init`, running as root with command name `systemd`; PID 1 has PPID 0 and supervises the Linux user space.
- Highest-CPU processes: A user-owned `MainThread` process was highest at approximately 1.4% CPU in this instant snapshot; related `MainThread` processes are consistent with VS Code's WSL-side components. No sustained pressure was demonstrated by this single sample.
- systemd state: `running`.
- Failed units: None (`0 loaded units listed`). WSL-specific processes include `/init`, `plan9` for host-filesystem integration, and `wsl-pro-service`.

## Networking

- Interfaces: Loopback `lo` is available; `eth0` is up with private IPv4 address `172.25.147.125/20` and a link-local IPv6 address.
- Default route: Traffic leaves through gateway `172.25.144.1` on `eth0`.
- Listening services: DNS stub listeners use port 53 on loopback/WSL resolver addresses; time synchronization uses UDP 323 on loopback; a VS Code-related `MainThread` listener uses TCP 35061 on `127.0.0.1`. No listed application listener was bound to all interfaces.
- DNS source: `/etc/resolv.conf` is generated automatically by WSL and points to WSL's resolver at `10.255.255.254`; local systemd-resolved stubs also listen on `127.0.0.53` and `127.0.0.54`.
- Connectivity test: `getent` successfully resolved `github.com` to an IPv4 address, and an HTTPS HEAD request completed with HTTP/2 status 200. DNS resolution, routing, TLS, and outbound HTTPS were operational.

## Logs

- Expected/informational: systemd reports no failed units. The journal files were renamed and recreated after an unclean WSL shutdown; this is consistent with the earlier forced WSL restart during VS Code troubleshooting. A startup DNS lookup failed transiently, but later DNS and HTTPS tests succeeded.
- Warnings: Xwayland triggered a kernel warning in WSL's `dxgkrnl` GPU-paravirtualization path (`dxgvmb_send_wait_sync_object_gpu`). The system remained operational; monitor for repeat events or visible WSLg rendering failures. PAM logged a missing `pam_lastlog.so`; Ubuntu 24.04 no longer ships that module although the login PAM configuration may still reference it.
- Actionable findings: No immediate corrective action is required while services, networking, and GUI applications work normally. Keep WSL and Windows graphics drivers updated; investigate the `dxgkrnl` warning if it repeats with observable GUI/GPU symptoms. Do not modify PAM configuration solely to silence the known optional-module warning without a tested rollback path.

## Permissions

- SSH directory mode: `700`, owned by `mrobert:mrobert`.
- Private-key mode: `600`, owned by `mrobert:mrobert`.
- Public-key mode: `644`, owned by `mrobert:mrobert`.
- Assessment: Permissions are appropriate. The home directory is `750`, which is more restrictive than the common `755`; other users cannot list or access it. With umask `0022`, a new regular file was created as `644`, as expected.

## Summary

- What I learned: A first-response Linux assessment should establish identity and privilege, distinguish distribution from kernel, inspect resource topology and filesystems, verify service and network state, correlate warnings with symptoms, and validate permissions before making changes.
- Anomalies requiring follow-up: Monitor the WSL `dxgkrnl`/Xwayland warning only if it repeats or produces visible GPU/GUI symptoms. The optional `pam_lastlog.so` warning is a known Ubuntu 24.04 configuration mismatch and currently requires no local change.
- Commands I want to remember: `hostnamectl`, `lscpu`, `free -h`, `findmnt`, `ps`, `systemctl --failed`, `ip -brief address`, `ss -lntup`, `journalctl -b`, `namei -l`, and `stat`.
