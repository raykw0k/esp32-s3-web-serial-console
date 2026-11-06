# ESP32-S3 Web Serial Console

Open a switch or access point's serial console in a browser. An ESP32-S3 stays beside the equipment and bridges UART to a web terminal on local Wi-Fi or, optionally, over Tailscale. Tested with an Aruba AP-315. The browser uses HTTP polling to receive output and HTTP POST to send keystrokes.

[Illustrated quick start](docs/web-serial-console-guide-en.png) · [中文说明](README.md)

## Flash and connect

The supplied image targets an ESP32-S3 N16R8 (16 MB flash, 8 MB octal PSRAM). For **first installation**, use esptool 4.x and replace PORT with your board's serial port. This merged image overwrites settings in the NVS partition.
```sh
python -m esptool --chip esp32s3 -p PORT write_flash 0x0 firmware/switch-console-s3-n16r8.bin
```

1. Join **raykwok-rmt-console**, password **12345678**. Open **http://192.168.4.1/** and enter the same setup password.
2. Select your site's 2.4 GHz Wi-Fi or phone hotspot. Leave **Use Tailscale** off for local access. For remote access, enable it and supply a Tailscale auth key on first setup.
3. Select **Test and save**. For local access, join the site network and open the displayed local IP over HTTP. In Tailscale mode, open the board's Tailscale address from a device on the same tailnet.
4. Log in with **12345678** and use **Change console password** to set your own. This changes the terminal password only. Use local HTTP on a trusted network. To reconfigure Wi-Fi, hold **BOOT** for four seconds after startup and release it.

## Serial wiring

For 3.3 V TTL, connect GPIO17 (TX) to the target's RX, GPIO18 (RX) to its TX, and GND to GND. Power the devices separately. Use a suitable level converter for RS-232. Start at 9600 baud, 8N1 for the Aruba AP-315. Adjust the settings in the web terminal for other devices.

## Build

Use ESP-IDF v5.4.3. With its environment active, run `idf.py build`, then `idf.py -p PORT flash`. MicroLink, its WireGuard components and xterm.js are bundled with their third-party notices. See the Chinese guide for upstream revisions and additional details.
