---
title: "Create Least Privileged User"
---
## Introduction
To ensure security and follow best practices, it is important to create a user with the least privilege necessary. In the following procedures, a user account will be created to interact with an RBD pool that houses virtual machine images. This document covers creating the user, applying capabilities (Ceph's terminology for permissions), and creating a keyring file for the user account. This account will be used on client machines to run and interact with the environment without the possibility of using administrator privileges.

## Procedure [^1]

[^1]: [ADDING A USER](https://docs.ceph.com/en/quincy/rados/operations/user-management/#adding-a-user)

1. Open terminal of a Ceph host.
2. Run `get-or-create` command.
   ```bash {filename="bash"}
   ceph auth get-or-create client.qemu mon 'profile rbd' osd 'profile rbd pool=rbd' mgr 'profile rbd pool=rbd'
   ```
   ***This command creates a user (client.qemu) with read/write permissions to the RBD pool.***
   
3. Create user keyring file.
   ```bash {filename="bash"}
   ceph auth get client.qemu -o /etc/ceph/ceph.client.qemu.keyring
   ```
4. Add the keyring to client machine.
   
   ***If the client machine has the client.admin keyring, run the following command:***
   ```bash {filename="bash"}
   ceph auth get client.qemu -o /etc/ceph/ceph.client.qemu.keyring
   ```

   ***If the client machine doesn't have the client.admin keyring, retrieve it using SCP:***
   ```bash {filename="bash"}
   scp root@host01.example.com:/etc/ceph/ceph.client.qemu.keyring /etc/ceph
   ```

## Veriy permission
To ensure the new user has access to the specified RBD pool, run the following command to retrieve the images on the pool:

```bash {filename="bash"}
rbd ls --id=qemu
```