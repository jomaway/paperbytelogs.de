+++
title = "Fix ESP32 firmware size in PlatformIO"

[taxonomies]
tags = [ "embedded", "esp32", "platformIO", "partitions"]
+++

{{ note(header="Error", body="Error: The program size (1564865 bytes) is greater than maximum allowed (1310720 bytes) ")}}

Most ESP32 boards ship with a **1.3 MB (1310720 bytes) application partition**, unless a custom partition table is used. So if your firmware size is bigger than that, it won't compile.


**Solution:**

Change `platformio.ini` to use a partition layout with a 2 MB app slot.

```ini
board_build.partitions = partitions.csv
```


Create `partitions.csv` in your project root:

**Option A - "huge app"** 
```csv
# Name,   Type, SubType, Offset,  Size,    Flags
nvs,      data, nvs,     0x9000,  0x5000,
otadata,  data, ota,     0xe000,  0x2000,
app0,     app,  ota_0,   0x10000, 2M,
app1,     app,  ota_1,   0x210000, 2M,
spiffs,   data, spiffs,  0x410000, 0x3F0000,
``` 

**Option B - "no OTA"**

```csv
# Name,   Type, SubType, Offset,  Size,     Flags
nvs,      data, nvs,     0x9000,  0x5000,
phy_init, data, phy,     0xe000,  0x1000,
factory,  app,  factory, 0x10000, 0x2F0000,
spiffs,   data, spiffs,  0x300000, 0x100000,
```

See [[Understanding ESP32 partition tables]] for more information.

Or have a look at those [examples](https://docs.espressif.com/projects/arduino-esp32/en/latest/tutorials/partition_table.html#examples).
