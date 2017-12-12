# BLE Partial Flashing Service Specification
The partial flashing service allows a BLE client to connect to a micro:bit and read and write the information required to partially update the firmware (e.g. the MakeCode section of the flash).
The device's flash layout can be read using the Memory Map Characteristic. This characteristic allows the client to read the memory map regions, and request more details about specific regions.

This allows the client to understand the flash regions and compare them with a new firmware HEX file. If the SD and DAL hashes are identical between the micro:bits current firmware and the new firmware partial flashing is possible and the phone begins to send flash data. Each packet of data contains 16 bytes of information plus the offset that it needs to be written to. If the hashes didn't match a full flash occurs.

## Memory Map Characteristic
### UUID: 0xe9,0x7d,0x3b,0x10,0x25,0x1d,0x47,0x0a,0xa0,0x62,0xfa,0x19,0x22,0xdf,0xa9,0xa8
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
### UUID: 0xe9,0x7f,0xaa,0x6d,0x25,0x1d,0x47,0x0a,0xa0,0x62,0xfa,0x19,0x22,0xdf,0xa9,0xa8
The flash charateristic allows the client to write the firmware to the micro:bit's flash memory. Each packet contains 16 bytes of data and the offset at which to write it. This offset is used with the currently selected region's start address to determine the datas location in flash.

The characteristic supports `WRITE_WITHOUT_RESPONSE` and the minimum connection interval is set to `7.5ms` to reduce transfer overheads.

### Writing Data
`WRITE` requests are structured as follows:

| Byte # | Value |
|---|---|
|0..15| Data |
|16 | Offset[0xFF00 >> 8] |
| 17 | Offset[0xFF] |

# Client Implementation
An example implementation for Android can be found here [here](https://github.com/microbit-sam/microbit-android/blob/partial-flash/app/src/main/java/com/samsung/microbit/service/PartialFlashService.java).
![Partial Flashing Flowchart](pfs.png "Partial Flashing Flow")