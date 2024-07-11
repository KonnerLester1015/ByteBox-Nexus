---
title: "Attaching RBD Storage Pool to Virt-Manager"
---
## Overview
This guide provides step-by-step instructions on how to attach an RBD (RADOS Block Device) storage pool to Virt-Manager using Ceph. You'll learn how to create a secret, define it, and set its value, as well as how to create and start a Ceph pool XML for use with Virt-Manager. Following these steps will enable seamless integration of Ceph storage with your virtualization environment.


## Create Secret [^1]

[^1]: [CONFIGURING THE VM - Ceph Docs](https://docs.ceph.com/en/quincy/rbd/libvirt/#configuring-the-vm)

1. Generate a secret.
    ```bash {filename="bash"}
    cat > secret.xml <<EOF
    <secret ephemeral='no' private='no'>
            <usage type='ceph'>
                    <name>client.admin secret</name>
            </usage>
    </secret>
    EOF
    ```
2. Define the secret.
   ```bash {filename="bash"}
   sudo virsh secret-define --file secret.xml
   ```
   **Output**
   ```
   <uuid of secret is output here>
   ```
3. Save the key string to a file.
   ```bash {filename="bash"}
   ceph auth get-key client.admin | sudo tee client.admin.key
   ```
4. Set the UUID of the secret
   ```bash {filename="bash"}
   sudo virsh secret-set-value --secret <uuid of secret is output here> --base64 $(cat client.admin.key) && rm client.admin.key secret.xml
   ```

## Create Ceph Pool XML [^3]
1. Navigate to **/etc/libvirt/storage**
2. Create new pool **rbd.xml** (nano, vim, etc) [^2]
   ```xml {linenos=table,linenostart=1,filename="rbd.xml"}
    <pool type="rbd">
    <name>rbd</name>
    <source>
        <name>rbd</name>
        <host name='host01.example.com' port='6789'/>
        <auth username='admin' type='ceph'>
        <secret uuid='{uuid of secret}'/>
        </auth>
    </source>
    </pool>
   ```
3. Create a persistent definition of storage pool.
   ```bash {filename="bash"}
   virsh pool-define rbd.xml
   ```
4. Start the RBD storage pool.
   ```bash {filename="bash"}
   virsh pool-start rbd
   ```
5. Verfy RBD pool shows active.
   ```bash {filename="bash"}
   virsh pool-list
   ```

[^1]: [CONFIGURING THE VM - Ceph Docs](https://docs.ceph.com/en/quincy/rbd/libvirt/#configuring-the-vm)
[^2]: [Libvert RBD Storage](https://libvirt.org/storage.html#rbd-pool)
[^3]: [Virsh Storage Pool Commands](https://www.ibm.com/docs/en/linux-on-systems?topic=commands-storage-pool)
