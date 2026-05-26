# Lab: What Is the `/sbin` Directory?

- **Series:** linux-ops-mastery — Linux Filesystem Hierarchy Standard
- **Subjects covered:** Filesystem Hierarchy Standard (FHS), `/sbin` purpose for essential system-administration binaries, contrast with `/bin`, UsrMerge symlink (`/sbin → /usr/sbin`), the `$PATH` and `secure_path` distinction between root and normal users, SUID admin tools (`mount`, `passwd`-class), inspecting privileged commands with `file` and `stat`
- **Career arcs covered:** RHCSA (every `systemctl`, `firewall-cmd`, `fdisk`, `mount` lives under `/sbin`), RHCE (Ansible `become: true` modules call into `/sbin` and `/usr/sbin`), SRE (incident triage: `ss`, `ip`, `tc` all live here), DevOps (container images that drop `/sbin` lose firewall and disk management capability), AI/MLOps (privileged GPU init containers need `/sbin/modprobe` and `/sbin/ldconfig`)
- **Prerequisite:** Basic `ls`, `cd`, `cat`, and a shell on RHEL 9 / Rocky 9 / Ubuntu
- **Time Estimate:** 20 to 35 minutes
- **Difficulty arc:** Task 1 inspect · 2–3 inventory + read · 4–5 demonstrate purpose · 6 capstone audit

---

## Objective

Treat `/sbin` as the **administrator's toolbox** in the FHS — distinct from `/bin` (user commands) by audience, not by mechanism. By the end of this lab you can list its contents, prove it is a symlink to `/usr/sbin` on modern RHEL, contrast its membership with `/bin`, explain why a normal user's `$PATH` may not include it, and identify SUID administrative tools that allow regular users to perform privileged operations safely.

The lab is **inspection-only**. You will not `chmod`, install, or relocate anything. You will look at binaries every RHCSA candidate uses dozens of times during the exam — `fdisk`, `mkfs`, `reboot`, `mount`, `iptables`, `ip`, `ss`, `lvm` — and understand exactly why they live in a separate directory from `ls` and `cat`.

The capstone is the RHCSA-realistic audit prompt: *"Produce a one-page report of `/sbin` showing the symlink target, the count of administrative binaries, the subset that are SUID-root (and therefore callable by non-root with full power), and the impact on `$PATH` for a normal user shell."*

---

## Concept: Why `/sbin` Exists

```
   ┌────────────────────────────────────────────────────────────────┐
   │  FHS root /                                                    │
   ├────────────────────────────────────────────────────────────────┤
   │  /bin    → essential USER commands (ls, cp, cat, bash)         │
   │  /sbin   → essential ADMIN commands (mount, fsck, ip, reboot)  │
   │  /usr/bin   → non-essential user commands                      │
   │  /usr/sbin  → non-essential admin commands                     │
   │                                                                │
   │  Audience matters more than location:                          │
   │     /bin  + /usr/bin   → expected on every user's $PATH        │
   │     /sbin + /usr/sbin  → root-only by default on many distros  │
   │                                                                │
   │  Modern RHEL (UsrMerge):                                       │
   │     /sbin ── symlink ──► /usr/sbin                             │
   └────────────────────────────────────────────────────────────────┘
```

`/sbin` holds the binaries the **system administrator** uses to bring the system to a working state and to keep it there. FHS lists the essential ones explicitly: `fdisk`, `fsck`, `getty`, `halt`, `ifconfig` (now `ip`), `init`, `mkfs`, `mkswap`, `reboot`, `route`, `swapoff`, `swapon`, `shutdown`. The point is not that normal users *cannot* run them — many can read and even invoke them — but that they are not what a logged-in user normally needs.

> **Why this matters:** On many distros, the **default user `$PATH` does not include `/sbin` or `/usr/sbin`** while the **root `$PATH` does**. That is why `ifconfig` "works for root but not for me" — same binary, different shell environment. RHEL 9 actually unifies the path for all users, but the historic split still trips up interview candidates every week.

---

## 📜 Why `/sbin` Exists — The Story

When UNIX expanded to multi-user time-sharing systems in the 1970s, the operators noticed something practical: end users had no business running `fdisk` or `mkfs`. Those commands could destroy the very disks the users' homes lived on. Bell Labs convention separated commands by **risk and audience** — user-facing binaries in `/bin`, administrator-facing binaries in `/etc` originally, then moved to `/sbin` ("system bin") by the early BSD releases. The "s" stood for **system** or **superuser**, depending on which manual page you read.

