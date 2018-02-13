Deprecated

-----------

# BLE Partial Flashing Service Specification
The partial flashing service allows a BLE client to connect to a micro:bit and read and write the information required to partially update the firmware (e.g. the MakeCode section of the flash).
The device's flash layout can be read using the Memory Map Characteristic. This characteristic allows the client to read the memory map regions, and request more details about specific regions.

This allows the client to understand the flash regions and compare them with a new firmware HEX file. If the SD and DAL hashes are identical between the micro:bits current firmware and the new firmware partial flashing is possible and the phone begins to send flash data. Each packet of data contains 16 bytes of information plus the offset that it needs to be written to. If the hashes didn't match a full flash occurs.

## Memory Map Characteristic
### UUID: e97d3b10-251d-470a-a062-fa1922dfa9a8
When the micro:bit powers on the Memory Map characteristic is initialised to return the names of the different flash regions. Each region has a 3 character name so that 6 regions (18 bytes) can be represented in one 20 byte BLE read response.

### Initial State
```
READ CHAR:
"SD DALPXT         "
```

Regions can then be selected using a `WRITE` request with an integer payload used to select the region. Once a region has been selected two `READ` requests can be issued to obtain the regions's start/end address and it's hash.
For example:
`WRITE CHAR: 0x00` returns information about the SD region
`WRITE CHAR: 0x01` returns information about the DAL region     
`WRITE CHAR: 0x01` returns information about the PXT region

Writing a payload of `0xFF` to the characteristic returns it to it's initial state.

### Reading Region Information
The Region Information is returned in two packets, the first containing the start and end addresses, the second containing the region's hash.
#### Packet 0
| Byte #  |  Value |
|---|---|
| 0 |  startAddress[0xFF] |
| 1 |  startAddress[0xFF00 >> 8] |
| 2 |  startAddress[0xFF0000 >> 16] |
| 3 |  startAddress[0xFF000000 >> 24] |
| 4..7 |  |
| 8 |  endAddress[0xFF] |
| 9 |  endAddress[0xFF00 >> 8] |
| 10 | endAddress[0xFF0000 >> 16] |
| 11 | endAddress[0xFF000000 >> 24] |
| 12..17 |   |
| 18 | Region Of Interest ID  |
| 19 |  Packet #: 0x00 |

#### Packet 1
| Byte #  |  Value |
|---|---|
| 0..7 |  Hash[#]
|8..17 | |
| 18 | Region Of Interest ID  |
| 19 |  Packet #: 0x01 |

## Flash Characteristic
### UUID: e97faa6d-251d-470a-a062-fa1922dfa9a8
The flash characteristic allows the client to write the firmware to the micro:bit's flash memory. Each packet contains 16 bytes of data and the offset at which to write it. This offset is used with the currently selected region's start address to determine the datas location in flash.

The characteristic supports `WRITE_WITHOUT_RESPONSE` and the minimum connection interval is set to `7.5ms` to reduce transfer overheads.

### Writing Data
`WRITE` requests are structured as follows:

| Byte # | Value |
|---|---|
|0..15| Data |
|16 | Offset[0xFF00 >> 8] |
| 17 | Offset[0xFF] |

## Flash Control Characteristic
### UUID: e97fab6d-251d-470a-a062-fa1922dfa9a8
The flash control characteristic is used to notify the app that a write has completed and the next packet can be transmitted.

A notification of `0xFF` indicates that a packet has been successfully written to flash.

A notification of `0xAA` indicates that a packet has _not_ been written to flash.
The client must then resend the packet whilst continuing to increment the packet number.

# Client Implementation
- An example implementation for Android can be found here [here](https://github.com/microbit-sam/microbit-android/blob/partial-flash/app/src/main/java/com/samsung/microbit/service/PartialFlashService.java).
- Example APK located [here](https://github.com/microbit-sam/microbit-android/blob/partial-flash/app/build/outputs/apk/) (Use debug).
- MakeCode using custom DAL [here](https://makecode.microbit.org/app/99df2005db364de096fa8f5769b171d3586ea9a0-569874149a)

(Diagram needs updating)
![Partial Flashing Flowchart](pfs.png "Partial Flashing Flow")

---
- [ ] Redraw Diagram
- [ ] Improve docs
- [ ] Remove source code from transfer
