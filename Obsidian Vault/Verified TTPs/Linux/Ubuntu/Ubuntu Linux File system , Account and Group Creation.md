# NMB Setup Procedure

## Table of Contents

1. [[Introduction]]
2. [[Objective]]
3. [[Prerequisites]]
4. [[Procedure]]
   1. [[Create Logical Volume]]
   2. [[Format and Mount Volume]]
   3. [[Create User and Group]]
   4. [[Set Permissions]]
   5. [[Add Additional User to Group]]
5. [[Verification]]
6. [[Troubleshooting]]
7. [[References]]

## Introduction

This document outlines the procedure for setting up a new logical volume for the /nmb directory, creating associated user and group accounts, and configuring appropriate permissions.

## Objective

To allocate unused disk space to a /nmb directory, owned by a svc.nmb.cm account and group, and to add an existing user to this group.

## Prerequisites

- Ubuntu Linux system with LVM configured
- Root or sudo access
- Sufficient unallocated space in the volume group

## Procedure

### Create Logical Volume

1. Create a new logical volume using all available free space:

   ```bash
   sudo lvcreate -l 100%FREE -n nmb_lv ubuntu-vg
   ```

   - `-l 100%FREE`: Use 100% of the free space
   - `-n nmb_lv`: Name the new logical volume "nmb_lv"
   - `ubuntu-vg`: The name of the volume group

   [Screenshot: lvcreate command output]

### Format and Mount Volume

1. Format the new logical volume with ext4:

   ```bash
   sudo mkfs.ext4 /dev/ubuntu-vg/nmb_lv
   ```

   [Screenshot: mkfs.ext4 command output]

2. Create the mount point:

   ```bash
   sudo mkdir /nmb
   ```

3. Add an entry to /etc/fstab:

   ```bash
   echo '/dev/ubuntu-vg/nmb_lv /nmb ext4 defaults 0 2' | sudo tee -a /etc/fstab
   ```

   - `defaults`: Use default mount options
   - `0`: Dump (backup) frequency (0 means no backup)
   - `2`: File system check order (2 means check after root file system)

4. Mount the new volume:

   ```bash
   sudo mount /nmb
   ```

   [Screenshot: mount command output]

### Create User and Group

1. Create the new group:

   ```bash
   sudo groupadd svc.nmb.cm
   ```

2. Create the new user and add it to the group:

   ```bash
   sudo useradd -m -g svc.nmb.cm svc.nmb.cm
   ```

   - `-m`: Create the user's home directory
   - `-g svc.nmb.cm`: Set the primary group for the new user

   [Screenshot: useradd command output]

### Set Permissions

1. Set ownership of the /nmb directory:

   ```bash
   sudo chown svc.nmb.cm:svc.nmb.cm /nmb
   ```

2. Set appropriate permissions:

   ```bash
   sudo chmod 755 /nmb
   ```

   - `755`: Owner has read/write/execute, group and others have read/execute

   [Screenshot: chown and chmod command output]

### Add Additional User to Group

1. Add the wiz0rd user to the svc.nmb.cm group:

   ```bash
   sudo usermod -a -G svc.nmb.cm wiz0rd
   ```

   - `-a`: Append the user to the group (don't remove from other groups)
   - `-G svc.nmb.cm`: Specify the group to add the user to

   [Screenshot: usermod command output]

## Verification

1. Check the mount and size:

   ```bash
   df -h /nmb
   ```

2. Check permissions and ownership:

   ```bash
   ls -ld /nmb
   ```

3. Verify user and group membership:

   ```bash
   id svc.nmb.cm
   id wiz0rd
   ```

   [Screenshot: Verification command outputs]

## Troubleshooting

- If the logical volume creation fails, check available space with `vgdisplay ubuntu-vg`
- If mounting fails, ensure the /etc/fstab entry is correct
- If permission issues occur, double-check the chown and chmod commands

## References

- [[LVM Documentation]]
- [[Ubuntu User Management]]
- [[File System Hierarchy Standard]]
- [[Linux Permissions Guide]]

[LVM Documentation]: https://www.tldp.org/HOWTO/LVM-HOWTO/
[Ubuntu User Management]: https://ubuntu.com/server/docs/security-users
[File System Hierarchy Standard]: https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html
[Linux Permissions Guide]: https://www.linux.com/training-tutorials/understanding-linux-file-permissions/