Linux adopted the `/sbin` convention from day one. The **Filesystem Hierarchy Standard**, first published in **1994** by the linux-fhs group and maintained today by the Linux Foundation, codified the rule: `/sbin` is for "system binaries" — programs essential for booting, restoring, recovering, and repairing the system, in addition to those in `/bin`. It also formalized the parallel: `/usr/sbin` for non-essential admin commands, just as `/usr/bin` is for non-essential user commands.

The Fedora **UsrMove** of 2012 (UsrMerge to the wider world) folded `/sbin` into `/usr/sbin` the same way `/bin` was folded into `/usr/bin`. RHEL 7 followed in 2014, and every modern RHEL, Rocky, AlmaLinux, Fedora, and Arch system ships `/sbin` as a symlink. Crucially, RHEL 9 also **unified the default `PATH`** in `/etc/bashrc` and `/etc/profile` so that ordinary users get `/sbin:/usr/sbin` appended automatically — though `sudo` still uses a separate `secure_path` defined in `/etc/sudoers`.

> **The point of the story:** `/sbin` survives as a **semantic label** even though the bytes live in `/usr/sbin`. It tells you "this is an admin command" without inspecting permissions. The directory name is a hint to the human reading the path, not a kernel-enforced boundary.

---

## 👪 The `/sbin` Family — Who Lives There

### Essential admin binaries by category

| Category | Examples | Purpose |
|---|---|---|
| Storage | `fdisk`, `parted`, `mkfs.ext4`, `mkfs.xfs`, `mkswap`, `lvm` | Create / inspect / resize disks and filesystems |
| Filesystem repair | `fsck`, `fsck.ext4`, `fsck.xfs`, `e2fsck`, `xfs_repair` | Run after crashes; required for rescue boots |
| Mount / swap | `mount`, `umount`, `swapon`, `swapoff` | Bring filesystems and swap online |
| Network | `ip`, `ss`, `tc`, `arp`, `route`, `iptables`, `nft` | Configure interfaces, routing, firewall |
| Power / boot | `reboot`, `shutdown`, `halt`, `poweroff`, `init`, `runlevel` | Init system control |
| Module / device | `modprobe`, `insmod`, `rmmod`, `lsmod`, `udevadm` | Kernel module and udev management |
| Service control | `service`, `chkconfig` (legacy compat) | Old SysV interfaces, often symlinked to systemd |
| User admin | `useradd`, `userdel`, `groupadd`, `chpasswd` | Account lifecycle (note: `/usr/sbin`) |

### Related directories you will visit

| Directory | Purpose | Relation to `/sbin` |
|---|---|---|
| `/bin` | Essential user binaries | Sibling — covered in the `/bin` lab |
| `/usr/sbin` | Non-essential admin binaries | Symlink target of `/sbin` on UsrMerge |
| `/usr/local/sbin` | Locally-installed admin binaries | First in root's `$PATH` by convention |
| `/etc/sudoers` | Defines `secure_path` for `sudo` | Controls which `/sbin` paths apply when escalating |
| `/lib64` | Shared libraries loaded by `/sbin/*` | Required for any dynamic binary to run |

### Tools that interact with `/sbin`

| Tool | What it tells you about `/sbin` |
|---|---|
| `ls -l /sbin` | Symlink vs directory |
| `readlink /sbin` | Symlink target (`usr/sbin`) |
| `file /sbin/fdisk` | ELF type, architecture |
| `stat /sbin/mount` | Permissions including SUID bit |
| `find /sbin -perm -4000` | Surface all SUID-root admin binaries |
| `rpm -qf /sbin/mkfs.xfs` | Owning package (`xfsprogs`) |
| `sudo -l` | Which `/sbin` paths `sudo` will search |

> **The point of the family tree:** Every admin binary lives at a specific FHS address with a specific package owner and a specific permission set. Knowing the address, owner, and bits is the difference between "run `dnf reinstall`" and "boot rescue media."

---

## 🔬 The Anatomy of `ls -l /sbin` — In One Diagram

