---
title: "NFS Setup Guide"
---

## Overview

NFS (Network File System) is a distributed file system protocol that allows a user on a client computer to access files over a network much like local storage is accessed. NFS enables file sharing across different operating systems, enhancing interoperability in a mixed OS environment. This guide outlines the process of configuring NFS on Windows Server, Rocky Linux, and Ubuntu, as well as mounting these shares across the various platforms.

## Environment Information

Below is the environment setup used for this configuration, including the server types and their respective IP addresses for reference:

- **Windows NFS Server** — Windows Server 2022 → `44.101.0.131`
- **Rocky Linux NFS Server** — Rocky Linux 9.x → `44.101.2.131`
- **Ubuntu NFS Server** — Ubuntu 24.04 → `44.101.4.131`

---

## 1. Create NFS Shares

### 1.1 Create NFS Share on Windows Server

1. Open **Server Manager**
2. Click **Manage** → **Add Roles and Features**.
3. Select **File and iSCSI Services** → **Server for NFS**.
4. Click **Add Features**, then **Next**.
5. Select **Client for NFS**, click **Next** → **Install** → **Close**.

**Create NFS Folder**  

1. Open **File Explorer** → **Local Disk (C:)** → Right-click → **New Folder** → Name it `var`.
2. Right-click `var` → **New Folder** → Name it `nfs`.

**Create NFS Share**  

1. Open **Server Manager** → **File and Storage Services** → **Shares**.
2. Click **Tasks** → **New Share…** → Select **NFS Share - Quick** → **Next**.
3. Choose a custom path: `/var/nfs`.
4. Set the share name to `windowsnfs`.
5. Enable **Unmapped User Access**.
6. Add client IP addresses with **Read/Write** permissions:
    - `44.101.2.131`
    - `44.101.4.131`
7. Click **Next** → **Next** → **Create** → **Close**.

---

### 1.2 Create NFS Share on Rocky Linux

1. Install NFS package:

    ```bash
    sudo dnf install nfs-utils
    ```

2. Create NFS directory:

    ```bash
    sudo mkdir -p /var/nfs  # -p creates parent directories as needed and avoids errors if the path already exists
    sudo chown nobody /var/nfs
    ```

3. Export NFS Share:

    ```bash
    sudo vim /etc/exports
    ```

    - Press `i` to enter insert mode.
    - Add the following lines:

        ```txt
        /var/nfs 44.101.0.131(rw,sync,no_subtree_check,insecure,all_squash)
        /var/nfs 44.101.4.131(rw,sync,no_subtree_check)
        ```

    - Press `esc` → `:wq` to save and quit.

4. Allow NFS through the firewall:

    ```bash
    sudo firewall-cmd --permanent --add-service=nfs       # --permanent saves the rule across reboots; --add-service allows the named service
    sudo firewall-cmd --permanent --add-service=mountd    # --permanent keeps the rule persistent; --add-service opens mountd for NFS mount requests
    sudo firewall-cmd --permanent --add-service=rpc-bind  # --permanent keeps the rule persistent; --add-service opens rpcbind used by NFS/RPC
    sudo firewall-cmd --reload                            # --reload applies permanent firewall changes to the active runtime config
    ```

5. Enable and start NFS service:

    ```bash
    sudo systemctl enable nfs-server
    sudo systemctl start nfs-server
    ```

---

### 1.3 Create NFS Share on Ubuntu

1. Install NFS package:

    ```bash
    sudo apt install nfs-kernel-server
    ```

2. Create NFS directory:

    ```bash
    sudo mkdir -p /var/nfs  # -p creates parent directories as needed and avoids errors if the path already exists
    sudo chown nobody:nogroup /var/nfs
    ```

3. Export NFS Share:

    ```bash
    sudo vim /etc/exports
    ```

    - Press `i` to enter insert mode.
    - Add the following lines:

        ```txt
        /var/nfs 44.101.0.131(rw,sync,no_subtree_check,insecure,all_squash)
        /var/nfs 44.101.2.131(rw,sync,no_subtree_check)
        ```

    - Press `esc` → `:wq` to save and quit.

4. Enable and restart the NFS service:

    ```bash
    sudo systemctl restart nfs-kernel-server
    ```

---

## 2. Mount NFS Shares

### 2.3 Mount NFS Shares to Windows

1. Press **Windows Key** → Type `cmd` → Open Command Prompt  
2. Enter the following commands:

    ```cmd
    mount 44.101.2.131:/var/nfs R:
    mount 44.101.4.131:/var/nfs U:
    ```

---

### 2.2 Mount NFS Shares on Rocky Linux

1. Install NFS package:

    ```bash
    sudo dnf install nfs-utils
    ```

2. Mount Windows NFS share:

    ```bash
    sudo mkdir -p /mnt/windowsnfs  # -p creates parent directories as needed and avoids errors if the path already exists
    sudo mount 44.101.0.131:/windowsnfs /mnt/windowsnfs
    ```

3. Mount Ubuntu NFS share:

    ```bash
    sudo mkdir -p /mnt/ubtnfs  # -p creates parent directories as needed and avoids errors if the path already exists
    sudo mount 44.101.4.131:/var/nfs /mnt/ubtnfs
    ```

---

### 2.3 Mount NFS Shares on Ubuntu

1. Install NFS package:

    ```bash
    sudo apt install nfs-common
    ```

2. Mount Windows NFS share:

    ```bash
    sudo mkdir -p /mnt/windowsnfs  # -p creates parent directories as needed and avoids errors if the path already exists
    sudo mount 44.101.0.131:/windowsnfs /mnt/windowsnfs
    ```

3. Mount Rocky NFS share:

    ```bash
    sudo mkdir -p /mnt/rockynfs  # -p creates parent directories as needed and avoids errors if the path already exists
    sudo mount 44.101.2.131:/var/nfs /mnt/rockynfs
    ```

---

## 3. Verification

To verify that the NFS mounts are mapped on **Windows**:

1. Press **Windows Key**
2. Type `File Explorer`
3. Click **This PC**

You should see the newly mapped drives representing the NFS shares.

Run the following commands to verify that the NFS shares are mounted on **Linux**:

```bash
showmount -e 44.101.0.131  # -e lists exported NFS shares from the target host
showmount -e 44.101.2.131  # -e lists exported NFS shares from the target host
showmount -e 44.101.4.131  # -e lists exported NFS shares from the target host
```
