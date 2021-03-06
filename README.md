# Udev Reminder
Reminder for tricks around udev :sunglasses:

If you need more information check here:

- [doc.ubuntu-fr](https://doc.ubuntu-fr.org/udev)

## Reload UDEV Rules and restart device triggering

Use this command to reload the udev rules and restart a device detection.

```bash
sudo udevadm control --reload && sudo udevadm trigger
```

## To list usb information usable in udev rules

This command list you all the attribute associated to you device. Those attributes are formatted to be used directly in udev rules

```bash
udevadm info -a /dev/ttyS0

#  looking at device '/devices/platform/serial8250/tty/ttyS0':
#     KERNEL=="ttyS0"
#     SUBSYSTEM=="tty"
#     DRIVER==""
#     ATTR{close_delay}=="50"
#     ATTR{closing_wait}=="3000"
#     ATTR{console}=="N"
#     ATTR{custom_divisor}=="0"
#     ATTR{flags}=="0x10000040"
#     ATTR{io_type}=="0"
#     ATTR{iomem_base}=="0x0"
#     ATTR{iomem_reg_shift}=="0"
#     ATTR{irq}=="4"
#     ATTR{line}=="0"
#     ATTR{port}=="0x3F8"
#     ATTR{power/async}=="disabled"
#     ATTR{power/control}=="auto"
#
#  [...]
#
```

You don't know your usb device path ? Use the monitor to see udev event on your machine.


```bash
# Start monitor
udevadm monitor -k

# Plug your device and watch
# KERNEL[5657.769265] add      /devices/pci0000:00/0000:00:06.0/usb1/1-2 (usb)
# KERNEL[5657.773323] add      /devices/pci0000:00/0000:00:06.0/usb1/1-2/1-2:1.0 (usb)
# KERNEL[5657.784851] add      /devices/pci0000:00/0000:00:06.0/usb1/1-2/1-2:1.0/0003:413C:301A.0002 (hid)
# KERNEL[5657.784881] add      /devices/pci0000:00/0000:00:06.0/usb1/1-2/1-2:1.0/0003:413C:301A.0002/input/input8 (input)
# KERNEL[5657.785082] add      /devices/pci0000:00/0000:00:06.0/usb1/1-2/1-2:1.0/0003:413C:301A.0002/input/input8/mouse2 (input)
```

## To list usb information usable in udev rules

Udev rules are stored in **/etc/udev/rules.d/**, you can create you onw rules 99-my-rules.rules.

```bash
sudo touch /etc/udev/rules.d/99-my-rules.rules
sudo chmod 644 /etc/udev/rules.d/99-my-rules.rules
```

### Quick Cheat Sheet

There is 2 types of operation on a udev rule

- **==** condition
- **=** affection

Affectations are applied if all conditions are true.

- **\\** To brake a rule in multiple lines

### Examples

- If the device has the idVendor "0403" and the idProduct "e0d0" (arduino uno)
- And if it is a tty interface (SUBSYSTEM=="tty")
- Then append a "/dev/ttyFOOO"

```bash
ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", \
SUBSYSTEM=="tty", \
SYMLINK+="ttyFOOO"
```

When you have multiple board with the same usb ids, use the serial

```bash
SUBSYSTEM=="tty", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", ATTRS{serial}=="75833353934351904112", \
SYMLINK+="ttyBOARD_1"

SUBSYSTEM=="tty", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", ATTRS{serial}=="75833350254248468412", \
SYMLINK+="ttyBOARD_2"
```

When you must use 'sudo' to access to an USB device. !Be carefull! there 4 digits to MODE

```bash
ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", \
MODE="0666"
```

## Python function to get serial port from usb ids

In the case you cannot set udev rules

```python
# pip install pyudev

import pyudev

def ttyPortfromUsbInfo(vendor_id, product_id, serial=None, base_dev_tty="/dev/ttyACM"):
    # Explore usb device with tty subsystem
    udev_context = pyudev.Context()
    for device in udev_context.list_devices(ID_BUS='usb', SUBSYSTEM='tty'):
        properties = dict(device.properties)
        
        # Need to find the one with the DEVNAME corresponding to the /dev serial port
        if 'DEVNAME' not in properties or not properties['DEVNAME'].startswith(base_dev_tty):
            continue

        # Check vendor/product/serial
        if vendor_id == properties["ID_VENDOR_ID"] and product_id == properties["ID_MODEL_ID"]:
            if serial:
                if serial == properties["ID_SERIAL_SHORT"]:
                    return properties["DEVNAME"]
            else:
                return properties["DEVNAME"]

    return None

# Usage
tty = ttyPortfromUsbInfo("abcd", "1234")
```
