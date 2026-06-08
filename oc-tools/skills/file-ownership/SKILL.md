---
name: file-ownership
description: >
  Enforce non-root file and folder ownership on every file operation. Use this skill whenever
  Claude creates, edits, moves, or copies any file or directory — including writing code files,
  generating output files, running bash commands that produce files, or using create_file /
  str_replace tools. Trigger any time files are touched in any way. The skill checks ownership
  after every operation and fixes it if root owns the file.
---

# File Ownership Safety

## Rule: No root-owned files or folders

After **every** file or directory operation — `create_file`, `str_replace`, `bash_tool` commands
that write files, `cp`, `mv`, `mkdir`, etc. — Claude MUST verify that the resulting path is NOT
owned by root and fix it if it is.

---

## Step-by-step

### 1. After any file/folder operation, check ownership

```bash
stat -c "%U:%G %n" /path/to/file_or_dir
```

Or for a whole directory tree:

```bash
find /path/to/dir -user root -o -group root 2>/dev/null
```

### 2. If root-owned, fix it immediately

Determine the correct owner — the user running the shell session:

```bash
# Who is the current non-root user?
logname 2>/dev/null || who am i | awk '{print $1}' || echo $SUDO_USER || stat -c "%U" /home
```

Then fix ownership:

```bash
# Single file
chown user:user /path/to/file

# Recursive (directory)
chown -R user:user /path/to/dir
```

### 3. Common safe owner targets (in priority order)

| Context | Command |
|---|---|
| Running as a specific user | `chown $(logname):$(logname) <path>` |
| Inside a project dir | Match the owner of the parent directory: `stat -c "%U:%G" <parent>` |
| Web server files | Typically `www-data:www-data` (nginx/apache) — confirm with user |
| Docker / CI | Match the image's expected user |

### 4. When in doubt, ask

If you can't determine the right owner (e.g. in a container with no login user), ask the user:
> "What user/group should own the created files? I want to make sure they're not root-owned."

---

## Quick reference: one-liner post-operation check

```bash
# After creating/editing FILE:
stat -c "%U:%G" FILE && { [[ "$(stat -c '%U' FILE)" == "root" ]] && chown $(logname 2>/dev/null || echo $USER):$(logname 2>/dev/null || echo $USER) FILE && echo "Fixed root ownership on FILE"; } || echo "Ownership OK"
```

---

## Don't forget

- This applies to **output files** too (e.g. files written to `/mnt/user-data/outputs/`)
- This applies when `sudo` or elevated bash is used mid-task
- If a whole directory is created as root, fix the directory AND its contents recursively
