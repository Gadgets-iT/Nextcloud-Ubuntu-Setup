# Part 3 - Fixing Nextclou data after data disk failure using SnapRAID

1. **Replace new harddisk with the failed harddrive**

2. **Format new partition to ext4**
    ```bash
    sudo gdisk /dev/sd?
    sudo mkfs.ext4 /dev/sd?1
    ```

3. **Create a directory for new HDD**
    ```bash
    sudo mkdir /mnt/parity2
    ```

    3.1 **Identify harddrive by ID using**
    ```bash
    ls -l /dev/disk/by-id
    ```

    3.2 **Copy the ID and replace the ID with the failed disk in fstab**
    ```bash
    sudo nano /etc/fstab
    ```

    3.3 **Also replace the directory to the one that we have just created**

    3.4 **Mount the drive**
    ```bash
    sudo mount -a
    ```

    3.5 **Recheck if the drive is mounted**
    ```bash
    df -h
    ```

    3.6 **Reboot**
    ```bash
    sudo reboot
    ```

4. **Change snapraid.conf file**
    ```bash
    sudo nano /etc/snapraid.conf
    ```

    4.1 **Replace failed path to the new path**

    4.2 **Sync SnapRAID**
    ```bash
    sudo snapraid --force-full sync
    ```

    4.3 **Check SnapRAID**
    ```bash
    sudo snapraid -d d1 -a check
    ```

5. **Check data at Nextcloud**
