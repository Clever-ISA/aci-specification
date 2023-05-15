# 1xN ACI-ACI Device Bridges, No Internal Clock

This specification defines the required behaviour for devices belonging to class 4, subclass 0 (1xN ACI-ACI Device Bridges, No Internal Clock). 
Such devices expose an interface for multiple downstream ACI devices over a single ACI connector. A single bridge device may support a minimum of 1 connected devices, and a maximum of 2016 connected devices. 

## §1 Device Class Information

An ACI Device that implements this specification shall use class 4 (bridges), subclass 0 (aci-bridge-1xn). 

## §2 Exposed Registers

The following registers are exposed by the device to the host:


| Register Address | Internal Name | Register Name | Effect on Read | Effect on Write |
|-|-|-|-|-|
| 0x010 | maxdevs | Max Connected Count | Returns the number of devices the bridge supports simultaneously | Discarded by device |
| 0x011 | interrupt-source | Interrupt Source Devid | Returns the device id of the device that last set Device to Host Interrupt | Signals Host to Device Interrupt on the nth connected device, where n is the value written |
| 0x020-0x7FF | device-n-configuration | Device Configuration | Current Value (0xFFFFFFFF at startup) for register | Modifies the configuration state of the nth device (when accessing register 0x20+n), where `n<maxdevs`. See below for bitfield meaning|

All other registers are reserved and read as `0`, with writes being discarded. A host may not rely on the value of a reserved register, nor on the result of writing to a reserved register.

The Configuration state of a given device is as folows

| Bits | Internal Name | Name | Meaning |
|-|-|-|-|
| 0-15 | config-devid  | Device ID | The ID assigned to the device. Modifying this value causes the appropriate device's device id assignment to be changed. |
| 16   | config-host-interrupt | Device to Host Interrupt Available | Whether or not the Device to Host Interrupt signal should be forwarded (1 to forward the signal, 0 to block the signal)
| 17   | config-host-interrupt-raised | Set by the bridge whenever the device signals a device to host interrupt, regardless of whether forwarding is enabled |
| 18   | config-device-interrupt-raise | When set by the client, the bridge asserts host-to-device interrupt on the line for 2 cycles. |
| 63   | device-unconnected | Device Unconnected | Set by the bridge while the device is not connected. Must not be modified by the host. | 

All other bits are reserved and must not be set by the host, and the host must ignore the value of those bits. THe device shall not modify those bits and shall ignore the state of those bits. 

The Device Configuration state is idempotent with respect to a particular field - no effect shall occur if the bitfield is written with the previous value. 

Any effect that results from modifying a device configuration register is queued, and occurs after the DMA Transfer completes. If multiple writes as part of a single transfer modify the same field several times, the result is undefined. 
