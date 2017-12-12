BLE Partial Flashing Service Specification

The partial flashing service allows a BLE client to connect to a micro:bit and read and write the information required to partially update the firmware. (PXT blob).

The device's flash layout can be read using the MemoryMapCharacteristic (UUID ###). This characteristic allows the client to read the memory map regions, and request more details about specific regions.

Write:  0xFF
Read:  "SD DALPXT      "

WRITE:  0x??
READ:   Start Address, End Address
READ    Hash

This process allows the client to understand the flash regions and compare them with a new firmware HEX file.
If the SD and DAL hashes are identical between the micro:bits current firmware and the new firmware partial flashing is possible. If not a full flash commences.

If a partial flash is possible the phone begins to send flash data. Each packet of data contains 16 bytes of information plus the offset that it needs to be written to. 