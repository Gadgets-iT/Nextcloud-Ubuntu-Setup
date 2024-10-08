# Part 1 Ubuntu Server Installation with Nextcloud and SnapRAID

1. **Check IP address from previous setup**
    ```sh
    ssh yourname@ipaddress
    ```

2. **Install mergerfs**
    ```sh
    sudo apt install mergerfs fuse
    ```

3. **Create new partition and Linux file system (except the OS disk)**

    3.1. Review hard disk with command:
    ```sh
    lsblk
    ```
    3.2. Create partition:
    ```sh
    sudo gdisk /dev/sd? -> o -> n -> p -> w
    sudo mkfs.ext4 /dev/sd?1
    ```

4. **Mount and merge all partitions**

    4.1. Create folders:
    ```sh
    sudo mkdir /mnt/disk{1,2,3}
    sudo mkdir /mnt/parity{1,2}
    sudo mkdir /mnt/storage
    ```
    4.2. See disk ID:
    ```sh
    ls -l /dev/disk/by-id
    ```
    4.3. Edit fstab file:
    ```sh
    sudo nano /etc/fstab
    ```

    ```plaintext
    //-------- within the fstab file----------//
    /dev/disk/by-id/ata-ST1000DM010-2EP102_Z9AQQ04H-part1 /mnt/parity1 ext4 defaults 0 0
    /dev/disk/by-id/ata-WDC_WD10JPVX-22JC3T0_WD-WX71E23LLS35-part1 /mnt/disk1 ext4 defaults 0 0

    /mnt/disk* /mnt/storage fuse.mergerfs direct_io,defaults,allow_other,minfreespace=10G,fsname=mergerfs 0 0
    //---------------------------------//
    ```

    4.4. Mount all the disks:
    ```sh
    sudo mount -a
    df -h
    ```

5. **Nextcloud**

    5.1. Install Nextcloud:
    ```sh
    sudo snap install nextcloud
    ```
    5.2. Check Nextcloud and start the system:
    ```plaintext
    ! Important to run and login to Nextcloud before going to next step because Nextcloud will generate a new config.php file.
    ```

6. **Change Directory**
    ```sh
    sudo snap connect nextcloud:removable-media
    sudo chown -R root:root /mnt/storage
    sudo snap stop nextcloud
    sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php
    ```

    ```php
    //---
    'trusted_domains' => array(
        0 => '*',
    ),
    'datadirectory' => '/mnt/storage/data',
    //----
    ```

    ```sh
    sudo mv /var/snap/nextcloud/common/nextcloud/data /mnt/storage
    sudo snap start nextcloud
    ```

7. **Install snapraid**
    ```sh
    sudo apt install snapraid
    sudo nano /etc/snapraid.conf
    ```

    ```plaintext
    # SnapRAID configuration file
    # Parity Location
    1-parity /mnt/parity1/snapraid.parity
    2-parity /mnt/parity2/snapraid.parity

    # Content file location
    content /var/snapraid.content
    content /mnt/disk1/snapraid.content
    content /mnt/disk2/snapraid.content

    # Defines the data disks to use
    data d1 /mnt/disk1/
    data d2 /mnt/disk2/
    data d3 /mnt/disk3/

    # Excludes hidden files and directories (uncomment to enable).
    #nohidden

    # Defines files and directories to exclude
    exclude *.unrecoverable
    exclude /tmp/
    exclude /lost+found/
    exclude downloads/
    exclude appdata/
    exclude *.!sync
    ```

    ```sh
    sudo snapraid sync
    ```

8. **Auto snapraid sync**
    ```sh
    git clone https://github.com/Chronial/snapraid-runner.git
    cd snapraid-runner
    mv snapraid-runner.conf.example snapraid-runner.conf
    nano snapraid-runner.conf
    ```

    8.1. Edit conf file:
    ```plaintext
    //--[snapraid]
    executable = /usr/bin/snapraid

    config = /etc/snapraid.conf
    //--[logging]
    file = /home/<username>/snapraid-runner/snapraid.log
    //---edit email as you wish --//
    ```

    8.2. Run test:
    ```sh
    sudo python3 /home/<username>/snapraid-runner/snapraid-runner.py -c /home/<username>/snapraid-runner/snapraid-runner.conf
    ```

    8.3. After success, setup with crontab:
    ```sh
    sudo crontab -e
    0 3 * * * python3 /home/<username>/snapraid-runner/snapraid-runner.py -c /home/<username>/snapraid-runner/snapraid-runner.conf
    ```

9.  **Access from Internet**
    ```sh
    sudo nextcloud.enable-https self-signed
    ```

    9.1. Port Forwarding

    9.2. DuckDDNS

10. **For someone who has their own domain name, we can have HTTPS**

    10.1. Install certbot with snap:
    ```sh
    sudo snap install certbot
    ```
	10.2 point sub domain to xxx.duckdns.org using CNAME

	10.3 using below command to auto generate let'sencrypt
		
    reference: [certbot_dns_script]https://github.com/EchoBased/certbot_dns_script
    
    ```
    read  -p "Enter domain: " domain

    echo $domain

    sudo certbot --manual --preferred-challenges dns certonly -d $domain

    echo "Moving certs to live dir"

    sudo cp /etc/letsencrypt/live/<your-domain>/chain.pem /var/snap/nextcloud/current/

    sudo cp /etc/letsencrypt/live/<your-domain>/privkey.pem /var/snap/nextcloud/current/

    sudo cp /etc/letsencrypt/live/<your-domain>/cert.pem /var/snap/nextcloud/current/

    sudo nextcloud.enable-https custom -s /var/snap/nextcloud/current/cert.pem /var/snap/nextcloud/current/privkey.pem /var/snap/nextcloud/current/chain.pem 
    ```

	10.4 during the script it will ask for the txt. you have to set up dns with text type something like this

		type - TXT
		name - _acme-challenge.<your-previous-subdomain> #forexample _acme-challenge.nextcloud
		content - <Copy the text that generate from the above code>

	10.5 Done! you can access with your sub domain without the warning and can share file to other people 

	10.6 However, this certificate is not renew automatically so we need to do a manual step 
		as they mention

		NEXT STEPS: This certificate will not be renewed automatically. Autorenewal of --manual certificates requires the use of an authentication hook script (--manual-auth-hook) but one was not provided. To renew this certificate, repeat this same certbot command before the certificate's expiry date.


