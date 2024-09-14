# Server Setup Instructions

1. **Replace new harddisk with the failed harddrive**

2. **Install new Ubuntu server**

3. **Remove old SSH key**
    ```bash
    ssh-keygen -R 192.168.x.x
    ```

4. **Install mergerfs fuse**
    ```bash
    sudo apt install mergerfs fuse
    ```

5. **Put disk and parity mount and merge all partitions**
    5.1 **Create folders**
    ```bash
    sudo mkdir /mnt/disk{1,2,3}
    sudo mkdir /mnt/parity{1,2}
    sudo mkdir /mnt/storage
    ```

    5.2 **See disk ID**
    ```bash
    ls -l /dev/disk/by-id
    ```

    5.3 **Edit fstab file**
    ```bash
    sudo nano /etc/fstab
    ```

    ```plaintext
    //-------- within the fstab file----------//
    //-- make sure which one is parity disk and which one is data disk --//

    /dev/disk/by-id/ata-WDC_WD10JPVX-22JC3T0_WD-WX71E23LLS35-part1 /mnt/parity1 ext4 defaults 0 0
    /dev/disk/by-id/ata-ST9120821AS_5PL2KZ9G-part1 /mnt/disk1 ext4 defaults 0 0

    /mnt/disk* /mnt/storage fuse.mergerfs direct_io,defaults,allow_other,minfreespace=50G,fsname=mergerfs 0 0
    //---------------------------------//
    ```

    5.4 **Mount all the disks**
    ```bash
    sudo mount -a
    df -h
    ```

6. **Nextcloud**
    6.1 **Install Nextcloud**
    ```bash
    sudo snap install nextcloud
    ```

    6.2 **Check Nextcloud and start the system**

7. **Change Directory**
    ```bash
    sudo snap connect nextcloud:removable-media
    sudo chown -R root:root /mnt/storage
    sudo snap stop nextcloud
    sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php
    ```

    ```plaintext
    //---
    'trusted_domains' =>'
        array(
            0 =>'*'
        )',
    'datadirectory'=>'/mnt/storage/data',
    //----
    ```

    ```bash
    sudo snap start nextcloud
    ```

8. **Install SnapRAID**
    ```bash
    sudo apt install snapraid
    sudo nano /etc/snapraid.conf
    ```

    ```plaintext
    # SnapRAID configuration file
    # Parity Location
    1-parity /mnt/parity1/snapraid.parity

    # Content file location
    # Defines the files to use as content list
    # You can use multiple specification to store more copies
    # You must have least one copy for each parity file plus one. Some more don't hurt
    # They can be in the disks used for data, parity or boot,
    # but each file must be in a different disk
    # Format: "content FILE"
    content /var/snapraid.content
    content /mnt/disk1/snapraid.content

    # Defines the data disks to use
    # The name and mount point association is relevant for parity, do not change it
    # WARNING: Adding here your /home, /var or /tmp disks is NOT a good idea!
    # SnapRAID is better suited for files that rarely changes!
    # Format: "data DISK_NAME DISK_MOUNT_POINT"
    data d1 /mnt/disk1/

    # Excludes hidden files and directories (uncomment to enable).
    #nohidden

    # Defines files and directories to exclude
    # Remember that all the paths are relative at the mount points
    # Format: "exclude FILE"
    # Format: "exclude DIR/"
    # Format: "exclude /PATH/FILE"
    # Format: "exclude /PATH/DIR/"
    exclude *.unrecoverable
    exclude /tmp/
    exclude /lost+found/
    exclude downloads/
    exclude appdata/
    exclude *.!sync
    ```

9. **Check sync data**
    9.1 **Check**
    ```bash
    sudo snapraid -d d1 -a check
    ```

    9.2 **Scan files**
    ```bash
    snap run nextcloud.occ files:scan --all
    ```

    9.3 **Sync data**
    ```bash
    sudo snapraid sync
    ```

10. **Check data on Nextcloud**
