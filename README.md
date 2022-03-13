# Udev Reminder
Reminder for tricks around udev :sunglasses:

## Reload UDEV Rules and restart device triggering

Use this command to reload the udev rules and restart a device detection.

```bash
sudo udevadm control --reload-rules && sudo service udev restart && sudo udevadm trigger
```

## To list usb information usable in udev rules

```bash
udevadm info -a /dev/ttyACM0
```

## Python function to get serial port from usb ids

```python
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
```
