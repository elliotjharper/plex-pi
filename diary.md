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

