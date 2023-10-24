- grabbed official ubuntu plex media server image for ras pi
    - https://ubuntu.com/appliance/plex/raspberry-pi

- flashed it with etcher 1.2.0 portable
    - https://etcher.balena.io/#download-etcher

- (Optional) wrote wifi setup file to the boot partition to preconfigure the pi to connect to wifi
    - path: /wpa_supplicant.conf
    - contents:
        country=GB
        ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
        update_config=1

        network={
            ssid="Your_SSID" # Replace with your Wi-Fi network name
            psk="Your_Password" # Replace with your Wi-Fi password
        }

- wrote ssh file to the boot partition to preconfigure the pi to enable ssh access
    - path: /ssh
    - contents: <empty>

- RESULT: FAIL
    - raspbian images no longer include the default user and password

- SOLUTION
    - instead of using etcher and then manually setting up ssh and the user and wifi
        just use the pi imager and you can select the custom image but use the cog to
        setup ssh, wifi and user accounts

- RESULT: FAIL
    - the plex image is a ubuntu core image that is setup to auth specifically with private keys only

- SOLUTION
    - setup private keys on mac using 
        - `ssh-keygen -t rsa -b 2048`
            - public key file: `~/.ssh/id_rsa.pub`
            - private key file: `~/.ssh/id_rsa`
    - imported the public key generated into a ubuntu account
    - hooked up the pi to a monitor and logged into that ubuntu email and it pulled down the public keys
    - then connected to the machine using the private key with the command 
        - `ssh -i ~/.ssh/id_rsa <username seen inside ubuntu account>@<ip of pi>`

- set pi up to mount my network shared media on startup so that it shows up to be added to plex as a library
    - made a mount point
        - `mkdir /mnt/drive-1`
    - wrote a file to instruct ubuntu core to mount our share
        - couldn't get a version of nano through snap store that was compiled for our
            architecture (arm64) so resorted to using winscp to write the file up
        - wrote file locally (in this repo)
        - copied it to home folder
        - then used ssh to copy it to the correct folder
            - `sudo cp /home/elltg/drive-1.mount /etc/systemd/system/nas-drive-1.mount`
        - then restart necessary deamons to make it take effect
            sudo systemctl daemon-reload
            sudo systemctl enable nas-drive-1.mount
            sudo systemctl start nas-drive-1.mount
        - ERROR: nas-drive-1.mount: Where= setting doesn't match unit name.
            - had done the where wrong. 
            - NOTE: MUST match the mount folder to the mount unit file name
                - this is achieved using `systemd-escape`
                - explanatory stackoverflow: 
                    https://unix.stackexchange.com/questions/283442/systemd-mount-fails-where-setting-doesnt-match-unit-name
            - RESULT: correct path for the mount unit is
                - `sudo cp /home/elltg/drive-1.mount /etc/systemd/system/mnt-nas\x2ddrive\x2d1.mount`
            - remove old mount
                - `sudo systemctl disable nas-drive-1.mount`
            - ADD CORRECT MOUNT!
                - `sudo systemctl enable mnt-nasx2ddrivex2d1.mount`
            - start correct mount
                - `sudo systemctl start mnt-nasx2ddrivex2d1.mount`
        - ERROR: still didn't line up.
        - SOLUTION: just simplified the mount point to not have the hyphens in it
        
- REVISED STEPS to setup network share:
    - used raspberry pi imager
    - picked pi os 64 lite (latest one with no desktop)
    - in the imager customised to setup a user and password and enabled SSH access
    - make a folder to mount at
        - `sudo mkdir /mnt/nasdrive1`
    - copy .mount file through ssh to home folder 
    - copy the file into relevant etc/systemd location
        - `sudo cp /home/pi/mnt-nasdrive1.mount /etc/systemd/system/mnt-nasdrive1.mount`
    - register and try to start the mount
        - `sudo systemctl daemon-reload`
        - `sudo systemctl enable mnt-nasdrive1.mount`
        - `sudo systemctl start mnt-nasdrive1.mount`

- install plexmediaserver via snap store
    - update everything and ensure that snap store is installed
        - `sudo apt update`
        - `sudo apt install snapd`
    - reboot
        - `sudo reboot`
    - install the ubuntu core snap
        - `sudo snap install core`
    - install the plexmediaserver snap
        - `sudo snap install plexmediaserver`
