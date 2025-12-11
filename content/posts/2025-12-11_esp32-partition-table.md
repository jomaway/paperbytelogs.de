+++
title = "Understanding ESP32 partition tables"

[taxonomies]
tags = ["esp32", "embedded", "partition"]

+++

I recently had the problem that my program size was greater than the default code partition. 
To fix this we can change the partition table of the ESP32. For a quick fix see [this note](/notes/esp32-firmware-size/).

# ESP32 partitions

The ESP32’s external flash (usually 4 MB, 8 MB, etc.) is divided into sections ("partitions"). 
Each partition has a **name**, **type**, **subtype**, **offset**, and **size** — just like a hard drive.
Let's have a quick look at an _example_ of a `partitions.csv` file.

```csv
# Name,   Type, SubType, Offset,   Size,     Flags
nvs,      data, nvs,     0x9000,   0x5000,
otadata,  data, ota,     0xe000,   0x2000,
app0,     app,  ota_0,   0x10000,  2M,
app1,     app,  ota_1,   0x210000, 2M,
spiffs,   data, spiffs,  0x410000, 0x3F0000,
``` 

But before we dive into the partition table we need to have a look at the bigger picture.

## ESP32 Flash Layout

ESP32 flash layout has **mandatory system regions** at fixed addresses, then a flexible area for your partition table and data.
Here you can see an **overview** of the layout.


| Address  | Description                           |
| -------- | ------------------------------------- |
| `0x0000` | Bootloader (fixed size, ~20-30 KB)    |
| `0x8000` | Partition Table (default location, 4KB) |
| `0x9000` | First data partition (NVS by default) |
| `0xXXXX` | Application firmware partitions       |
| `0xYYYY` | Filesystem partitions                 |

 So Bootloader and Partition Table use up the the first regions so the partition table starts at address `0x9000`.

## The partition table structure

The `partitions.csv` contains the following fields which represent the partition table.

| Field       | Size     | Description                                     |
| ----------- | -------- | ----------------------------------------------- |
| **Name**    | 16 bytes | Partition name (null-terminated string)         |
| **Type**    | 1 byte   | app=0x00, data=0x01                             |
| **Subtype** | 1 byte   | e.g. OTA_0=0x10, NVS=0x02, SPIFFS=0x82          |
| **Offset**  | 4 bytes  | Flash address (must be aligned to 0x1000 pages) |
| **Size**    | 4 bytes  | Partition size                                  |
| **Flags**   | 4 bytes  | Rarely used                                     |

### Types and Subtypes

Only two major types exist:

|Type|Meaning|
|---|---|
|`app`|Contains executable firmware|
|`data`|Contains NVS, SPIFFS, keys, system data|

#### App subtypes

For the `app` type, you can specify the following subtypes.

| Subtype   | Description                     |
| --------- | ------------------------------- |
| `factory` | Default firmware if no OTA used |
| `ota_0`   | OTA firmware slot 0             |
| `ota_1`   | OTA firmware slot 1             |
| `ota_x`   | Additional OTA slots (up to 15) |

#### Data Subtypes

For `data`, those subtypes are available

| Subtype    | Purpose                                             |
| ---------- | --------------------------------------------------- |
| `nvs`      | Key-value storage (Wi-Fi, preferences, calibration) |
| `otadata`  | OTA bootloader state (which slot is active)         |
| `spiffs`   | SPIFFS filesystem                                   |
| `fat`      | FAT filesystem                                      |
| `nvs_keys` | Secure NVS encryption keys                          |
| `phy`      | Radio calibration                                   |
| `coredump` | Crash dump storage                                  |
| `efuse_em` | Emulated eFuse storage (rare)                       |

{{ note(header="Tip", body="You should include at least `0x3000` bytes for the NVS partition.") }}

## Alignment rules

ESP32 flash partitions must align to **4 KB**.

{{ note(header="Note", body="
Partitions always start at addresses divisible by **0x1000** (4096 Bytes) \
Example: `0x9000`, `0x10000`, `0x210000` 
") }}

So to get back to our example from the start we have:

```csv
# Name,   Type, SubType, Offset,   Size,     Flags
nvs,      data, nvs,     0x9000,   0x5000,
otadata,  data, ota,     0xe000,   0x2000,
app0,     app,  ota_0,   0x10000,  2M,
app1,     app,  ota_1,   0x210000, 2M,
spiffs,   data, spiffs,  0x410000, 0x3F0000,
``` 

- A `20 KiB` Byte sized NVS partition starting at the address `0x9000`.
- A `8 KiB` Byte sized otadata partition starting at `0x9000` + `0x5000` = `0xe000`.
- Two `2 MB` Byte sized app partitions for our code.
- A `4 MiB` Byte sized SPIFFS partition filling the rest of the available memory.


## Ressources

- [Espressif Docs - Partition Tables](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/partition-tables.html)
- [Espressif Docs - Partition Tables (Arduino)](https://docs.espressif.com/projects/arduino-esp32/en/latest/tutorials/partition_table.html)
- [Medium - How to use custom partition tables](https://medium.com/the-esp-journal/how-to-use-custom-partition-tables-on-esp32-69c0f3fa89c8)

