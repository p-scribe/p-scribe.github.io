Day 1: I set up this page! 

---

some notes from today!

- I learnt what is Network storage (storage over network)
- I learnt on surface level, how to implement Network storage (set up NFS or SMB server, create share, broadcast them, and mount them on the receiving client endpoints, in `/etc/fstab` if needed to mount permanently)
- Also learnt, it's possible to access NFS server storage in Kubernetes clusters, with the [csi-driver-nfs](https://github.com/kubernetes-csi/csi-driver-nfs).
- There's one to access SMB server as well – [csi-driver-smb](https://github.com/kubernetes-csi/csi-driver-smb)

---

Setting up a simple Network File System (NFS) storage involves configuring both the server (where the storage is located) and the client (which accesses the storage). Here's a detailed guide for setting up NFS on a Linux system.

### Step 1: Install NFS Server

#### On Ubuntu/Debian
```sh
sudo apt update
sudo apt install nfs-kernel-server
```

#### On CentOS/RHEL
```sh
sudo yum install nfs-utils
```

### Step 2: Create a Directory to Share

```sh
sudo mkdir -p /mnt/nfs_share
sudo chown nobody:nogroup /mnt/nfs_share
sudo chmod 777 /mnt/nfs_share
```

### Step 3: Configure NFS Exports

Edit the NFS exports file to define which directories to share and the permissions.

```sh
sudo nano /etc/exports
```

Add the following line to share the directory with specific options:

```
/mnt/nfs_share 192.168.1.0/24(rw,sync,no_subtree_check)
```

- `/mnt/nfs_share`: The directory to share.
- `192.168.1.0/24`: The network range that is allowed to access the share. Replace with your network range.
- `rw`: Read and write permissions.
- `sync`: Writes are synchronized.
- `no_subtree_check`: Prevents subtree checking, which can improve performance.

### Step 4: Export the Shared Directory

```sh
sudo exportfs -a
```

### Step 5: Start and Enable the NFS Server

#### On Ubuntu/Debian
```sh
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

#### On CentOS/RHEL
```sh
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
```

### Step 6: Configure Firewall

Allow NFS service through the firewall.

#### On Ubuntu/Debian (using UFW)
```sh
sudo ufw allow from 192.168.1.0/24 to any port nfs
```

#### On CentOS/RHEL (using Firewalld)
```sh
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --reload
```

### Step 7: Install NFS Client

#### On Ubuntu/Debian
```sh
sudo apt update
sudo apt install nfs-common
```

#### On CentOS/RHEL
```sh
sudo yum install nfs-utils
```

### Step 8: Mount the NFS Share on the Client

1. Create a mount point:

    ```sh
    sudo mkdir -p /mnt/nfs_clientshare
    ```

2. Mount the NFS share:

    ```sh
    sudo mount 192.168.1.100:/mnt/nfs_share /mnt/nfs_clientshare
    ```

    Replace `192.168.1.100` with the IP address of your NFS server.

3. To make the mount permanent, add it to the `/etc/fstab` file:

    ```sh
    sudo nano /etc/fstab
    ```

    Add the following line:

    ```
    192.168.1.100:/mnt/nfs_share /mnt/nfs_clientshare nfs defaults 0 0
    ```

### Step 9: Verify the NFS Share

On the client, you can check if the NFS share is mounted correctly:

```sh
df -h /mnt/nfs_clientshare
```

You should see an entry for the NFS share. You can now read and write to `/mnt/nfs_clientshare` as if it were a local directory.

### Additional Notes

- **Security Considerations:** NFS does not provide encryption, so it's best used within a trusted network. For secure setups, consider using NFS with Kerberos.
- **Performance Tuning:** For performance optimization, you can tune NFS mount options such as `rsize`, `wsize`, and others based on your needs.

That's it! You now have a simple NFS setup for sharing files between systems on your network.