```
$ ls -l /sbin
lrwxrwxrwx. 1 root root 8 Apr 12  2024 /sbin -> usr/sbin
 │          │ │    │    │  │            │      │
 │          │ │    │    │  │            │      └─ Target: relative path "usr/sbin"
 │          │ │    │    │  │            └─ The path being inspected
 │          │ │    │    │  └─ mtime of the symlink itself (RPM install time)
 │          │ │    │    └─ Size in bytes (length of target string)
 │          │ │    └─ Group owner
 │          │ └─ Owner (root)
 │          └─ Link count (1)
 └─ File type + permissions:
      l → symbolic link
      rwxrwxrwx → mode of the symlink (kernel ignores; only target's perms matter)
      .         → SELinux context attached
```

Compare to a SUID admin binary inside the target:

```
$ ls -l /sbin/mount
-rwsr-xr-x. 1 root root 60768 Mar 22  2024 /sbin/mount
 │          │ │    │    │     │            │
 │          │ │    │    │     │            └─ Path
 │          │ │    │    │     └─ mtime
 │          │ │    │    └─ Size (real ELF)
 │          │ │    └─ Group
 │          │ └─ Owner: root
 │          └─ Link count
 └─ `-rwsr-xr-x`
       │ │
       │ └─ The `s` in the owner-execute slot → SUID bit set:
       │    "When a normal user runs this, the kernel sets euid=root."
       └─ regular file, world-readable + executable
```

> **Reading rule:** Mount needs to call kernel syscalls only root can issue. SUID is what lets a normal user invoke `mount` from a desktop session without `sudo`. Every SUID binary under `/sbin` is a deliberate, audited concession.

---

## 📚 `/sbin` Reference Table

| Task | Command | Notes |
|---|---|---|
| See if `/sbin` is a symlink | `ls -ld /sbin` | First char is `l` on UsrMerge |
| Print symlink target | `readlink /sbin` | Returns `usr/sbin` |
| List contents | `ls /sbin/` | Trailing slash to follow link |
| Count binaries | `ls /sbin/ \| wc -l` | Typically 250-400 on RHEL 9 |
| Find SUID admin tools | `find /usr/sbin -perm -4000 -type f` | Use `/usr/sbin` to avoid symlink |
| Show your `$PATH` | `echo $PATH` | Check for `/sbin` and `/usr/sbin` |
| Show sudo's `secure_path` | `sudo grep -i secure_path /etc/sudoers` | Independent of user `$PATH` |
| Find owning package | `rpm -qf /sbin/fdisk` | Returns `util-linux-core-...` |
| Identify binary type | `file /sbin/fdisk` | ELF 64-bit binary |
| Show library deps | `ldd /sbin/mount` | Bridge to `/lib64` lab |

> **Rule one of `/sbin`:** If a command is **not in your `$PATH`** but root can run it, it almost certainly lives in `/sbin` or `/usr/sbin`. Try `/sbin/<name>` or `/usr/sbin/<name>` explicitly before assuming it is not installed.

---

## 🎯 Career Pathway Sidebar

| Level | Why this lab matters |
|---|---|
| **RHCSA candidate** | Every exam objective involving storage, networking, services, users, or boot resolves to a binary under `/sbin` or `/usr/sbin`. |
| **RHCE candidate** | Ansible's `command:`, `shell:`, and built-in modules all rely on root being able to find admin binaries. `become_method: sudo` uses `secure_path`. |
| **SRE / Platform** | Production triage commands (`ss`, `ip`, `tc`, `iptables`, `nstat`) all live here — knowing the path is part of the muscle memory. |
| **DevOps** | Distroless and `ubi-minimal` images strip `/sbin` aggressively. Knowing which admin tools you lose helps you decide whether to add them back. |
| **AI / MLOps** | NVIDIA GPU operator init containers run `nvidia-modprobe` and `ldconfig` — both in `/sbin`. |

---

## 🔧 The 6 Tasks

> Six inspection-only phases that build the **identify → enumerate → contrast → permission → audit** habit for `/sbin`.

---

### Task 1 — Inspect `/sbin` itself

**Purpose:** Confirm whether `/sbin` is a directory or a symlink on your RHEL 9 system and capture the metadata you will reference throughout the lab.

```bash
ls -ld /sbin
stat /sbin
readlink /sbin
file /sbin
readlink -f /sbin
```

**Human-Readable Breakdown:** `ls -ld` keeps the symlink line, `stat` prints the full inode entry, `readlink` returns just the target, `file` classifies the path, and `readlink -f` canonicalizes the symlink chain to its absolute resolved path.

**Reading it left to right:** `stat /sbin` will end with a `File: /sbin -> usr/sbin` indicator on UsrMerge. `readlink` returns the raw target string (`usr/sbin`, relative). `readlink -f` resolves to `/usr/sbin` (absolute).

