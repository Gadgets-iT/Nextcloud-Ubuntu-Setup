# Steps for Fixing Video Thumbnails in Nextcloud using Preview Generator with snap command

1. **Install Preview Generator from Admin Dashboard**
2. Go to the command line and download `ffmpeg` with:
    ```bash
    wget https://johnvansickle.com/ffmpeg/builds/ffmpeg-git-amd64-static.tar.xz
    ```
3. Extract the downloaded file:
    ```bash
    tar xvf ffmpeg-git-amd64-static.tar.xz
    ```
4. Create a `bin` folder:
    ```bash
    sudo mkdir /var/snap/nextcloud/bin
    ```
5. Move `ffmpeg` and `ffprobe` to the `bin` folder:
    ```bash
    sudo mv ffmpeg-git-20240629-amd64-static/ffmpeg ffmpeg-git-20240629-amd64-static/ffprobe /var/snap/nextcloud/bin/
    ```
6. Edit `config.php`:
    ```bash
    sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php
    ```

    Add the following lines:
    ```php
    'preview_ffmpeg_path' => '/var/snap/nextcloud/bin/ffmpeg',
    'enable_previews' => 'true',
    'enabledPreviewProviders' => array (
        0 => 'OC\\Preview\\TXT',
        1 => 'OC\\Preview\\MarkDown',
        2 => 'OC\\Preview\\OpenDocument',
        3 => 'OC\\Preview\\PDF',
        4 => 'OC\\Preview\\MSOffice2003',
        5 => 'OC\\Preview\\MSOfficeDoc',
        6 => 'OC\\Preview\\PDF',
        7 => 'OC\\Preview\\Image',
        8 => 'OC\\Preview\\Photoshop',
        9 => 'OC\\Preview\\TIFF',
        10 => 'OC\\Preview\\SVG',
        11 => 'OC\\Preview\\Font',
        12 => 'OC\\Preview\\MP3',
        13 => 'OC\\Preview\\Movie',
        14 => 'OC\\Preview\\MKV',
        15 => 'OC\\Preview\\MP4',
        16 => 'OC\\Preview\\AVI',
        17 => 'OC\\Preview\\MOV',
    ),
    ```

7. Generate all previews after installing the plugin:
    ```bash
    sudo snap run nextcloud.occ preview:generate-all -vvv
    sudo snap run nextcloud.occ preview:pre-generate
    ```

8. For older versions, you might need to set `crontab -e`, but for me, it worked immediately after uploading.

9. For those using the Memories app, add this script to `config.php`:
    ```php
    'memories.vod.ffmpeg' => '/var/snap/nextcloud/bin/ffmpeg',
    'memories.vod.ffprobe' => '/var/snap/nextcloud/bin/ffprobe',
    ```
