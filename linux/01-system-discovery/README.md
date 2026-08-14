# Lab 01: Linux System Discovery

## Objective

Build a repeatable first-response workflow for understanding an unfamiliar Linux system before changing it.

## Rules

- Run commands as the normal user unless a step explicitly requires `sudo`.
- Read command output before moving on.
- Do not paste secrets, tokens, private keys, or full environment-variable dumps into this repository.
- Record conclusions, not pages of raw output, in `observations.md`.

## Checkpoint 1: Identity and platform

```bash
whoami
id
pwd
hostnamectl
cat /etc/os-release
uname -a
```

Answer:

1. Which user and groups are active?
2. Is this a physical host, VM, container, or WSL environment?
3. What is the distribution version?
4. What kernel is running, and how is it different from the distribution version?

## Checkpoint 2: CPU and memory

```bash
lscpu
free -h
cat /proc/meminfo | head -n 10
```

Answer:

1. How many logical CPUs are visible?
2. Which CPU architecture is in use?
3. How much memory is available to Linux?
4. What is the difference between free and available memory?

## Checkpoint 3: Filesystems and storage

```bash
lsblk -f
df -hT
findmnt /
findmnt /mnt/c
```

Answer:

1. Which filesystem backs `/`?
2. How is the Windows `C:` drive exposed?
3. Why should Linux projects normally remain under `/home` rather than `/mnt/c`?

## Checkpoint 4: Processes and services

```bash
ps -ef | head
ps -eo pid,ppid,user,stat,%cpu,%mem,comm --sort=-%cpu | head
systemctl is-system-running
systemctl --failed
```

Answer:

1. What is PID 1?
2. Which processes currently use the most CPU?
3. Is systemd operational?
4. Are any units failed?

## Checkpoint 5: Networking

```bash
ip -brief address
ip route
ss -lntup
cat /etc/resolv.conf
```

Answer:

1. Which interfaces and addresses exist?
2. What is the default route?
3. Which TCP or UDP ports are listening?
4. Where does DNS configuration come from in WSL?

## Checkpoint 6: Logs and recent events

```bash
journalctl -b -p warning --no-pager | tail -n 30
dmesg --level=err,warn 2>/dev/null | tail -n 30
```

Classify findings as expected, informational, warning, or actionable. WSL may restrict some kernel-log access for non-root users; record that behavior rather than bypassing it immediately.

## Checkpoint 7: Permissions

```bash
umask
ls -ld ~ ~/.ssh
ls -l ~/.ssh
namei -l ~/.ssh/id_ed25519
```

Do not display the contents of the private key.

Confirm that:

- `~/.ssh` is accessible only by its owner.
- The private key is readable and writable only by its owner.
- The public key may be world-readable.

## Completion criteria

- All seven checkpoints executed.
- `observations.md` contains concise conclusions and any anomalies.
- No secrets or machine-specific sensitive data are committed.
- Changes are reviewed with `git diff` before committing.