**The story:** Every audit begins by establishing ground truth. The five commands above answer "is this directory what I think it is, where does it really point, and what does the kernel think of it" in one breath.

**Expected output:**

```text
lrwxrwxrwx. 1 root root 8 Apr 12  2024 /sbin -> usr/sbin
  File: /sbin -> usr/sbin
  Size: 8         	Blocks: 0          IO Block: 4096   symbolic link
Device: fd00h/64768d	Inode: 1572866     Links: 1
Access: (0777/lrwxrwxrwx)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2026-05-26 09:14:11.000000000 -0400
Modify: 2024-04-12 03:24:18.000000000 -0400
Change: 2024-04-12 03:24:18.000000000 -0400
 Birth: 2024-04-12 03:24:18.000000000 -0400
usr/sbin
/sbin: symbolic link to usr/sbin
/usr/sbin
```

**Switches**

| Token | Meaning |
|---|---|
| `ls -ld` | Long format, do not descend |
| `stat` | Inode metadata + symlink target |
| `readlink` | Raw target string |
| `readlink -f` | Canonicalized full path |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `/sbin` shows as directory | Non-UsrMerge distro (e.g. very old RHEL 6) — fine, lab continues |
| `Permission denied` on `stat` | SELinux denial — `ausearch -m avc -ts recent` |
| `readlink -f` returns nothing | Broken symlink — recreate from rescue media only |

---

### Task 2 — Inventory the binaries inside `/sbin`

**Purpose:** Measure the **size and shape** of `/sbin`: total count, alphabetical head, and the iconic admin commands you should recognize on sight.

```bash
ls /sbin/ | head -n 20
ls /sbin/ | wc -l
ls /sbin/ | grep -E '^(fdisk|mkfs.xfs|mount|reboot|shutdown|iptables|ip|ss|modprobe|ldconfig)$'
ls -l /sbin/fdisk /sbin/mount /sbin/reboot /sbin/ip
```

**Human-Readable Breakdown:** First 20 entries to scan the alphabetical head, total count, anchored regex for the iconic names, and long-format metadata for four widely-used admin binaries.

**Reading it left to right:** `ls /sbin/` follows the symlink and lists target contents. `grep -E '^(a\|b\|c)$'` ensures exact matches. `ls -l` on four binaries shows mode bits — including any SUID flags.

**The story:** A default RHEL 9 server install ships somewhere between 250 and 400 binaries under `/sbin`. A `ubi-minimal` container often has fewer than 30. The first thing you do on a new system or image is `ls /sbin | wc -l` so you can spot "this image cannot fdisk" before you need to fdisk.

**Expected output:**

```text
accessdb
addgnupghome
addpart
agetty
alternatives
applygnupgdefaults
arpd
arping
audispd
auditctl
auditd
augenrules
authselect
badblocks
biosdecode
blkdeactivate
blkdiscard
blkid
blockdev
blogd
312
fdisk
iptables
ip
ldconfig
modprobe
mount
mkfs.xfs
reboot
shutdown
ss
-rwxr-xr-x. 1 root root 130608 Mar 22  2024 /sbin/fdisk
-rwsr-xr-x. 1 root root  60768 Mar 22  2024 /sbin/mount
-rwxr-xr-x. 1 root root  82768 Mar 22  2024 /sbin/reboot
-rwxr-xr-x. 1 root root 692520 Mar 22  2024 /sbin/ip
```

**Switches**

| Token | Meaning |
|---|---|
| `ls /sbin/` | Follow symlink, list contents |
| `wc -l` | Count entries |
| `grep -E` | Extended regex with anchors |
| `ls -l` | Long mode listing (reveals SUID) |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `ls: cannot access '/sbin/'` | Filesystem corruption — boot rescue media |
| Count is very small (< 30) | Minimal container — install `util-linux iproute systemd` |
| Binary missing from `grep` filter | Packaged differently on Ubuntu — try `command -v <name>` |

---

### Task 3 — Contrast `/sbin` membership with `/bin`

**Purpose:** Demonstrate the **audience boundary** between user binaries (`/bin`) and admin binaries (`/sbin`) by computing the set difference.

```bash
comm -23 <(ls /usr/sbin/ | sort -u) <(ls /usr/bin/ | sort -u) | head -n 20
comm -12 <(ls /usr/sbin/ | sort -u) <(ls /usr/bin/ | sort -u) | head
ls /usr/sbin/ | wc -l
ls /usr/bin/ | wc -l
```

