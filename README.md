# How to update firmware on Redbear Duo devices

Why write this? Because RB's instructions are complex and do things out of order.

Last performed on: July 28 2018
  - flashed devices Duo-[1-8] to 0.3.1

Resources:
  - https://github.com/redbear/Duo/blob/master/docs/firmware_deployment_guide.md
  - https://github.com/redbear/Duo/blob/master/docs/devices_provisioning_guide.md
  - https://github.com/redbear/device-provisioning-helper

Directories for operating within: (clone these)
  - [Code/c++/Duo](https://github.com/redbear/Duo.git)
  - [Code/nodejs/device-provisioning-helper](https://github.com/redbear/device-provisioning-helper.git)

## Updating

From `Code/c++/Duo/firmware`:

```
# Wipe temp backup files from previous runs
rm dct/device_private_key_backup.der
rm dct/device_private_key.der
rm dct/dct_backup.bin

# Export keys for safekeeping/fallback
dfu-util -d 2b04:d058 -a 1 -s 34 -U dct/device_private_key_backup.der
dfu-util -d 2b04:d058 -a 1 -s 34 -U dct/device_private_key.der

# Load new Device Configuration Table, re-set keys
dfu-util -d 2b04:d058 -a 0 -s 0x8008000 -D dct/fac-dct-r1.bin
dfu-util -d 2b04:d058 -a 1 -s 2082 -D dct/server_public_key.der
dfu-util -d 2b04:d058 -a 1 -s 34 -D dct/device_private_key.der

# Load new firmware
dfu-util -d 2b04:d058 -a 0 -s 0x8020000 -D system/v0.3.2/duo-system-part1-v0.3.2.bin
dfu-util -d 2b04:d058 -a 0 -s 0x8040000 -D system/v0.3.2/duo-system-part2-v0.3.2.bin

# Load new Factory Reset Application & new Wifi system
dfu-util -d 2b04:d058 -a 2 -s 0x140000 -D system/v0.3.2/duo-fac-web-server-v0.3.2.bin
dfu-util -d 2b04:d058 -a 2 -s 0x180000 -D wifi/duo-wifi-r1.bin
```

## Provisioning

From `Code/c++/Duo/firmware`:

```
# Enter DFU (blinking yellow)
dfu-util -d 2b04:d058 -a 1 -s 34:leave -D dct/reset_device_private_key.bin

# Enter LISTENING (blinking blue)
picocom /dev/tty.usbmodem1411
i   # prints DEVICE_ID
v   # prints new firmware id
w   # enter network details
```

From `Code/nodejs/device-provisioning-helper`:

```
node main.js DEVICE_ID
```