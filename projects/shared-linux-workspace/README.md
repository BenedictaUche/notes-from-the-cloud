# Shared Linux Workspace

A practical Linux administration project built as part of my AWS re/Start learning journey.

## Project Overview

This project demonstrates how to create and manage a shared Linux workspace for a company with multiple departments. Each department has its own users, group, and directory, with permissions configured to control access to files and folders.

The goal was to simulate a simple workplace environment where employees can collaborate within their departments while keeping other departments' files restricted.

I implemented and tested the environment on an Amazon EC2 Linux instance.

## Scenario

CloudDesk is a fictional company with three departments:

| Department | Users |
|---|---|
| Engineering | Alice, John |
| Finance | Mary, David |
| HR | Sarah, Peter |

Each department needs a shared directory that its members can access. Users should not be able to access directories belonging to other departments.

Department members should also be able to collaborate on files within their shared directory.

## Directory Structure

The workspace uses the following directory structure:

```text
/clouddesk/
├── engineering/
├── finance/
└── hr/
```

Each department directory is owned by `root` and assigned to its corresponding Linux group.

## Linux Concepts Applied

The project involved the following Linux administration concepts:

- **User management:** Creating individual Linux user accounts.
- **Group management:** Creating departmental groups and assigning users to them.
- **File ownership:** Assigning directory ownership to the appropriate groups.
- **File permissions:** Using `chmod` to control read, write, and execute access.
- **Setgid:** Using the setgid bit on departmental directories to ensure newly created files inherit the directory's group.
- **Access Control Lists (ACLs):** Granting group members additional permissions to collaborate on existing files.
- **Default ACLs:** Configuring permissions that are inherited by newly created files and directories.
- **Access verification:** Testing that authorised users can access their department's workspace while unauthorised users are denied.

## Implementation

### 1. User and Group Creation

Created six individual Linux accounts and three departmental groups.

Users were assigned to their respective groups using `usermod -aG`.

### 2. Directory Creation and Ownership

Created the three departmental directories under `/clouddesk`.

Each directory was assigned to its corresponding departmental group.

### 3. Permission Configuration

Applied permission mode `2770` to each departmental directory.

This provides:

- Full permissions to the directory owner.
- Full permissions to members of the directory's group.
- No permissions to other users.
- Setgid behaviour, so new files and directories inherit the directory's group.

### 4. ACL Configuration

Configured ACLs to support collaboration among department members.

Named group ACLs provide the Engineering, Finance, and HR groups with the required access. Default ACLs ensure that new files and directories inherit the configured access rules.

### 5. Access Testing

Tested access using `sudo -u` to execute commands as different users.

Verified both successful access within departments and denied access across departments.

## Project Results

The implementation was tested successfully.

- Engineering users could access and collaborate within the Engineering directory.
- Finance users could access their own directory but were denied access to Engineering.
- HR users could access their own directory but were denied access to other departments.
- Users from different departments could not access restricted departmental directories.
- New files inherited the configured departmental ACL.
- Alice successfully wrote to a file owned by John, demonstrating shared file access.

## Key Learning

One important issue encountered during implementation was that setting permissions on a shared directory does not automatically make every existing file writable by all department members.

For example, a file created by John initially had permissions that prevented Alice from modifying it. Although both users belonged to the Engineering group, the file's permissions still restricted collaboration.

I addressed this by configuring an ACL on the existing file and default ACLs on the directory.

This helped me understand the difference between directory permissions, file permissions, ACLs, and default ACL inheritance.

## Environment

- Cloud platform: Amazon Web Services (AWS)
- Compute service: Amazon EC2
- Operating system: Linux (Amazon Linux)
- Shell: Bash
- Tools: `useradd`, `groupadd`, `usermod`, `mkdir`, `chown`, `chmod`, `setfacl`, `getfacl`, `sudo`

## Repository Contents

- [`commands.md`](commands.md): Commands used to implement the project, with explanations.
- [`test-results.md`](test-results.md): Access-control tests and their outcomes.

## Conclusion

This project provided practical experience with Linux user and group administration, file permissions, and access control.

It also reinforced the importance of testing permissions from the perspective of different users rather than relying only on configuration commands.