**Human-Readable Breakdown:** `comm -23 a b` shows lines only in file `a`. `comm -12 a b` shows lines in both. With the directory listings sorted, this gives you the binaries that are **only** in `/usr/sbin` (true admin tools), the ones that are in **both** (typically `chkconfig`-style legacy or shared utilities), and the total counts.

**Reading it left to right:** Process substitution `<(...)` feeds the sorted listing as a file. `comm` is a set-comparison tool. `head` truncates the output. The two `wc -l` commands give you the cardinality of each set so you can reason about ratios.

**The story:** On RHEL 9 you should expect roughly **300 binaries in `/usr/sbin` and 1000+ in `/usr/bin`**. The intersection is small (a few legacy compat shims). That ratio reflects the FHS audience split: many more commands serve users than serve administrators.

**Expected output:**

```text
accessdb
addgnupghome
addpart
agetty
applygnupgdefaults
arpd
audispd
auditctl
auditd
augenrules
authselect
badblocks
biosdecode
blkdeactivate
blkdiscard
blkid
blockdev
blogd
btrfs
btrfs-convert
chkconfig
mkfs
service
312
1024
```

**Switches**

| Token | Meaning |
|---|---|
| `comm -23 A B` | Lines only in A |
| `comm -12 A B` | Lines in both A and B |
| `<(...)` | Process substitution (Bash) |
| `sort -u` | Sort + unique |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `comm` complains files not sorted | Re-pipe through `sort -u` |
| Empty intersection | Distro packages `chkconfig`-style shims into other dirs |
| Counts identical to `/bin` listing | You ran `ls /bin/` instead of `/usr/bin/` — same dir, but verify |

---

### Task 4 — Sudo `$PATH` and `secure_path` effects

**Purpose:** See exactly what changes between **your shell's `$PATH`**, **root's `$PATH`**, and **`sudo`'s `secure_path`** — because that is where `/sbin` reachability is decided.

```bash
echo "USER:  $USER"
echo "PATH:  $PATH"
echo "---"
sudo -i bash -c 'echo "USER:  $USER"; echo "PATH:  $PATH"'
echo "---"
sudo grep -E '^(Defaults\s+secure_path|Defaults\s+env_keep)' /etc/sudoers
echo "---"
type ip || echo "ip not in user PATH"
sudo type ip || echo "ip not in sudo PATH"
```

**Human-Readable Breakdown:** Print your normal-user `$PATH`, then root's `$PATH` via `sudo -i`, then the `secure_path` directive that `sudo` enforces, and finally check whether `ip` resolves in both contexts.

**Reading it left to right:** `sudo -i` runs a login shell as root and inherits root's `~/.bash_profile`. `secure_path` in `/etc/sudoers` is what `sudo somecmd` (without `-i`) actually uses — it deliberately ignores the calling user's `$PATH` to avoid path-injection attacks. `type` is a shell builtin that reports whether a name is a file, alias, function, or not found.

**The story:** On a long-standing UNIX habit, this is where users new to RHEL stumble: `sudo` does not inherit `$PATH`. Even if your shell has `/sbin` in `$PATH`, `sudo` substitutes its own `secure_path`. RHEL 9 ships a `secure_path` that includes `/sbin` and `/usr/sbin` — but stripped-down container images or hardened sudoers may not.

**Expected output:**

```text
USER:  kelvin
PATH:  /home/kelvin/.local/bin:/home/kelvin/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
---
USER:  root
PATH:  /root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
---
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
---
ip is /usr/sbin/ip
ip is /usr/sbin/ip
```

**Switches**

