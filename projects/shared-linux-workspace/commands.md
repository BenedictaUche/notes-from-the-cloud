# Implementation Commands

This document contains the Linux commands used to build the Shared Linux Workspace on an Amazon EC2 Linux instance.

## 1. Create Departmental Groups

Created one Linux group for each department.

```bash
sudo groupadd engineering
sudo groupadd finance
sudo groupadd hr
```

**Explanation:** `groupadd` creates a new Linux group. These groups are used to manage access to departmental directories.

## 2. Create User Accounts

Created six individual user accounts.

```bash
sudo useradd -m alice
sudo useradd -m john
sudo useradd -m mary
sudo useradd -m david
sudo useradd -m sarah
sudo useradd -m peter
```

**Explanation:** `useradd` creates a Linux user account. The `-m` option creates a home directory for each user.

## 3. Assign Users to Departmental Groups

Added each user to their respective department group.

```bash
sudo usermod -aG engineering alice
sudo usermod -aG engineering john

sudo usermod -aG finance mary
sudo usermod -aG finance david

sudo usermod -aG hr sarah
sudo usermod -aG hr peter
```

**Explanation:** `usermod -aG` adds a user to a supplementary group without removing their existing supplementary group memberships.

## 4. Verify User and Group Memberships

```bash
getent group engineering
getent group finance
getent group hr

groups alice
groups john
groups mary
groups david
groups sarah
groups peter
```

**Explanation:** `getent group` displays group information, while `groups` displays the groups a user belongs to.

## 5. Create Departmental Directories

Created the main workspace and its departmental subdirectories.

```bash
sudo mkdir -p /clouddesk/engineering
sudo mkdir -p /clouddesk/finance
sudo mkdir -p /clouddesk/hr
```

**Explanation:** `mkdir` creates directories. The `-p` option creates parent directories when necessary and avoids errors if the directories already exist.

## 6. Configure Directory Ownership

Assigned each departmental directory to its corresponding group.

```bash
sudo chown root:engineering /clouddesk/engineering
sudo chown root:finance /clouddesk/finance
sudo chown root:hr /clouddesk/hr
```

**Explanation:** `chown` changes file or directory ownership. In this case, `root` remains the owner, while each department's group receives ownership of its directory.

## 7. Configure Directory Permissions

Applied permission mode `2770` to all departmental directories.

```bash
sudo chmod 2770 /clouddesk/engineering
sudo chmod 2770 /clouddesk/finance
sudo chmod 2770 /clouddesk/hr
```

**Understanding 2770:**

- `2`: Enables the setgid bit.
- `7`: Gives the owner read, write, and execute permissions.
- `7`: Gives the group read, write, and execute permissions.
- `0`: Gives other users no permissions.

The setgid bit ensures that new files and directories inherit the parent directory's group.

## 8. Verify Directory Ownership and Permissions

```bash
sudo ls -ld /clouddesk /clouddesk/*
```

Expected departmental directory permissions:

```text
drwxrws--- root engineering /clouddesk/engineering
drwxrws--- root finance     /clouddesk/finance
drwxrws--- root hr          /clouddesk/hr
```

The `s` in the group permission section indicates that setgid is enabled.

## 9. Configure Access Control Lists (ACLs)

ACLs were configured to support collaboration among members of each department.

### Engineering

```bash
sudo setfacl -m g:engineering:rwx /clouddesk/engineering
sudo setfacl -d -m g:engineering:rwx /clouddesk/engineering
sudo setfacl -d -m g::rwx /clouddesk/engineering
sudo setfacl -d -m o::--- /clouddesk/engineering
```

### Finance

```bash
sudo setfacl -m g:finance:rwx /clouddesk/finance
sudo setfacl -d -m g:finance:rwx /clouddesk/finance
sudo setfacl -d -m g::rwx /clouddesk/finance
sudo setfacl -d -m o::--- /clouddesk/finance
```

### HR

```bash
sudo setfacl -m g:hr:rwx /clouddesk/hr
sudo setfacl -d -m g:hr:rwx /clouddesk/hr
sudo setfacl -d -m g::rwx /clouddesk/hr
sudo setfacl -d -m o::--- /clouddesk/hr
```

**Explanation:**

- `setfacl` modifies Access Control Lists.
- `-m` modifies the existing ACL.
- `-d` configures a default ACL for newly created items.
- `g:` specifies a group entry.
- `o::---` denies access to other users.

The default ACL allows newly created files and directories to inherit the configured departmental access rules, subject to the permissions requested when they are created.

## 10. Grant Engineering Group Access to an Existing File

During testing, Alice initially could not modify a file created by John because the file's original permissions did not allow group write access.

An ACL was added to the existing file:

```bash
sudo setfacl -m g:engineering:rw- /clouddesk/engineering/john.txt
```

**Explanation:** This grants the Engineering group read and write access to John's existing file.

## 11. Verify ACL Configuration

```bash
sudo getfacl -p /clouddesk/engineering
sudo getfacl -p /clouddesk/engineering/john.txt
```

To inspect a newly created file:

```bash
sudo getfacl -p /clouddesk/engineering/alice-new.txt
```

**Explanation:** `getfacl` displays the ACL entries associated with a file or directory.

## 12. Test Departmental Access

The `sudo -u` command allows commands to be executed as another user.

### Test authorised access

```bash
sudo -u alice touch /clouddesk/engineering/test.txt
sudo -u mary touch /clouddesk/finance/test.txt
sudo -u sarah touch /clouddesk/hr/test.txt
```

### Test unauthorised access

```bash
sudo -u mary ls /clouddesk/engineering
sudo -u alice ls /clouddesk/finance
sudo -u peter ls /clouddesk/engineering
```

The unauthorised access attempts should return `Permission denied`.

## 13. Test File Collaboration

Tested whether Alice could modify a file owned by John.

```bash
sudo -u alice sh -c 'echo "Alice is contributing to this file" >> /clouddesk/engineering/john.txt'
```

Verified the file contents:

```bash
sudo cat /clouddesk/engineering/john.txt
```

Verified the file ACL:

```bash
sudo getfacl -p /clouddesk/engineering/john.txt
```

The test succeeded. Alice was able to write to John's file because the Engineering group had the required ACL permissions.

## 14. Useful Linux Commands Used

| Command | Purpose |
|---|---|
| `groupadd` | Creates a group |
| `useradd` | Creates a user |
| `usermod` | Modifies user account properties |
| `groups` | Displays group memberships |
| `getent` | Retrieves account and group information |
| `mkdir` | Creates directories |
| `chown` | Changes ownership |
| `chmod` | Changes file permissions |
| `ls -ld` | Displays directory details |
| `setfacl` | Modifies ACL entries |
| `getfacl` | Displays ACL entries |
| `sudo -u` | Runs a command as another user |
| `cat` | Displays file contents |

---

**Note:** These commands document the implementation and tests performed in the project environment. They are intended as a reproducible learning reference, not as a complete production security configuration.
