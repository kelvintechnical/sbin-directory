# What is the /sbin Directory?

**Linux Filesystem Hierarchy Standard (FHS)** | Lab 02  
**Part of:** [Linux-Filesystem-Hierarchy-Standard](https://github.com/kelvintechnical/Linux-Filesystem-Hierarchy-Standard-)  
**Previous Lab:** [What is the /bin directory?](https://github.com/kelvintechnical/bin-directory)  
**Time Estimate:** 10–15 minutes

---

## 🎯 Objective

Understand what the `/sbin` directory is, what lives inside it, why it exists, and how it differs from `/bin` — through hands-on exploration.

---

## 🧠 Big Idea — What is /sbin?

`/sbin` stands for **System Binaries**. It contains essential administrative commands that are primarily meant to be run by the **root user** to manage and maintain the system.

> Think of `/sbin` as the **admin toolkit** — commands that regular users don't need day-to-day but the system administrator cannot live without.

**Key rule:** If a command is used to **manage the system itself** (disks, networking, boot, services), it lives in `/sbin`.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| List contents of /sbin | `ls /sbin` |
| Count how many binaries are in /sbin | `ls /sbin \| wc -l` |
| Find where an admin command lives | `which <command>` |
| Check if /sbin is a symlink | `ls -la / \| grep sbin` |
| See what type of file a binary is | `file /sbin/<command>` |
| Read what a command does | `man <command>` |

---

## 🔧 Steps

### Step 1 — List the contents of /sbin

```bash
ls /sbin
```

**What this does:**
- `ls` — list directory contents
- `/sbin` — the target directory

> You will see admin commands like: `fdisk`, `reboot`, `shutdown`, `iptables`, `ip`, `fsck`, `useradd`, `groupadd`, and more.

---

### Step 2 — Count how many binaries are in /sbin

```bash
ls /sbin | wc -l
```

**Command explained:**

| Part | Meaning |
|---|---|
| `ls /sbin` | List all files in /sbin |
| `\|` | Pipe — send that output to the next command |
| `wc -l` | Count the number of lines (one per file) |

---

### Step 3 — Find where admin commands live

```bash
which reboot
which fdisk
which useradd
```

**What this does:**
- `which` — searches your PATH and returns the full path of a command
- Confirms these commands live in `/sbin` or `/usr/sbin`

---

### Step 4 — Check if /sbin is a symlink on modern RHEL

```bash
ls -la / | grep sbin
```

**What this does:**

| Part | Meaning |
|---|---|
| `ls -la /` | List root directory with hidden files and full details |
| `\|` | Pipe output forward |
| `grep sbin` | Filter only lines containing "sbin" |

**Expected output on RHEL 9:**
```
lrwxrwxrwx.  1 root root    8 ... sbin -> usr/sbin
```

**Output explained:**

| Detail | Meaning |
|---|---|
| `lrwxrwxrwx` | `l` = this is a symbolic link (symlink) |
| `sbin -> usr/sbin` | `/sbin` is just a pointer to `/usr/sbin` |

> On modern RHEL 9, `/sbin` and `/usr/sbin` are the **same directory** — `/sbin` is a symlink to `/usr/sbin`. This is part of **UsrMerge**, the same change that unified `/bin` and `/usr/bin`.

---

### Step 5 — Explore a specific system binary

```bash
which fdisk
file /sbin/fdisk
man fdisk
```

**Command explained:**

| Command | Meaning |
|---|---|
| `which fdisk` | Shows the full path of the `fdisk` command |
| `file /sbin/fdisk` | Shows what type of file it is (ELF binary = compiled executable) |
| `man fdisk` | Opens the manual page — press `q` to quit |

---

### Step 6 — Try running an /sbin command as a regular user

```bash
reboot --help
fdisk --help
```

> Some `/sbin` commands show help without root. Actually executing them (like `reboot`) requires `sudo`. This demonstrates that `/sbin` commands are **available** to all users but **restricted** in what they can do without root privileges.

---

### Step 7 — Compare /bin vs /sbin contents

```bash
ls /bin | wc -l
ls /sbin | wc -l
```

> Compare the counts. Both directories are large — `/bin` is for general user commands, `/sbin` is for system administration commands.

---

## ✅ Lab Checklist

- [ ] `ls /sbin` shows admin system commands
- [ ] `ls /sbin | wc -l` returns a count of binaries
- [ ] `which reboot` returns `/sbin/reboot` or `/usr/sbin/reboot`
- [ ] `ls -la / | grep sbin` shows `/sbin -> usr/sbin` symlink
- [ ] `file /sbin/fdisk` confirms it is an ELF executable
- [ ] Compared `/bin` and `/sbin` counts

---

## 🧠 /bin vs /sbin — What's the Difference?

| Directory | Purpose | Who uses it | Example commands |
|---|---|---|---|
| `/bin` | Essential user commands | All users | `ls`, `cp`, `cat`, `grep` |
| `/sbin` | System administration commands | Root / sudo users | `fdisk`, `reboot`, `useradd`, `iptables` |
| On RHEL 9 | `/bin → usr/bin`, `/sbin → usr/sbin` | Both unified via UsrMerge | Same physical location |

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---|---|
| Expecting `/sbin` and `/usr/sbin` to be different on RHEL 9 | They are the same — `/sbin` is a symlink |
| Confusing `/sbin` with `/bin` | `/sbin` = admin only; `/bin` = all users |
| Running `/sbin` commands without sudo | Most require root — use `sudo <command>` |
| Deleting files from `/sbin` | Never — breaks system administration entirely |

---

## 📌 Exam Tips

- Know that on RHEL 9, `/sbin → usr/sbin` — same UsrMerge as `/bin`.
- `/sbin` commands often require `sudo` — always check with `--help` first.
- `which <command>` is the fastest way to confirm where an admin command lives.
- Common RHCSA `/sbin` commands: `useradd`, `groupadd`, `fdisk`, `ip`, `firewall-cmd`.

---

## 🔗 Series Index

| # | Directory | Repo |
|---|---|---|
| 01 | `/bin` | [bin-directory](https://github.com/kelvintechnical/bin-directory) |
| 👉 02 | `/sbin` | **You are here** |
| 03 | `/lib` | [what-is-the-lib-directory](https://github.com/kelvintechnical/what-is-the-lib-directory) |
| 04 | `/lib64` | [what-is-the-lib64-directory](https://github.com/kelvintechnical/what-is-the-lib64-directory) |
| 05 | `/usr` | [what-is-the-usr-directory](https://github.com/kelvintechnical/what-is-the-usr-directory) |
| 06 | `/etc` | [what-is-the-etc-directory](https://github.com/kelvintechnical/what-is-the-etc-directory) |
| 07 | `/boot` | [what-is-the-boot-directory](https://github.com/kelvintechnical/what-is-the-boot-directory) |
| 08 | `/home` | [what-is-the-home-directory](https://github.com/kelvintechnical/what-is-the-home-directory) |
| 09 | `/root` | [what-is-the-root-directory](https://github.com/kelvintechnical/what-is-the-root-directory) |
| 10 | `/var` | [what-is-the-var-directory](https://github.com/kelvintechnical/what-is-the-var-directory) |
| 11 | `/tmp` | [what-is-the-tmp-directory](https://github.com/kelvintechnical/what-is-the-tmp-directory) |
| 12 | `/opt` | [what-is-the-opt-directory](https://github.com/kelvintechnical/what-is-the-opt-directory) |
| 13 | `/srv` | [what-is-the-srv-directory](https://github.com/kelvintechnical/what-is-the-srv-directory) |
| 14 | `/dev` | [what-is-the-dev-directory](https://github.com/kelvintechnical/what-is-the-dev-directory) |
| 15 | `/proc` | [what-is-the-proc-directory](https://github.com/kelvintechnical/what-is-the-proc-directory) |
| 16 | `/sys` | [what-is-the-sys-directory](https://github.com/kelvintechnical/what-is-the-sys-directory) |
| 17 | `/run` | [what-is-the-run-directory](https://github.com/kelvintechnical/what-is-the-run-directory) |
| 18 | `/media` | [what-is-the-media-directory](https://github.com/kelvintechnical/what-is-the-media-directory) |
| 19 | `/mnt` | [what-is-the-mnt-directory](https://github.com/kelvintechnical/what-is-the-mnt-directory) |
| 20 | `/afs` | [what-is-the-afs-directory](https://github.com/kelvintechnical/what-is-the-afs-directory) |

---

## 🔗 Part of Linux Ops Mastery

- [Linux-Filesystem-Hierarchy-Standard](https://github.com/kelvintechnical/Linux-Filesystem-Hierarchy-Standard-)
- [Linux Ops Mastery](https://github.com/kelvintechnical/linux-ops-mastery)

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