| Token | Meaning |
|---|---|
| `sudo -i` | Login shell as root (inherits root's profile) |
| `secure_path` | Sudo-only `$PATH` defined in `/etc/sudoers` |
| `type CMD` | Resolve a command name in current shell |
| `env_keep` | Variables preserved from caller across `sudo` |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `ip` not found as normal user | Your distro pre-RHEL 9 — prepend `/sbin:/usr/sbin` to `PATH` in `~/.bashrc` |
| `sudo` ignores your custom `PATH` | That is `secure_path` working as designed; edit `/etc/sudoers` with `visudo` only |
| `Defaults env_keep` missing entries | Add via `visudo`; never edit `/etc/sudoers` directly |

---

### Task 5 — Identify SUID administrative tools

**Purpose:** Surface every binary under `/usr/sbin` (and `/usr/bin`) that has the **SUID bit** set so it runs with `euid=0` regardless of who invokes it — these are the audited backdoors that let non-root users still mount disks, change passwords, etc.

```bash
find /usr/sbin -xdev -type f -perm -4000 -printf '%M %u %p\n' 2>/dev/null
echo "---"
find /usr/bin -xdev -type f -perm -4000 -printf '%M %u %p\n' 2>/dev/null
echo "---"
stat -c '%A %U %n' /usr/bin/passwd /usr/bin/sudo /usr/bin/mount /usr/sbin/userhelper 2>/dev/null
```

**Human-Readable Breakdown:** Walk `/usr/sbin` with `find -perm -4000` (SUID bit), print mode + owner + path, then walk `/usr/bin` for comparison (where most SUID binaries actually live), then `stat` a handful of well-known SUID tools to compare modes side by side.

**Reading it left to right:** `-xdev` keeps `find` on the same filesystem (no crossing mount points). `-perm -4000` matches any file where the SUID bit (mode 4000) is set, regardless of other bits. `-printf '%M %u %p\n'` formats mode + owner + path. `stat -c '%A %U %n'` formats the same three fields for arbitrary paths.

**The story:** A clean RHEL 9 system has a **small, deliberate SUID set** — usually fewer than 20 binaries total across `/usr/bin` and `/usr/sbin`. Each one is a CVE waiting to happen, so distros review them carefully. `passwd` is SUID so users can update `/etc/shadow`. `sudo` is SUID to launch the policy engine as root. `mount` is SUID so desktop users can mount USB drives. **An unexpected SUID binary is a major red flag.**

**Expected output:**

```text
-rwsr-xr-x root /usr/sbin/userhelper
-rwsr-xr-x root /usr/sbin/pam_timestamp_check
-rwsr-xr-x root /usr/sbin/unix_chkpwd
-rwsr-xr-x root /usr/sbin/grub2-set-bootflag
-rwsr-xr-x root /usr/sbin/mount.nfs
---
-rwsr-xr-x root /usr/bin/su
-rwsr-xr-x root /usr/bin/passwd
-rwsr-xr-x root /usr/bin/chage
-rwsr-xr-x root /usr/bin/gpasswd
-rwsr-xr-x root /usr/bin/newgrp
-rwsr-xr-x root /usr/bin/sudo
-rwsr-xr-x root /usr/bin/mount
-rwsr-xr-x root /usr/bin/umount
---
-rwsr-xr-x root /usr/bin/passwd
-rwsr-xr-x root /usr/bin/sudo
-rwsr-xr-x root /usr/bin/mount
-rwsr-xr-x root /usr/sbin/userhelper
```

**Switches**

| Token | Meaning |
|---|---|
| `find -perm -4000` | SUID bit set (any other bits OK) |
| `-xdev` | Stay on one filesystem |
| `-printf '%M %u %p\n'` | Mode, owner, path |
| `stat -c '%A %U %n'` | Same format via `stat` |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Empty output from `find` | Cap dropped or filesystem mounted `nosuid` — check `mount` |
| Unexpected SUID binary | `rpm -V <owning-pkg>` to verify; treat as incident if unverified |
| `Permission denied` errors | Pipe `2>/dev/null` to silence; you are not root |

---

### Task 6 — Capstone: `/sbin` admin-binary audit report

**Task statement:** *"Produce a one-page audit of `/sbin` showing the symlink target, the count of administrative binaries, the subset that are SUID-root, the impact on `$PATH` for a normal user shell, and one concrete repair scenario."*

**Purpose:** Combine inspection (Tasks 1–5) into a single deliverable that an interviewer or auditor could read.

```bash
echo "=== /sbin Admin Audit on $(hostname) at $(date -Is) ==="
echo
echo "--- 1. Resolution ---"
ls -ld /sbin
echo "Canonical: $(readlink -f /sbin)"
echo
echo "--- 2. Binary count ---"
printf "Total /usr/sbin entries: %s\n" "$(ls /usr/sbin/ | wc -l)"
printf "Of those that are ELF:   %s\n" "$(file /usr/sbin/* 2>/dev/null | grep -c ELF)"
echo
echo "--- 3. SUID-root admin binaries ---"
find /usr/sbin /usr/bin -xdev -type f -perm -4000 -user root -printf '  %p\n' 2>/dev/null | sort
echo
echo "--- 4. PATH reachability ---"
if echo "$PATH" | tr ':' '\n' | grep -q '^/sbin\|^/usr/sbin'; then
  echo "  ✅ /sbin reachable from non-root \$PATH"
else
  echo "  ⚠️  /sbin NOT in non-root \$PATH — workaround: /sbin/<cmd> or sudo"
fi
echo
echo "--- 5. Repair scenario ---"
echo "If /sbin symlink lost: run from rescue or absolute paths:"
echo "    /usr/bin/ln -s usr/sbin /sbin"
echo "    /usr/bin/ls -l /sbin"
```

**Human-Readable Breakdown:** Five-section report. Section 1 prints resolution. Section 2 counts binaries and ELF subset. Section 3 lists SUID-root binaries the system is currently trusting. Section 4 checks whether the current shell can reach `/sbin` via `$PATH`. Section 5 records the repair one-liner using absolute `/usr/bin/...` paths.

**Reading it left to right:** `printf` formats labeled lines. `file ... | grep -c ELF` counts how many entries are actual binaries (vs. shell scripts or symlinks). `tr ':' '\n'` splits `$PATH` into one-per-line. The repair section uses `/usr/bin/ln` (absolute) because if `/sbin` is broken you cannot rely on `$PATH` resolution.

**The story:** This is the audit you hand to a senior or paste into an incident ticket. It establishes ground truth in ~30 seconds, highlights any unexpected SUID binaries, and documents the exact recovery command — all without writing anything to disk.

**Expected verification output:**

```text
=== /sbin Admin Audit on rhel9.lab at 2026-05-26T15:42:11-04:00 ===

--- 1. Resolution ---
lrwxrwxrwx. 1 root root 8 Apr 12  2024 /sbin -> usr/sbin
Canonical: /usr/sbin

--- 2. Binary count ---
Total /usr/sbin entries: 312
Of those that are ELF:   268

--- 3. SUID-root admin binaries ---
  /usr/bin/chage
  /usr/bin/gpasswd
  /usr/bin/mount
  /usr/bin/newgrp
  /usr/bin/passwd
  /usr/bin/su
  /usr/bin/sudo
  /usr/bin/umount
  /usr/sbin/grub2-set-bootflag
  /usr/sbin/mount.nfs
  /usr/sbin/pam_timestamp_check
  /usr/sbin/unix_chkpwd
  /usr/sbin/userhelper

--- 4. PATH reachability ---
  ✅ /sbin reachable from non-root $PATH

--- 5. Repair scenario ---
If /sbin symlink lost: run from rescue or absolute paths:
    /usr/bin/ln -s usr/sbin /sbin
    /usr/bin/ls -l /sbin
```

**Cleanup**

```bash
# nothing destructive — this lab was inspection-only
# no files written, no symlinks changed, no packages touched
echo "audit complete — /sbin untouched"
```

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Section 4 reports `/sbin not in PATH` | Add `export PATH=$PATH:/sbin:/usr/sbin` to `~/.bashrc` |
| Section 3 finds unexpected SUID | Compare to `rpm -Va` and treat as incident |
| `file /usr/sbin/* 2>/dev/null` is slow | Normal on large installs — wait or pipe through `head` |

---

## 🔍 `/sbin` Decision Guide

```
Looking for an admin command on RHEL 9?
  │
  ├── "Disk / filesystem? (fdisk, mkfs, fsck, lvm)"
  │       └── ✅ /sbin/<name>            (resolves to /usr/sbin/<name>)
  │
  ├── "Network? (ip, ss, iptables, nft, tc)"
  │       └── ✅ /sbin/<name>
  │
  ├── "Service control? (systemctl, service, chkconfig)"
  │       └── ✅ /usr/bin/systemctl       (systemd in /usr/bin — not /sbin)
  │
  ├── "User / group? (useradd, userdel, groupadd)"
  │       └── ✅ /usr/sbin/<name>
  │
  ├── "Boot / power? (reboot, shutdown, poweroff)"
  │       └── ✅ /sbin/<name>             (also symlinked into systemd)
  │
  └── "I'm a normal user, command not found"
          └── ✅ Try /sbin/<cmd> with absolute path,
                 or check $PATH for /sbin and /usr/sbin
```

---

## ✅ Lab Checklist (6 Tasks)

- [ ] 01 Inspect `/sbin` with `ls -ld`, `stat`, `readlink`, and `file`
- [ ] 02 Inventory `/sbin` binaries and identify iconic admin tools
- [ ] 03 Contrast `/usr/sbin` vs `/usr/bin` membership with `comm`
- [ ] 04 Examine your `$PATH`, root's `$PATH`, and sudo's `secure_path`
- [ ] 05 Enumerate SUID-root admin binaries with `find -perm -4000`
- [ ] 06 Produce the one-page `/sbin` admin audit report

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| Assuming `/sbin` always in `$PATH` | "command not found" as normal user | Use absolute `/sbin/<cmd>` or fix `~/.bashrc` |
| Editing `/etc/sudoers` directly | Sudo refuses to run | Always use `visudo` |
| Confusing `/sbin` and `/usr/sbin` | Looking for binary in wrong place | They are the same on UsrMerge |
| Treating SUID as bad by default | Removing critical tools | SUID is sometimes correct; audit, do not delete |
| Stripping `/sbin` from container image | Lose `mount`, `ip`, `modprobe` | Keep `util-linux` and `iproute` packages |
| Running `ldd` on untrusted admin binary | Code execution risk | Use `readelf -d` or `objdump -p` instead |

---

## 🎯 Career & Interview Strategy

**RHCSA candidate**
- Memorize where the boot, storage, network, and user-admin tools live. On the exam, calling `/sbin/fdisk` by absolute path is faster than fixing `$PATH`.

**RHCE candidate**
- In playbooks, set `ansible_become: true` and never assume `/sbin` in `$PATH`. Use full paths or `command: /usr/sbin/<x>` in role tasks targeting hardened sudoers.

**SRE / Platform interview**
- "Cron job ran as `nobody` and couldn't find `ip`" → walk through cron's empty `$PATH`, prepend `/sbin:/usr/sbin`, or use absolute paths.

**DevOps**
- Justify which admin binaries a base image needs. A web-app container does not need `fdisk` but may need `mount` for `tmpfs` volumes.

**AI / MLOps**
- GPU driver init containers run `nvidia-modprobe` and `ldconfig` from `/sbin`. A stripped image breaks GPU initialization in obvious ways.

---

## 🔗 Related Labs

| Lab | Connection |
|---|---|
| [/bin](https://github.com/kelvintechnical/bin-directory) | Sibling — essential user commands |
| [/lib](https://github.com/kelvintechnical/lib-directory) | Shared libraries `/sbin/*` depend on |
| [/lib64](https://github.com/kelvintechnical/lib64-directory) | 64-bit libraries — every `/sbin` binary links here |
| [/usr](https://github.com/kelvintechnical/usr-directory) | Canonical home for `/sbin` after UsrMerge |
| [/etc](https://github.com/kelvintechnical/etc-directory) | `/etc/sudoers`, `/etc/fstab` consumed by `/sbin` tools |
| [/boot](https://github.com/kelvintechnical/boot-directory) | Boot loader writes here via `/sbin/grub2-mkconfig` |
| [/home](https://github.com/kelvintechnical/home-directory) | Created by `/usr/sbin/useradd` |
| [/root](https://github.com/kelvintechnical/root-directory) | The first home with `/sbin` in default `$PATH` |
| [/var](https://github.com/kelvintechnical/var-directory) | Audit logs of `/sbin` invocations |
| [/tmp](https://github.com/kelvintechnical/tmp-directory) | Scratch space for admin tools |
| [/opt](https://github.com/kelvintechnical/opt-directory) | Third-party admin tools |
| [/srv](https://github.com/kelvintechnical/srv-directory) | Service data initialized by admin scripts |
| [/dev](https://github.com/kelvintechnical/dev-directory) | Device nodes that admin tools open |
| [/proc](https://github.com/kelvintechnical/proc-directory) | `/sbin/sysctl` writes here |
| [/sys](https://github.com/kelvintechnical/sys-directory) | `/sbin/udevadm` interacts here |
| [/run](https://github.com/kelvintechnical/run-directory) | PID files and sockets created by daemons |
| [/media](https://github.com/kelvintechnical/media-directory) | Auto-mounted by tools in `/sbin` |
| [/mnt](https://github.com/kelvintechnical/mnt-directory) | Manual mount points used by `/sbin/mount` |
| [/afs](https://github.com/kelvintechnical/afs-directory) | AFS mount managed by admin tools |

---

## 👤 Author

**Kelvin R. Tobias**
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
