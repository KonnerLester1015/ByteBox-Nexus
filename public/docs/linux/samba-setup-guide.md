# Samba Setup Guide
## Overview

Samba is an open-source software suite that provides file and print services to SMB/CIFS clients. It enables seamless file and printer sharing between Linux/Unix and Windows machines. This guide outlines the process of configuring Samba shares on Windows Server, Rocky Linux, and Ubuntu, as well as mounting these shares across the various platforms.

## Environment Information

Below is the environment setup used for this configuration, including the server types and their respective IP addresses for reference:

- **Windows SMB Server** — Windows Server 2022 → `44.101.0.131`
- **Rocky Linux SMB Server** — Rocky Linux 9.x → `44.101.2.131`
- **Ubuntu SMB Server** — Ubuntu 24.04 → `44.101.4.131`

---

## 1. Create Samba Shares

### 1.1 Create SMB Share on Windows Server

1. Open **Server Manager**
2. Click **Manage** → **Add Roles and Features** → **Next** → **Next** → **Next** → **Next**.  
3. Select **SMB 1.0/CIFS File Sharing Support**.  
4. Click **Next** → **Install** → **Close**.

**Create SMB Folder**  

1. Open **File Explorer** → **Local Disk (C:)** → Right-click → **New Folder** → Name it `var`.
2. Right-click `var` → **New Folder** → Name it `smb`.

**Create SMB Share**  

1. Open **Server Manager**
2. Click **File and Storage Services** → **Shares** → **Tasks** → **New Share…**.  
3. Select **SMB Share - Quick** → **Next**.  
4. Browse to `/var/smb`, select the folder, and click **Next**.  
5. Enter `winsmb` as the share name → **Next** → **Next**.  
6. Click **Customize Permissions…** → Select **Everyone** → **Edit** → **Full Control** → **OK** → **OK**.  
7. Click **Next** → **Create** → **Close**.

---

### 1.2 Create Samba Share on Rocky Linux

1. Install Samba package:

    ```bash
    sudo dnf install samba samba-client -y
    ```

2. Create Samba directory:

    ```bash
    sudo mkdir -p /var/smb
    sudo chmod 777 /var/smb
    sudo chown nobody:nogroup /var/smb
    ```

3. Export Samba Share:

    ```bash
    sudo vim /etc/samba/smb.conf
    ```

    - Press `i` to enter insert mode.
    - Add the following lines:

        ```txt
        [rockysmb]
        path = /var/smb
        browsable = yes
        writable = yes
        read only = no
        valid users = jack
        create mask = 0777
        directory mask = 0777
        ```

    - Press `esc` → `:wq` to save and quit.

4. Allow Samba through the firewall:

    ```bash
    sudo firewall-cmd --permanent --add-service=samba
    sudo firewall-cmd --reload
    ```

5. Enable and start Samba service:

    ```bash
    sudo systemctl enable smb
    sudo systemctl enable nmb
    sudo systemctl start smb
    sudo systemctl start nmb
    ```

6. Create Samba user:

    ```bash
    sudo smbpasswd -a jack
    sudo smbpasswd -e jack
    ```

---

### 1.3 Create Samba Share on Ubuntu

1. Install Samba package:

    ```bash
    sudo apt install samba -y
    ```

2. Create Samba directory:

    ```bash
    sudo mkdir -p /var/smb
    sudo chmod 777 /var/smb
    sudo chown nobody:nogroup /var/smb
    ```

3. Export Samba Share:

    ```bash
    sudo vim /etc/samba/smb.conf
    ```

    - Press `i` to enter insert mode.
    - Add the following lines:

        ```txt
        [ubtsmb]
        path = /var/smb
        browsable = yes
        writable = yes
        read only = no
        valid users = jack
        create mask = 0777
        directory mask = 0777
        ```

    - Press `esc` → `:wq` to save and quit.

4. Enable and start Samba service:

    ```bash
    sudo systemctl enable smbd nmbd
    sudo systemctl start smbd nmbd
    ```

5. Create Samba user:

    ```bash
    sudo smbpasswd -a jack
    sudo smbpasswd -e jack
    ```

---

## 2. Mount Samba Shares

### 2.1 Mount SMB Shares to Windows

1. Press **Windows Key** → Type `cmd` → Open Command Prompt  
2. Enter the following commands:

    ```cmd
    net use Z:\\44.101.2.131\rockysmb /user:jack [secure password]
    net use Y:\\44.101.4.131\ubtsmb /user:jack [secure password]
    ```

---

### 2.2 Mount Samba Shares on Rocky Linux

1. Install CIFS utilities:

    ```bash
    sudo dnf install cifs-utils -y
    ```

2. Mount Windows Samba share:

    ```bash
    sudo mkdir -p /mnt/windowssamba
    sudo mount -t cifs //44.101.0.131/winsmb /mnt/windowssamba -o username=Administrator,password=[Secure Password]
    ```

3. Mount Ubuntu Samba share:

    ```bash
    sudo mkdir -p /mnt/ubtsamba
    sudo mount -t cifs //44.101.4.131/ubtsmb /mnt/ubtsamba -o username=jack,password=[Secure Password]
    ```

---

### 2.3 Mount Samba Shares on Ubuntu

1. Install CIFS utilities:

    ```bash
    sudo apt install cifs-utils -y
    ```

2. Mount Windows Samba share:

    ```bash
    sudo mkdir -p /mnt/windowssamba
    sudo mount -t cifs //44.101.0.131/winsmb /mnt/windowssamba -o username=Administrator,password=[Secure Password]
    ```

3. Mount Rocky Samba share:

    ```bash
    sudo mkdir -p /mnt/rockysamba
    sudo mount -t cifs //44.101.2.131/rockysmb /mnt/rockysamba -o username=jack,password=[Secure Password]
    ```

---

## 3. Verification

To verify the Samba shares are mounted on **Windows**:

1. Press **Windows Key**  
2. Type `File Explorer` → Open it  
3. Click **This PC**  
4. You should see the newly mapped drives representing the Samba shares.  

To verify the Samba shares are mounted on **Linux**:

```bash
df -h | grep smb

