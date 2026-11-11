# Test Results

This document records the access-control and permission tests performed during the Shared Linux Workspace project.

All tests were conducted on an Amazon EC2 Linux instance using the `sudo -u` command to execute operations as different users.

## 1. User and Group Verification

**Objective:** Confirm that each user belongs to the correct departmental group.

| User | Expected Group | Result |
|---|---|---|
| Alice | engineering | Passed |
| John | engineering | Passed |
| Mary | finance | Passed |
| David | finance | Passed |
| Sarah | hr | Passed |
| Peter | hr | Passed |

Membership was verified using `id`, `groups`, and `getent group`.

## 2. Directory Ownership and Permission Verification

**Objective:** Confirm that each departmental directory is owned by root and assigned to its respective group, with setgid enabled.

Command used:

```bash
sudo ls -ld /clouddesk /clouddesk/*
```

Observed output:

```text
drwxr-xr-x 5 root root        50 /clouddesk
drwxrws--- 2 root engineering 38 /clouddesk/engineering
drwxrws--- 2 root finance     39 /clouddesk/finance
drwxrws--- 2 root hr          40 /clouddesk/hr
```

**Result: PASSED**

The departmental directories had the expected group ownership and `drwxrws---` permissions. The `s` confirmed that the setgid bit was enabled.

## 3. Departmental Access Tests

**Objective:** Verify that authorised users can access their own departmental directories and that users from other departments are denied access.

| Test | Expected Behaviour | Actual Result | Status |
|---|---|---|---|
| Alice accesses Engineering | Access granted | Access granted | Passed |
| John accesses Engineering | Access granted | Access granted | Passed |
| Mary accesses Finance | Access granted | Access granted | Passed |
| David accesses Finance | Access granted | Access granted | Passed |
| Sarah accesses HR | Access granted | Access granted | Passed |
| Peter accesses HR | Access granted | Access granted | Passed |
| Alice accesses Finance | Access denied | Permission denied | Passed |
| Mary accesses Engineering | Access denied | Permission denied | Passed |
| Peter accesses Engineering | Access denied | Permission denied | Passed |

**Result: PASSED**

Users could access their own departmental workspaces but could not access restricted directories belonging to other departments.

## 4. Setgid Group Inheritance Test

**Objective:** Confirm that files created within a departmental directory inherit its group ownership.

A file created by John in the Engineering directory was inspected using:

```bash
sudo -u john ls -l /clouddesk/engineering
```

Observed output:

```text
-rw-r--r-- 1 john  engineering 0 john.txt
-rw-r--r-- 1 alice engineering 0 test.txt
```

**Result: PASSED**

Files created by different users inherited the Engineering group, demonstrating the effect of the setgid bit.

## 5. ACL and File Collaboration Tests

**Objective:** Verify that members of the same department can collaborate on files, including files created by another department member.

### Initial issue

John created a file with the following permissions:

```text
-rw-r--r--
```

Alice initially received a `Permission denied` error when attempting to append content to the file.

Although both users belonged to the Engineering group, the file's original permissions did not grant group write access.

### ACL configuration

Added an ACL entry to grant the Engineering group read and write access:

```bash
sudo setfacl -m g:engineering:rw- /clouddesk/engineering/john.txt
```

### Final collaboration test

Alice attempted to append content:

```bash
sudo -u alice sh -c 'echo "Alice is contributing to this file" >> /clouddesk/engineering/john.txt'
```

The command completed successfully.

Verified the contents:

```bash
sudo cat /clouddesk/engineering/john.txt
```

Observed output:

```text
Alice is contributing to this file
```

Verified the ACL:

```text
# file: /clouddesk/engineering/john.txt
# owner: john
# group: engineering
user::rw-
group::r--
group:engineering:rw-
mask::rw-
other::r--
```

**Result: PASSED**

Alice successfully modified John's file using the additional group ACL permissions.

## 6. Default ACL Inheritance Test

**Objective:** Confirm that new files inherit the departmental ACL configuration.

Created a new file as Alice:

```bash
sudo -u alice touch /clouddesk/engineering/alice-new.txt
```

Inspected its ACL:

```bash
sudo getfacl -p /clouddesk/engineering/alice-new.txt
```

Observed output:

```text
# file: /clouddesk/engineering/alice-new.txt
# owner: alice
# group: engineering
user::rw-
group::rwx                      #effective:rw-
group:engineering:rwx           #effective:rw-
mask::rw-
other::---
```

**Result: PASSED**

The new file inherited the default ACL entries from its parent directory.

The effective group permissions were `rw-` rather than `rwx` because the file was created without execute permissions. This is expected behaviour for a regular file created using `touch`.

## 7. Final Project Verification

| Requirement | Status |
|---|---|
| Create departmental user accounts | Passed |
| Create departmental groups | Passed |
| Assign users to groups | Passed |
| Create departmental directories | Passed |
| Configure directory ownership | Passed |
| Apply `2770` permissions | Passed |
| Enable setgid group inheritance | Passed |
| Restrict access between departments | Passed |
| Configure ACLs | Passed |
| Configure default ACL inheritance | Passed |
| Allow shared file collaboration | Passed |
| Verify file and directory permissions | Passed |

## Conclusion

All planned implementation and verification tests were completed successfully.

The project demonstrated how Linux users, groups, directory ownership, standard permissions, setgid, and Access Control Lists work together to manage access to shared resources.

One key lesson was that directory access and file access are separate permission checks. Configuring a shared directory alone does not guarantee that every member can modify every existing file within it. ACLs provide a way to manage those additional collaboration requirements.
