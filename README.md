# Watch Info
* No product name in the box or on [AliExpress](https://www.aliexpress.com/item/4000618243280.html)
* SoC is HS6620D by [HunterSun](https://huntersun.com.cn/), now called [OnMicro](https://www.onmicro.com.cn/)
  * Low power BLE chip, seems highly inspired by Nordic's nRF52 series
  * ARM Cortex-M3, so programmable/debuggable by SWD
  * 128KB RAM
  * 1MB SFLASH (What's that?)
  * Claims 5uA in deep sleep
* External 1MB PUYA SPI flash
  * [Datasheet](https://datasheet.lcsc.com/lcsc/2006180936_PUYA-P25Q64H-SSH-IT_C648137.pdf)

# Interesting links
* [AliExpress link](https://www.aliexpress.com/item/4000618243280.html)
* [HunterSun product list](http://huntersun.com.cn/product/overview.html)
  * [HS6620](https://www.onmicro.com.cn/portal/article/index/cid/40/id/632.html)
    * [Datasheet](https://www.scribd.com/document/487169502/HS6620D-data-sheet-V3-0)
* [hs662x.gdb.py](https://github.com/fengyichui/.dotfiles/blob/master/.gdb/hs662x.gdb.py)
  * [harvest](https://github.com/fengyichui/harvest/) looks like
* [hfirmware](https://github.com/kevinname/hfirmware) looks like a build project for HS6600 chip
  * [SDK drivers?](https://github.com/kevinname/hrom/tree/7cc46c4e05aba9a6b660d5ca6896f5c863e2c191/driver)
* [dt78 smart watch rev eng](https://github.com/fbiego/dt78)
  * Uses the same HS6620 chip!
  * [fccid](https://fccid.io/2ASHI-DT78/User-Manual/15-DT78-UserMan-US-r1-4523077)
* [How to SWD + gdb](https://wiki.seeedstudio.com/Software-SWD/)

# JLink
## Dumping RAM
```bash
% echo "savebin ram2.bin 0x0 0x20000\n" | JLinkExe -autoconnect 1 -Device CORTEX-M3 -If SWD -Speed 4000
SEGGER J-Link Commander V7.20 (Compiled Apr 28 2021 17:34:16)
DLL version V7.20, compiled Apr 28 2021 17:34:09

Connecting to J-Link via USB...O.K.
Firmware: J-Link ARM-OB STM32 compiled Aug 22 2012 19:52:04
Hardware version: V7.00
S/N: 20171218
License(s): RDI,FlashDL,FlashBP,JFlash,GDBFull
VTref=3.300V
Device "CORTEX-M3" selected.


Connecting to target via SWD
Found SW-DP with ID 0x2BA01477
Failed to power up DAP
Found SW-DP with ID 0x2BA01477
DPv0 detected
Scanning AP map to find all available APs
AP[1]: Stopped AP scan as end of AP map has been reached
AP[0]: AHB-AP (IDR: 0x24770011)
Iterating through AP map to find AHB-AP to use
AP[0]: Core found
AP[0]: AHB-AP ROM base: 0xE00FF000
CPUID register: 0x412FC231. Implementer code: 0x41 (ARM)
Found Cortex-M3 r2p1, Little endian.
FPUnit: 6 code (BP) slots and 2 literal slots
CoreSight components:
ROMTbl[0] @ E00FF000
ROMTbl[0][0]: E000E000, CID: B105E00D, PID: 000BB000 SCS
ROMTbl[0][1]: E0001000, CID: B105E00D, PID: 003BB002 DWT
ROMTbl[0][2]: E0002000, CID: B105E00D, PID: 002BB003 FPB
ROMTbl[0][3]: E0000000, CID: B105E00D, PID: 003BB001 ITM
ROMTbl[0][4]: E0040000, CID: B105900D, PID: 003BB923 TPIU-Lite
Cortex-M3 identified.
Opening binary file for writing... [ram2.bin]
Reading 131072 bytes from addr 0x00000000 into file...O.K.
```

## Chip information registers
From the gdb script above, register at `0x40000004` holds information about the chip
```
% echo "mem8 0x40000004 64\n" | JLinkExe -autoconnect 1 -Device CORTEX-M3 -If SWD -Speed 4000
...
40000004 = 20 66 A1 00 20 66 A1 00 20 66 A1 00 20 66 A1 00
40000014 = 00 00 00 00 00 00 00 00 70 C7 D7 FE 20 66 A1 00
40000024 = 00 00 20 00 FF FF FF FF E0 03 00 00 20 66 A1 00
40000034 = 20 66 A1 00 00 00 00 00 20 66 A1 00 00 00 00 00
40000044 = 20 66 A1 00 20 66 A1 00 20 66 A1 00 20 66 A1 00
40000054 = 20 66 A1 00 20 66 A1 00 20 66 A1 00 20 66 A1 00
40000064 = 20 66 A1 00
...
```
The chip ID seems to be `0x6620`, which matches the chip name - HS6620.

and device_version is in `0x08000034`, which seems to be `0x03`:
```
% echo "mem8 0x08000034 64\n" | JLinkExe -autoconnect 1 -Device CORTEX-M3 -If SWD -Speed 4000
...
08000034 = 00 03 20 66 E9 6B 03 08 A9 7E 03 08 91 8D 01 08
08000044 = FB 8E 01 08 CF 61 01 08 ED 6B 03 08 ED 6B 03 08
08000054 = ED 6B 03 08 ED 6B 03 08 ED 6B 03 08 01 0D 01 08
08000064 = 05 0D 01 08 ED 6B 03 08 B5 08 01 08 93 5C 01 08
08000074 = ED 6B 03 08 ED 6B 03 08 ED 6B 03 08 ED 6B 03 08
08000084 = ED 6B 03 08 ED 6B 03 08 ED 6B 03 08 69 6E 01 08
08000094 = 9F 6E 01 08
...
```

# GDB with Python support
```bash
% arm-none-eabi-gdb-py
```

Auto-connecting with `jlink` didn't work. I had to run `JLinkGDBServer.app` manually on my macOS.

## Getting device info
```bash
(gdb) device_info
Device: HS6620 A3
Flash: PUYA 1024KB (0x856014)
FlashExt: None 0KB (0x000000)
```
## Dumping the SPI flash
```bash
(gdb) flash_upload flash.bin
Device: HS6620 A3
Flash: PUYA 1024KB (0x856014)
FlashExt: None 0KB (0x000000)
  Upload... 100% (12.2s)
Finish: flash.bin
```