# arduino-gameboy-printer-emulator

**[Featured On Hack A Day Article](https://hackaday.com/2017/12/01/arduino-saves-gameboy-camera/)**

Code to emulate a gameboy printer via the gameboy link cable

![](./sample_image/gameboy_printer_emulator.png)

Blog Post: http://briankhuu.com/projects/gameboy_camera_arduino/gameboy_camera_arduino.html

Goal is to provide an easy way for people to quickly setup and download the images from their gameboy to their computer before the battery of these gameboy cameras dies of old age.

I hope there will be a project to collate these gameboy images somewhere.

## Quick Start

### Construct the Arduino Gameboy Printer Emulator

Use an arduino nano and wire the gameboy link cable as shown below.
If you can fit the gameboy camera to gameboy advance etc... you may need a
differen pinout reference. But the wiring should be similar.

* Pinout Ref: http://www.hardwarebook.info/Game_Boy_Link


### Pinout Diagram

Thanks to West McGowan (twitter: @imwestm) who was able to replicate this project on his Arduino Nano plus Gameboy Color and helpfully submitted a handy picture of how to wire this project up. You can find his tutorial in [here](https://westm.co.uk/arduino-game-boy-printer-emulator/)

![](GBP_Emu_Micro_pinout_West_McGowan.webp.png)

### General Pinout

```
Gameboy Original/Color Link Cable Pinout
 ___________
|  6  4  2  |
 \_5__3__1_/   (at cable)
```

| Arduino Pin | Gameboy Link Pin                 |
|-------------|----------------------------------|
|  unused     | Pin 1 : 5.0V                     |
|  D4         | Pin 2 : Serial OUTPUT            |
|  D3         | Pin 3 : Serial INPUT             |
|  unused     | Pin 4 : Serial Data              |
|  D2         | Pin 5 : Serial Clock (Interrupt) |
|  GND        | Pin 6 : GND (Attach to GND Pin)  |

### Programming the emulator

* File: `./gbp_emulator/gpb_emulator.ino` + `gameboy_printer_protocol.h`
* Baud 115200 baud

Next download `./gbp_emulator/gpb_emulator.ino` to your arduino nano.
After that, open the serial console and set the baud rate to 115200 baud.

### Download the image

Press the download button in your gameboy. The emulator will automatically start to download and dump the data as a string of hex in the console display.

After the download has complete. Copy the content of the console to the javascript decoder in `./jsdecoder/gameboy_printer_js_decoder.html`. Press click to render button.

One you done that, your image will show up below. You can then right click on the image to save it to your computer. Or you can click upload to imgur to upload it to the web in public, so you can share it. (Feel free to share with me at mofosyne@gmail.com).

A copy of the decoder is accessible via my website as well:
    - http://briankhuu.com/projects/gameboy_camera_arduino/gameboy_camera_arduino.html

You are all done!

## Project Makeup

* Arduino sketch emulating a gameboy printer to a computer via a serial link.
    - `./gbp_emulator/gpb_emulator.ino` : Main source file
    - `./gbp_emulator/gameboy_printer_protocol.h` : Reusable header containing information about the gameboy protocol
    - The serial output is outputting a gameboy tile per line filled with hex. (Based on http://www.huderlem.com/demos/gameboy2bpp.html)
    - A tile in the serial output is 16 hex char per line: e.g. `55 00 FB 00 5D 00 FF 00 55 00 FF 00 55 00 FF 00`

* Javascript gameboy printer hex stream rendering to image in browser.
    - [js decoder page](./jsdecoder/gameboy_printer_js_decoder.html)
    - `./jsdecoder/gameboy_printer_js_decoder.html` :
    - This is a convenient way to convert the hex payload into canvas image that can be downloaded.

* Research folder.
    - Contains some files that I found online to research this project

* Sample Images
    - Some sample images along with the sample logs. There has been some changes to the serial output, but the hex format remains the same.

## Credits / Other Resources

* GameBoy PROGRAMMING MANUAL Version 1.0 DMG-06-4216-001-A Released 11/09/1999
    - Is the original programming manual from nintendo. Has section on gameboy printer. Copy included in research folder.

* [http://www.huderlem.com/demos/gameboy2bpp.html](http://www.huderlem.com/demos/gameboy2bpp.html) Part of the js decoder code is based on the gameboy tile decoder tutorial
* [https://github.com/gism/GBcamera-ImageSaver](https://github.com/gism/GBcamera-ImageSaver) - Eventally found that someone else has already tackled the same project.
    - However was not able to run this sketch, and his python did not run. So there may have been some code rot.
    - Nevertheless I was able to get some ideas on using an ISR to capture the bits fast enough.
    - I liked how he outputs in .bmp format
* [http://gbdev.gg8.se/wiki/articles/Gameboy_Printer](http://gbdev.gg8.se/wiki/articles/Gameboy_Printer) - Main gb documentation on gb protocol, great for the inital investigation.
* [http://furrtek.free.fr/?a=gbprinter&i=2](http://furrtek.free.fr/?a=gbprinter&i=2) - Previous guy who was able to print to a gameboy printer
* [http://playground.arduino.cc/Main/Printf](http://playground.arduino.cc/Main/Printf) - printf in arduino
* [https://www.mikrocontroller.net/attachment/34801/gb-printer.txt](https://www.mikrocontroller.net/attachment/34801/gb-printer.txt)
    - Backup if above link is dead [here](./ext_doc/gb-printer.txt)
    - Most detailed writeup on the protocol I found online.
* [http://gbdev.gg8.se/wiki/articles/Serial_Data_Transfer_(Link_Cable)](http://gbdev.gg8.se/wiki/articles/Serial_Data_Transfer_(Link_Cable))
* [https://github.com/avivace/awesome-gbdev](https://github.com/avivace/awesome-gbdev) Collections of gameboy development resources
* [https://shonumi.github.io/articles/art2.html](https://shonumi.github.io/articles/art2.html) An in-depth technical document about the printer hardware, the communication protocol and the usual routine that games used for implementing the print feature.

## Technical Information

### Protocol

```
  | BYTE POS :    |     0     |     1     |     2     |      3      |     4     |     5     |  6 + X    | 6 + X + 1 | 6 + X + 2 | 6 + X + 3 | 6 + X + 4 |
  |---------------|-----------|-----------|-----------|-------------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|
  | SIZE          |        2 Bytes        |  1 Byte   |   1 Byte    |  1 Bytes  |  1 Bytes  | Variable  |        2 Bytes        |  1 Bytes  |  1 Bytes  |
  | DESCRIPTION   |       SYNC_WORD       | COMMAND   | COMPRESSION |     DATA_LENGTH(X)    | Payload   |       CHECKSUM        |  DEVICEID |  STATUS   |
  | GB TO PRINTER |    0x88   |    0x33   | See Below | See Below   | Low Byte  | High Byte | See Below |       See Below       |    0x00   |    0x00   |
  | TO PRINTER    |    0x00   |    0x00   |    0x00   |   0x00      |    0x00   |    0x00   |    0x00   |    0x00   |    0x00   |    0x81   | See Below |
```

  * Header is the Command, Compression and Data Length
  * Command field may be either Initialize (0x01), Data (0x04), Print (0x02), or Inquiry (0x0F).
  * Compression field is a compression indicator. No compression (0x00), Yes Compression (0x01)
  * Payload byte count size depends on the value of the `DATA_LENGTH` field.
  * Checksum is 2 bytes of data representing the sum of the header + all data in the data portion of the packet
  * Status byte is a bitfield byte indicating various status of the printer itself. (e.g. If it is still printing)

### Gameboy Printer Timing

Below measurements was obtained via the ANALOG DISCOVERY via digilent

```
                       1.153ms
        <--------------------------------------->
         0   1   2   3   4   5   6   7             0   1   2   3   4   5   6   7
     __   _   _   _   _   _   _   _   ___________   _   _   _   _   _   _   _   _
CLK:   |_| |_| |_| |_| |_| |_| |_| |_|           |_| |_| |_| |_| |_| |_| |_| |_|
DAT: ___XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX____________XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX_
       <-->                           <---------->
       127.63 us                         229.26 us

```


* Clock Frequency: 8kHz (127.63 us)
* Transmission Speed: 867 baud (1.153ms per 8bit symbol)
* Between Symbol Period: 229.26 us

![](gameboy_to_ghost_printer.png)

## Research/Dev log

### 2017-11-30

* Time to wrap this up. I have pushed the arduino to it's maxium ability. The show stopper to futher dev is the puny ram.
* Still cannot get CRC working. But don't really care anymore. Since its taking too much time to debug crc. Works good enough.
* There is not enough time/ram to actually do much processing of the image in the arduino. Instead the image data has to be transferred raw to over serial at 115200baud via hex, and processed futher in the computer.


### 2017-4-12

* Checksum works for init, inqiry, but not data, and possibly inqury. Possibly messed up the summation somehow. But I found that this may not matter, as the gameboy camera doesn't seem to check for it.
* While checksum worked for short messages, the checksum system didn't work for the normal data bytes. But... it seems that the gameboy printer doesn't actually pay attention to the checksum bit.
* Investigation with http://www.huderlem.com/demos/gameboy2bpp.html shows that this is actually encoded as "16 byte standard gameboy tile format".
* According to http://furrtek.free.fr/?a=gbprinter&i=2 the `The GameBoy Camera buffers tile data by blocs of 2 20 tiles wide lines.`.

## My Face In BW

![](./sample_image/bk_portrait.png)


<details>

```
# GAMEBOY PRINTER EMULATION PROJECT
# By Brian Khuu (2017)
!INIT: length: 0 | CRC: 1 | CRC CALC: 1 (0 1) | crc raw: 0 1 |Printer Status:  |
!DATA: length: 640 | CRC: 1 | CRC CALC: 366 (1 110) | crc raw: 0 1 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF F8 FF E0 F0 C1 E0 80 C3 80 83 00 83 80 03
FF FF 7F FF 5F 3F 8F 1F 4F BF 3F 1F 0A 14 44 30
FF FF FF FF FF FF FF FF FF FF 3E FF 3C 18 D0 09
FF FF FF FF FF FF EF F7 F3 E7 76 E3 10 26 B4 00
FF FF FF FF FF FF FF FF FF FF E3 FF E3 41 01 C8
FF FF FF FF FF FF BF FF 3E 9F 1C 9E 0C 0E 0E 9C
FF FF FF FF FF FF 07 FF 03 07 51 23 78 73 FF 7E
FF FF FF FF FF FF FF FF FF FF 8D FF 04 0C 60 04
FF FF FF FF FF FF FF FF FF FF 93 FF 01 01 48 01
FF FF FF FF FF FF FF FF FF FF C7 FF C6 83 22 91
FF FF FF FF FF FF FF FF FF FF 5C FF 00 08 3B 00
FF FF FF FF FF FF FF FF FF FF 7F FF 3F 7F 1F 3F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
81 03 00 83 80 83 81 C2 C0 E0 F0 F0 FF FF FF FF
C4 E1 0C 01 00 0C 0E 0C 1E 1F 7F 3F FF FF FF FF
CB C1 C9 C3 18 C1 0C 18 FD 3E FF FF FF FF FF FF
D1 A0 90 E0 36 80 12 26 AE 77 FF FF FF FF FF FF
D9 80 5E 81 DA 0D 60 01 EB 77 FF FF FF FF FF FF
1E 9C 1E 9C 0C 9E 0E 87 BF CF FF FF FF FF FF FF
F2 7F 79 72 50 22 03 06 3F CF FF FF FF FF FF FF
C2 04 60 04 00 64 00 04 CD 36 FF FF FF FF FF FF
0D C8 1D C8 1D C8 9C C9 EB DD FF FF FF FF FF FF
32 01 3C 03 34 9B C0 83 D3 EF FF FF FF FF FF FF
77 38 7B 30 70 33 70 30 BA 7D FF FF FF FF FF FF
3F 3F 3F 3F 1F 3F 3F 1F DF BF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 8 | CRC CALC: 2394 (9 90) | crc raw: 0 8 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
2E 51 FF 00 AB 54 FF 00 7E 01 FF 00 7C 01 FF 00
E6 11 FF 00 AB 54 FF 00 6E 11 FF 00 DD 00 FF 00
EE 11 FF 00 BF 40 FF 00 EF 10 FF 00 DD 00 FF 00
F7 00 FF 00 DD 00 FF 00 77 00 FF 00 ED 10 FF 00
75 00 FF 00 55 00 FF 00 55 00 FF 00 55 00 FF 00
55 00 FB 00 55 00 EA 00 55 00 B0 00 54 00 A8 00
55 00 AA 00 55 00 88 00 51 00 00 00 00 00 00 00
54 01 B9 03 57 02 AE 05 56 09 38 02 10 05 0A 00
57 AD BF 4A 3E D5 7F A9 5D 52 9F A4 7D 00 1E 00
DE 3D 6F F3 64 9F 96 79 DB 84 BF 00 5A 05 FF 00
6B F7 8D FE DC 73 F7 88 2F 50 DF 20 BF 40 FF 00
EB 1C 7F C0 FB 04 FE 00 E5 10 DB 20 95 40 EA 00
55 00 BE 00 55 00 E8 00 55 00 AA 00 55 00 A8 00
55 00 AB 00 55 00 FA 00 5D 00 A3 00 55 00 8A 00
55 00 BB 00 55 00 AF 00 55 00 BB 00 75 00 AF 00
77 00 FF 00 5D 00 FF 00 57 00 BF 00 5F 00 FF 00
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FB F7 F1 E3 C8 E3 F2 E2 F6 FC F8 FC FD F9 FC F9
FF FF FF FF EF 1F 07 0F 0F 07 07 07 E7 F7 17 E7
77 00 FF 00 DD 00 FF 00 77 00 FF 00 9D 40 FF 00
F6 01 FF 00 FF 00 FF 00 77 00 FF 00 F9 04 FF 00
E7 10 FF 00 FD 00 FF 00 77 00 FF 00 55 00 FF 00
77 00 FF 00 D5 00 FF 00 77 00 FF 00 55 00 FF 00
77 00 FB 00 55 00 FE 00 55 00 BA 00 55 00 FA 00
50 00 A0 00 50 00 80 00 40 00 00 00 40 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
15 00 03 00 05 00 00 00 01 00 00 00 00 00 00 00
56 01 BF 00 55 00 8F 00 55 00 2B 00 45 00 0B 00
3E 41 FF 00 F9 04 FA 00 65 10 E6 00 DD 00 B0 00
55 00 A2 00 55 00 C8 00 15 40 48 40 14 40 F0 00
55 00 00 00 55 00 80 00 11 00 00 00 00 00 00 00
55 00 22 00 55 00 00 00 11 00 00 00 01 00 00 00
57 00 AB 00 55 00 AF 00 55 00 2F 00 56 01 3F 00
76 01 FF 00 6B 14 FE 00 75 00 FA 00 D5 00 AA 00
FF FF FF FF FF FF FF FF F0 FF E0 F0 F0 E0 F0 E0
FF FF FF FF FF FF FF FF 07 FF 07 03 03 03 07 03
!DATA: length: 640 | CRC: 238 | CRC CALC: 61220 (239 36) | crc raw: 0 238 |Printer Status: Ready To Print, Checksum Error,  |
F8 FC FE FC FF FE E0 FF C0 E0 C0 E0 C0 E0 E1 FC
07 07 0F 07 1F 0F 47 BF 07 07 07 07 07 07 17 E7
77 00 FF 00 7D 00 FF 00 67 10 FF 00 BF 40 FF 00
7F 00 FF 00 DD 00 FF 00 77 00 FF 00 9D 40 FF 00
77 00 FF 00 D5 00 FF 00 75 00 FF 00 D5 00 FF 00
75 00 FF 00 55 00 FF 00 55 00 FF 00 55 00 FF 00
55 00 BA 00 55 00 EA 00 55 00 BA 00 55 00 FA 00
40 00 00 00 40 00 00 00 00 00 00 00 50 00 A0 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 01 00 03 00
00 00 00 00 00 00 00 00 11 02 37 0C 5A 05 F5 0A
01 00 00 00 00 00 00 00 00 00 48 B0 35 CF 04 FB
55 00 03 00 09 00 0E 01 03 03 0E 0D FB 6F 17 FF
50 10 30 80 A1 50 06 F8 48 F7 A2 FC 55 FF BE FF
00 00 00 00 01 00 05 0A 34 30 E0 C0 5A 8D 60 E0
01 00 00 00 41 00 83 00 06 01 2E 00 5C 00 20 00
7D 00 B8 00 34 40 C0 00 00 00 00 00 00 00 00 00
55 00 00 00 44 00 00 00 00 00 00 00 00 00 00 00
F0 E0 F0 E0 F0 E0 F1 E0 F0 E0 F0 E0 F0 E0 F0 E0
07 03 87 03 9B 83 E3 F3 07 03 87 43 DB 83 7B C3
F9 FC FC FC FE FC FE FE FF FF FC FE FC FC F8 FC
17 E7 07 07 0F 07 1F 0F 5F BF 07 07 07 07 07 07
67 10 FF 00 2F 50 FF 00 7F 00 FF 00 2F 50 FF 00
F6 01 FF 00 DD 00 FF 00 77 00 FF 00 DD 00 FF 00
75 00 FF 00 D5 00 FF 00 75 00 FF 00 D5 00 FF 00
55 00 FB 00 5D 00 FF 00 55 00 FF 00 55 00 FF 00
55 00 BB 00 55 00 FB 00 57 00 BF 00 5D 00 FE 00
50 00 A0 00 54 00 A8 00 55 00 BB 00 55 00 EE 01
00 00 02 00 15 00 7F 00 2A 55 0C F3 22 DD 00 FF
57 00 BF 00 5E 01 FD 02 AA 55 40 BF 84 7F 00 FF
EA 15 C4 3B A8 57 00 FF 29 D7 00 FF 21 DF 0B FF
80 7F 00 FF 45 FF 02 FF D5 7F FF FF DD FF FF FF
57 FF 2A FF 5F FF DE FF 77 FF AF FF 7D FF EF FF
FD FF 3C FF 5D FF FB FF 7F FF FF FF DD FF EF FF
10 F0 08 F8 15 FE AE FE 15 FF AB FF 55 FF FB FF
40 00 00 00 00 00 00 00 00 00 00 00 C0 80 80 80
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F0 E0 F0 E0 F0 E0 F3 E0 F0 E0 F0 E0 F0 E0 F0 E0
73 23 83 03 83 73 F3 63 73 03 D7 03 9B 83 53 83
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 65 | CRC CALC: 16872 (65 232) | crc raw: 0 65 |Printer Status: Ready To Print, Checksum Error,  |
F8 FC FD FC FE FC FC FC FC FC FE FD FE FF FE FC
07 FF FF FF 0F 07 07 07 07 07 FF 07 8F 7F 67 0F
3F 40 F7 08 AB 54 DF 20 2B 54 C6 39 AA 55 C1 3E
77 00 FF 00 D5 00 FF 00 77 00 FF 00 DF 00 FF 00
75 00 FF 00 D5 00 FF 00 75 00 FF 00 9D 40 FF 00
55 00 BB 00 55 00 FE 00 55 00 BB 00 55 00 FC 00
75 00 FB 00 55 00 EE 00 55 00 BB 00 55 00 AE 00
57 01 BA 03 51 07 FE 03 55 07 B2 0F 55 0F F8 0F
50 FF 00 FF 55 FF C8 FF 55 FF 8A FF 55 FF BE FF
55 FF 01 FF 55 FF 06 FF 57 FF B7 FF 55 FF CF FF
15 FF AB FF 55 FF EA FF 51 FF E8 FF D2 FD 85 FA
57 FF BF FF 55 FF 80 FF 82 7D 00 FF AA 55 7F 80
75 FF A0 FF 4A F5 45 BA A6 59 57 A8 AA 55 F7 08
77 FF 3F FF 97 7F 43 BF A5 5F D1 2F AA 55 70 8F
55 FF FE FF 55 FF FE FF D7 FF FF FF 7D FF FA FF
40 C0 40 C0 70 E0 A0 E0 40 F0 90 F0 50 F0 D0 F0
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0
53 23 67 03 13 E3 1B 83 93 63 07 03 F3 63 9B 83
FC FC FD F8 FD F9 F9 FC FC FC F8 FE FC FB F1 FB
67 07 77 27 67 37 57 27 07 07 07 0F EF 1F FF FF
AA 55 FF 00 AB 54 FF 00 2F 50 FF 00 BF 40 FF 00
AA 55 FF 00 FD 00 FF 00 E7 10 FF 00 D5 00 FF 00
B5 40 FF 00 D5 00 FE 00 55 00 B8 00 54 00 80 00
57 00 FA 00 55 00 AB 00 55 00 00 00 00 00 00 00
55 00 3F 00 55 00 FE 01 55 00 2B 00 15 00 00 00
55 0F BB 0F 55 0F 5A 2F 41 3F 22 3F 05 3F 20 3F
55 FF B3 FF 55 FF FE FF 55 FF BF FF 5D FF BF FF
55 FF AB FF 55 FF 8A FF 51 FF 60 FF 42 FD C5 FA
0A F5 01 FE 2B D4 1F E0 AE 51 7F 80 AF 50 FF 00
AF 50 FF 00 FA 05 FF 00 FF 00 FF 00 FF 00 FF 00
EE 11 DF 20 BF 40 FF 00 FE 01 DF 20 BF 40 FF 00
AB 55 FC 03 BA 45 F7 08 EA 15 FE 01 BA 45 FF 00
1D FF 3F FF DD 7F 0F FF 87 7F 43 BF A1 5F 53 AF
58 F0 D0 F0 D0 F0 F8 F8 DC F8 F8 F8 DC F8 FC FC
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
30 00 25 1A 1B 14 05 1A 1B 04 0F 08 01 00 0F 00
F0 E0 F0 E0 EF F0 FF FF F7 E7 E1 F3 F1 F8 F8 FC
73 83 8B 77 F7 0F FF FF FF FF FF FF FF FF 07 7F
!DATA: length: 640 | CRC: 22 | CRC CALC: 5986 (23 98) | crc raw: 0 22 |Printer Status: Ready To Print, Checksum Error,  |
E0 F0 E0 F0 E9 F0 F1 FB FF FA FE FC FC FC F8 FC
07 07 07 07 FF 07 0F FF FF 07 07 07 07 07 F7 0F
6A 15 FF 00 6C 11 FE 01 77 01 FF 01 5D 01 FE 01
75 00 FF 00 D5 00 7E 00 35 40 7A 00 55 00 7F 00
40 00 00 00 00 00 00 00 00 00 00 00 00 00 80 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
29 37 03 3F 2D 17 10 2F 09 17 33 0F 09 17 20 1F
55 FF 3F FF 55 FF 6F FF 57 FF EE FF 56 FD AD FE
6A D5 57 A8 2A D5 7F 80 AA 55 FC 03 A8 57 C0 3F
AB 54 FD 02 AB 54 FF 00 AA 55 20 FF 55 FF 2F FF
77 00 FF 00 DF 00 FF 00 EE 11 15 EA 4A F5 A1 FE
FB 04 FF 00 BF 40 FF 00 FE 01 F5 0A AA 55 D4 2B
FE 01 FF 00 FF 00 F5 0A A8 57 2B FF 1D FF 2F FF
AB 57 53 AF 29 D7 D1 2F 59 F7 21 FF 5A F5 A6 FB
5C FC FE FC D4 FC EE FC F4 FC FE FC DC FC FC FC
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
01 04 07 00 03 04 03 04 07 00 01 02 03 00 00 03
FA FC E9 F0 F1 E3 EE F0 F8 F0 F4 E3 F3 E7 F3 E7
07 03 3F C3 FF FF DF 3F 17 0F 43 87 F7 E3 E3 F3
FD FC FC FC FC FC FC FC FC FC FF FF D0 E4 C0 E4
FF FF 07 FF 07 07 07 07 07 07 FF FF 07 07 07 07
74 01 FF 00 54 01 2F 00 15 00 00 00 00 00 00 00
75 00 7E 00 3F 40 7F 00 57 00 3F 00 15 00 00 00
00 00 00 00 40 00 E8 00 55 00 A2 00 55 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
01 1F 01 1F 11 1F 10 1F 11 1F 11 1F 15 1F 10 1F
56 FD F1 FE 52 FD CB FC 52 FD B7 F8 52 FD A7 F8
E1 1F C8 37 A2 5D 43 BC AB 54 DF 20 AA 55 53 AF
41 FF 00 FF A2 5D F5 0A AA 55 DC 23 7F FF 7C FB
5A F5 00 FF AA 55 14 EB AA 55 47 B8 2B D4 97 E8
AA 55 50 AF AA 55 55 AA AA 55 7C 83 FA 05 FC 03
01 FF 40 BF AA 55 55 AA 0A F5 4F BF 3F FF 6A FF
51 FF 50 AF E8 17 50 AF A8 57 94 EB F2 7D 01 FE
F4 FC FE FC 7C FC FE FC 7C FC 7E FE DC 7F 7F FF
00 00 00 00 01 00 07 03 1F 0F 7B 3F D5 FF E2 FF
07 0F 3F 3F FD FF F8 FF 75 FF FE FF DF FF FF FF
E5 F3 F9 F0 F8 FC FE FF F9 FF E0 F0 F4 E2 F4 E2
E3 F3 F7 03 0F 07 8F 7F 87 8F 23 07 77 23 63 73
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 14 | CRC CALC: 3808 (14 224) | crc raw: 0 14 |Printer Status: Ready To Print, Checksum Error,  |
C0 E4 E4 FF EC F3 C0 E0 C0 E0 D0 E0 FF FF FA FC
07 07 07 FF 17 EF 07 07 07 07 07 07 97 0F 1F 3F
00 00 00 00 00 00 00 00 01 00 01 00 15 00 0B 00
00 00 00 00 10 00 3F 00 55 00 3A 00 2A 55 F7 08
00 00 00 00 00 00 E8 00 55 00 29 00 D5 00 C0 00
00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
11 1F 01 1F 01 1F 12 0F 1D 17 02 1F 4D 17 03 1F
7E F1 AF F0 5D F0 FF F0 57 F0 EF F0 5D F0 FF F0
A5 5F F4 0B FB 04 FF 00 7E 01 FF 00 DF 00 FF 00
F1 7F 00 FF AA 55 75 8A AA 55 D5 2A BB 44 FF 00
6B D4 57 A8 AF 50 7F 80 EF 10 FF 00 EF 10 FF 00
7E 01 FD 02 5A 05 FF 00 7F 00 FF 00 5E 01 FF 00
95 7F 55 AA AA 55 F5 0A FA 05 FF 00 BB 44 FF 00
AA 55 55 AA AA 55 75 8A AA 55 F7 08 AA 55 FF 00
FC 7F 3C FB FF 79 3A F9 F1 71 E0 61 D1 61 20 C1
59 F7 23 FF 15 FB 37 F9 93 7D 57 B8 1C FD 0A FC
57 FF 7B FF 55 FF 92 FF C1 FF F0 EB EF F1 F0 FF
F0 E0 EE F0 FF FF FF FF F7 FF E3 F7 F2 E7 F3 E6
73 03 03 03 97 E3 FF FF F7 FF E7 F3 A3 73 63 33
F0 F8 D1 E0 C0 E0 C0 E0 C0 E0 FF FF FF FF FF FF
7F FF F7 0F 07 07 07 07 07 07 FF FF FF FF FF FF
07 00 03 00 15 00 08 00 11 00 00 00 00 00 80 00
75 00 FF 00 55 00 08 00 54 00 00 00 04 00 00 00
55 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
50 00 00 00 15 00 02 00 01 00 00 00 01 00 00 00
01 00 02 00 55 00 AA 00 55 00 2A 00 55 00 AA 00
41 1F 10 0F 52 0D C8 0F 5B 0D 48 0F 4A 0D CC 0B
5F F0 FF F0 5F F0 57 F8 FB 54 17 F8 22 DD 1D EA
7F 00 FF 00 CF 10 FF 00 EF 10 FF 00 FF 00 FF 00
F6 01 FF 00 FF 00 FF 00 F6 01 FF 00 FE 01 FF 00
EE 11 FF 00 AB 54 C7 38 AA 55 1F E0 7A C5 1F E0
7E 01 FD 02 DE 01 FF 00 76 01 FF 00 FA 05 FF 00
AE 51 5F A0 EE 11 17 E8 8A 75 D5 2A CA 35 45 BA
EA 15 FD 02 EA 15 FE 01 FB 05 DD 23 EB 15 76 8A
91 40 00 C0 C0 00 80 00 00 00 00 00 00 00 00 00
1F FC 8C FE 87 FC 8C FF 16 FF 8A FF 8F 7C 8C 73
5D F7 E3 7F 58 77 69 3E 7E 31 93 2F 3F 5F 8F FF
F3 E6 F0 E0 EC F0 FE FF F0 FF F0 E0 E0 F0 F8 FC
63 33 03 33 03 03 1F E3 1B E7 03 03 C3 7F 5F 3F
!DATA: length: 640 | CRC: 15 | CRC CALC: 3858 (15 18) | crc raw: 0 15 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF F1 F8 E0 F0 F4 E2 F0 E6 E0 F0 F9 F0
FF FF FF FF 87 0F 07 07 37 67 37 67 27 07 87 0F
00 00 00 00 00 00 80 00 00 00 00 00 00 00 80 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
15 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
7E 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00
C8 0F 48 0F 4A 0D 08 0F 08 0F 08 0F 0D 0F 28 0F
A2 7D 1B F4 1B FC 0F FC 16 FD 1F FE 6E DF 0F FF
AA 55 7F 80 BA 45 77 88 AA 55 D5 2A AA 55 D0 2F
AA 55 FC 03 AA 55 75 8A AA 55 45 BA AA 55 10 EF
AA 55 02 FF 95 7F 00 FF 81 7F 41 BF 03 FD A8 FF
EA 15 05 FB 21 DF 80 FF 59 F7 88 FF 50 FF 04 FB
28 D7 84 FB 22 DD 10 EF A8 57 80 FF 00 FF 0C FF
AB 56 40 BE AC 54 04 FC AC 54 00 F8 2C D8 48 B8
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
75 4F 38 7F 03 7F AF FF DF FF 7F 7F 3F 7F 2D 3E
D7 3F AF FF DF FF FF FF 57 FF FF FF 5F FF 7F FF
FF FE FC FF E8 F0 F0 E0 F4 F8 FC FF FF FF FF FF
1F 0F 83 07 37 0F BF 7F 6F 1F 87 03 C3 E3 F3 FF
FF FF FC FF F8 F0 E0 F0 F4 E3 F2 E7 F0 E0 F8 F0
FF FF 3F FF 17 0F 07 07 57 27 77 27 47 27 47 6F
00 00 00 00 00 00 80 00 00 00 00 00 00 00 80 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
05 0F 26 0F 0D 07 2C 07 25 07 26 07 05 07 24 07
1F F7 0F FF 5F FF BF FF 7D FF FF FF FF FF FF FF
AA 55 00 FF AA D5 C0 FF C8 F7 D0 FF F0 DF CE FF
AD 57 01 FF A5 5F 00 FF 05 FF 40 BF 42 FD 80 FF
4A F5 F0 FF DD FF 80 FF 12 FD 00 FF 15 FF 0A FF
80 7F 08 FF 55 FF 01 FE A8 57 02 FF 00 FF EE FF
1C FF F8 FF 54 FF 08 FF 01 FF 8A FF 01 FF 28 FF
AF 5E 0F FF 1F FF 1F FF 3F FF 3F FF 7F FF FF FF
00 00 00 00 00 00 80 80 80 C0 E0 C0 E0 C0 E0 E0
35 38 3A 39 3D 3F 1F 3F 15 1F 38 1F 00 1F 0B 1F
FF 7F BF FF 75 FF F2 FF 51 FF A1 FF 71 FF E3 FF
F8 FF E0 F0 F0 E0 F4 F8 FD FE FF FF EB F7 F3 E7
C7 03 1B 07 3F 9F 9F 1F 1F 0F 47 83 F3 E3 D3 BF
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 200 | CRC CALC: 51362 (200 162) | crc raw: 0 200 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FC FF F8 F0 E0 F0 F4 E3 F2 E7 F0 E0 F8 F0
FF FF 3F FF 17 0F 07 07 57 27 77 27 47 27 47 6F
00 00 00 00 00 00 80 00 00 00 00 00 00 00 80 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
05 07 27 07 15 07 27 07 17 07 07 07 1D 1F FF 7F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
E5 DF C7 FF D7 FF CB FF 57 FF FF FF FD FF FF FF
51 FF 20 FF 55 FF E8 FF F1 FF F8 FF FD FF FF FF
91 7F 21 FF 54 FF 08 FF 17 FF 2B FF 5F FF BF FF
74 FF 22 FF 45 FF 8A FF 5D FF FB FF 5D FF FF FF
45 FF 3F FF 5F FF 7F FF 7F FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FB FF FF FF FF FF
E0 F0 F0 F0 F0 F0 FE FC FF FF FF FF FF FF FF FF
17 0F 1F 0F 1F 0F 0F 0F 17 0F C7 8F D5 CF FF FF
D7 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F3 E7 F3 E7 E7 F3 E3 F3 F1 F9 FC F8 FE FC FE FF
83 87 93 83 E3 93 E3 F3 E7 F3 43 07 07 0F 5F BF
FF FF FF E0 F0 E0 E4 F3 FB F7 FF FF FF FF FF FF
FF FF FF 07 07 07 07 FF FF FF FF FF FF FF FF FF
00 00 00 00 55 00 B9 46 7D FF BF FF 5F FF FF FF
00 00 00 00 54 00 04 FB 7F FF FF FF 5D FF FF FF
00 00 00 00 04 00 0F FF FD FF FB FF D5 FF E3 FF
00 00 00 00 00 00 C1 C0 C7 C3 FE CF 7D FF EF FF
01 01 07 03 1F 0F FF FF FF FF FF FF 57 FF FF FF
FF FF FF FF FF FF FF FF 7F FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF EF DF FB F7
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
DF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF 7F FF FF FF FF FF FF FF
F7 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!DATA: length: 640 | CRC: 21 | CRC CALC: 5800 (22 168) | crc raw: 0 21 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FE C1 CC 80 DE 8C
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FB 47
FF FF FF FF FF FF FF FF FF FF FC FE FE FC FC 86
FF FF FF FF FF FF FF FF FF FF 7F FF FF 7F FB 4C
FF FF FF FF FF FF FF FF FF FF FF F3 E3 F3 B0 63
FF FF FF FF FF FF FF FF FF FF FD E0 C6 E0 C3 E6
FF FF FF FF FF FF FF FF FF FF 7E FF 3E 7F 5E 21
FF FF FF FF FF FF FF FF FF FF 3F 7F 7F 3F 37 68
FF FF FF FF FF FF FF FF FF FF FF CF C7 CF CB 86
FF FF FF FF FF FF FF FF FC FF F8 F0 F8 F0 FA 10
FF FF FF FF FF FF FF FF 03 FF 03 01 03 01 C3 01
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
C0 8C C0 80 C1 83 DF 8F DF 8F BF CF DF FF FF FF
43 81 31 99 B5 18 B0 19 1B 81 A7 C3 E7 FF FF FF
84 02 60 32 7C 3E 78 32 30 02 4C 86 CE FF FF FF
D8 08 3A 11 18 10 49 10 80 49 5A EC FE FF FF FF
00 21 A3 13 07 13 E7 33 A2 11 78 31 79 FF FF FF
C1 E6 C0 E0 C0 E1 C7 E7 C3 E7 FF E7 FF FF FF FF
61 00 6B 46 EF C6 EF C6 EF C6 E6 CF EF FF FF FF
20 20 22 24 2E 24 2E 24 2E 24 BD 66 7E FF FF FF
C2 04 C0 4C C0 4C C0 4C CC 40 C5 62 E7 FF FF FF
0A 13 12 C3 13 02 EB 12 9B 42 38 12 30 F8 F8 FF
43 81 03 01 03 01 03 01 03 01 03 01 01 03 03 FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!DATA: length: 0 | CRC: 4 | CRC CALC: 4 (0 4) | crc raw: 0 4 |Printer Status: Ready To Print,  |
!PRNT: 01 13 E4 40 | : length: 4 | CRC: 1 | CRC CALC: 380 (1 124) | crc raw: 0 1 |Printer Status: Print Reqested, Currently Printing, Checksum Error,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
# Finished Pretending To Print for fun!
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested,  |
```

</details>

## White Screen

![](./gbp_white_screen.png)

<details>

```
# GAMEBOY PRINTER EMULATION PROJECT
# By Brian Khuu (2017)
!INIT: length: 0 | CRC: 1 | CRC CALC: 1 (0 1) | crc raw: 0 1 |Printer Status:  |
!DATA: length: 640 | CRC: 1 | CRC CALC: 366 (1 110) | crc raw: 0 1 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF F8 FF E0 F0 C1 E0 80 C3 80 83 00 83 80 03
FF FF 7F FF 5F 3F 8F 1F 4F BF 3F 1F 0A 14 44 30
FF FF FF FF FF FF FF FF FF FF 3E FF 3C 18 D0 09
FF FF FF FF FF FF EF F7 F3 E7 76 E3 10 26 B4 00
FF FF FF FF FF FF FF FF FF FF E3 FF E3 41 01 C8
FF FF FF FF FF FF BF FF 3E 9F 1C 9E 0C 0E 0E 9C
FF FF FF FF FF FF 07 FF 03 07 51 23 78 73 FF 7E
FF FF FF FF FF FF FF FF FF FF 8D FF 04 0C 60 04
FF FF FF FF FF FF FF FF FF FF 93 FF 01 01 48 01
FF FF FF FF FF FF FF FF FF FF C7 FF C6 83 22 91
FF FF FF FF FF FF FF FF FF FF 5C FF 00 08 3B 00
FF FF FF FF FF FF FF FF FF FF 7F FF 3F 7F 1F 3F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
81 03 00 83 80 83 81 C2 C0 E0 F0 F0 FF FF FF FF
C4 E1 0C 01 00 0C 0E 0C 1E 1F 7F 3F FF FF FF FF
CB C1 C9 C3 18 C1 0C 18 FD 3E FF FF FF FF FF FF
D1 A0 90 E0 36 80 12 26 AE 77 FF FF FF FF FF FF
D9 80 5E 81 DA 0D 60 01 EB 77 FF FF FF FF FF FF
1E 9C 1E 9C 0C 9E 0E 87 BF CF FF FF FF FF FF FF
F2 7F 79 72 50 22 03 06 3F CF FF FF FF FF FF FF
C2 04 60 04 00 64 00 04 CD 36 FF FF FF FF FF FF
0D C8 1D C8 1D C8 9C C9 EB DD FF FF FF FF FF FF
32 01 3C 03 34 9B C0 83 D3 EF FF FF FF FF FF FF
77 38 7B 30 70 33 70 30 BA 7D FF FF FF FF FF FF
3F 3F 3F 3F 1F 3F 3F 1F DF BF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 111 | CRC CALC: 28886 (112 214) | crc raw: 0 111 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FB F7 F1 E3 C8 E3 F2 E2 F6 FC F8 FC FD F9 FC F9
FF FF FF FF EF 1F 07 0F 0F 07 07 07 E7 F7 17 E7
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
FF FF FF FF FF FF FF FF F0 FF E0 F0 F0 E0 F0 E0
FF FF FF FF FF FF FF FF 07 FF 07 03 03 03 07 03
!DATA: length: 640 | CRC: 78 | CRC CALC: 20264 (79 40) | crc raw: 0 78 |Printer Status: Ready To Print, Checksum Error,  |
F8 FC FE FC FF FE E0 FF C0 E0 C0 E0 C0 E0 E1 FC
07 07 0F 07 1F 0F 47 BF 07 07 07 07 07 07 17 E7
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F0 E0 F0 E0 F0 E0 F1 E0 F0 E0 F0 E0 F0 E0 F0 E0
07 03 87 03 9B 83 E3 F3 07 03 87 43 DB 83 7B C3
F9 FC FC FC FE FC FE FE FF FF FC FE FC FC F8 FC
17 E7 07 07 0F 07 1F 0F 5F BF 07 07 07 07 07 07
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F0 E0 F0 E0 F0 E0 F3 E0 F0 E0 F0 E0 F0 E0 F0 E0
73 23 83 03 83 73 F3 63 73 03 D7 03 9B 83 53 83
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 90 | CRC CALC: 23040 (90 0) | crc raw: 0 90 |Printer Status: Ready To Print, Checksum Error,  |
F8 FC FD FC FE FC FC FC FC FC FE FD FE FF FE FC
07 FF FF FF 0F 07 07 07 07 07 FF 07 8F 7F 67 0F
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0
53 23 67 03 13 E3 1B 83 93 63 07 03 F3 63 9B 83
FC FC FD F8 FD F9 F9 FC FC FC F8 FE FC FB F1 FB
67 07 77 27 67 37 57 27 07 07 07 0F EF 1F FF FF
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F0 E0 F0 E0 EF F0 FF FF F7 E7 E1 F3 F1 F8 F8 FC
73 83 8B 77 F7 0F FF FF FF FF FF FF FF FF 07 7F
!DATA: length: 640 | CRC: 86 | CRC CALC: 22078 (86 62) | crc raw: 0 86 |Printer Status: Ready To Print, Checksum Error,  |
E0 F0 E0 F0 E9 F0 F1 FB FF FA FE FC FC FC F8 FC
07 07 07 07 FF 07 0F FF FF 07 07 07 07 07 F7 0F
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
FA FC E9 F0 F1 E3 EE F0 F8 F0 F4 E3 F3 E7 F3 E7
07 03 3F C3 FF FF DF 3F 17 0F 43 87 F7 E3 E3 F3
FD FC FC FC FC FC FC FC FC FC FF FF D0 E4 C0 E4
FF FF 07 FF 07 07 07 07 07 07 FF FF 07 07 07 07
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
E5 F3 F9 F0 F8 FC FE FF F9 FF E0 F0 F4 E2 F4 E2
E3 F3 F7 03 0F 07 8F 7F 87 8F 23 07 77 23 63 73
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 85 | CRC CALC: 22048 (86 32) | crc raw: 0 85 |Printer Status: Ready To Print, Checksum Error,  |
C0 E4 E4 FF EC F3 C0 E0 C0 E0 D0 E0 FF FF FA FC
07 07 07 FF 17 EF 07 07 07 07 07 07 97 0F 1F 3F
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F0 E0 EE F0 FF FF FF FF F7 FF E3 F7 F2 E7 F3 E6
73 03 03 03 97 E3 FF FF F7 FF E7 F3 A3 73 63 33
F0 F8 D1 E0 C0 E0 C0 E0 C0 E0 FF FF FF FF FF FF
7F FF F7 0F 07 07 07 07 07 07 FF FF FF FF FF FF
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F3 E6 F0 E0 EC F0 FE FF F0 FF F0 E0 E0 F0 F8 FC
63 33 03 33 03 03 1F E3 1B E7 03 03 C3 7F 5F 3F
!DATA: length: 640 | CRC: 87 | CRC CALC: 22576 (88 48) | crc raw: 0 87 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF F1 F8 E0 F0 F4 E2 F0 E6 E0 F0 F9 F0
FF FF FF FF 87 0F 07 07 37 67 37 67 27 07 87 0F
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
FF FE FC FF E8 F0 F0 E0 F4 F8 FC FF FF FF FF FF
1F 0F 83 07 37 0F BF 7F 6F 1F 87 03 C3 E3 F3 FF
FF FF FC FF F8 F0 E0 F0 F4 E3 F2 E7 F0 E0 F8 F0
FF FF 3F FF 17 0F 07 07 57 27 77 27 47 27 47 6F
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F8 FF E0 F0 F0 E0 F4 F8 FD FE FF FF EB F7 F3 E7
C7 03 1B 07 3F 9F 9F 1F 1F 0F 47 83 F3 E3 D3 BF
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 104 | CRC CALC: 26968 (105 88) | crc raw: 0 104 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FC FF F8 F0 E0 F0 F4 E3 F2 E7 F0 E0 F8 F0
FF FF 3F FF 17 0F 07 07 57 27 77 27 47 27 47 6F
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
F3 E7 F3 E7 E7 F3 E3 F3 F1 F9 FC F8 FE FC FE FF
83 87 93 83 E3 93 E3 F3 E7 F3 43 07 07 0F 5F BF
FF FF FF E0 F0 E0 E4 F3 FB F7 FF FF FF FF FF FF
FF FF FF 07 07 07 07 FF FF FF FF FF FF FF FF FF
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!DATA: length: 640 | CRC: 21 | CRC CALC: 5800 (22 168) | crc raw: 0 21 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FE C1 CC 80 DE 8C
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FB 47
FF FF FF FF FF FF FF FF FF FF FC FE FE FC FC 86
FF FF FF FF FF FF FF FF FF FF 7F FF FF 7F FB 4C
FF FF FF FF FF FF FF FF FF FF FF F3 E3 F3 B0 63
FF FF FF FF FF FF FF FF FF FF FD E0 C6 E0 C3 E6
FF FF FF FF FF FF FF FF FF FF 7E FF 3E 7F 5E 21
FF FF FF FF FF FF FF FF FF FF 3F 7F 7F 3F 37 68
FF FF FF FF FF FF FF FF FF FF FF CF C7 CF CB 86
FF FF FF FF FF FF FF FF FC FF F8 F0 F8 F0 FA 10
FF FF FF FF FF FF FF FF 03 FF 03 01 03 01 C3 01
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
C0 8C C0 80 C1 83 DF 8F DF 8F BF CF DF FF FF FF
43 81 31 99 B5 18 B0 19 1B 81 A7 C3 E7 FF FF FF
84 02 60 32 7C 3E 78 32 30 02 4C 86 CE FF FF FF
D8 08 3A 11 18 10 49 10 80 49 5A EC FE FF FF FF
00 21 A3 13 07 13 E7 33 A2 11 78 31 79 FF FF FF
C1 E6 C0 E0 C0 E1 C7 E7 C3 E7 FF E7 FF FF FF FF
61 00 6B 46 EF C6 EF C6 EF C6 E6 CF EF FF FF FF
20 20 22 24 2E 24 2E 24 2E 24 BD 66 7E FF FF FF
C2 04 C0 4C C0 4C C0 4C CC 40 C5 62 E7 FF FF FF
0A 13 12 C3 13 02 EB 12 9B 42 38 12 30 F8 F8 FF
43 81 03 01 03 01 03 01 03 01 03 01 01 03 03 FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!DATA: length: 0 | CRC: 4 | CRC CALC: 4 (0 4) | crc raw: 0 4 |Printer Status: Ready To Print,  |
!PRNT: 01 13 E4 40 | : length: 4 | CRC: 1 | CRC CALC: 380 (1 124) | crc raw: 0 1 |Printer Status: Print Reqested, Currently Printing, Checksum Error,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
# Finished Pretending To Print for fun!
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested,  |

```

</details>

## Black Screen

![](./gbp_black_screen.png)

<details>

```
# GAMEBOY PRINTER EMULATION PROJECT
# By Brian Khuu (2017)
!INIT: length: 0 | CRC: 1 | CRC CALC: 1 (0 1) | crc raw: 0 1 |Printer Status:  |
!DATA: length: 640 | CRC: 1 | CRC CALC: 366 (1 110) | crc raw: 0 1 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF F8 FF E0 F0 C1 E0 80 C3 80 83 00 83 80 03
FF FF 7F FF 5F 3F 8F 1F 4F BF 3F 1F 0A 14 44 30
FF FF FF FF FF FF FF FF FF FF 3E FF 3C 18 D0 09
FF FF FF FF FF FF EF F7 F3 E7 76 E3 10 26 B4 00
FF FF FF FF FF FF FF FF FF FF E3 FF E3 41 01 C8
FF FF FF FF FF FF BF FF 3E 9F 1C 9E 0C 0E 0E 9C
FF FF FF FF FF FF 07 FF 03 07 51 23 78 73 FF 7E
FF FF FF FF FF FF FF FF FF FF 8D FF 04 0C 60 04
FF FF FF FF FF FF FF FF FF FF 93 FF 01 01 48 01
FF FF FF FF FF FF FF FF FF FF C7 FF C6 83 22 91
FF FF FF FF FF FF FF FF FF FF 5C FF 00 08 3B 00
FF FF FF FF FF FF FF FF FF FF 7F FF 3F 7F 1F 3F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
81 03 00 83 80 83 81 C2 C0 E0 F0 F0 FF FF FF FF
C4 E1 0C 01 00 0C 0E 0C 1E 1F 7F 3F FF FF FF FF
CB C1 C9 C3 18 C1 0C 18 FD 3E FF FF FF FF FF FF
D1 A0 90 E0 36 80 12 26 AE 77 FF FF FF FF FF FF
D9 80 5E 81 DA 0D 60 01 EB 77 FF FF FF FF FF FF
1E 9C 1E 9C 0C 9E 0E 87 BF CF FF FF FF FF FF FF
F2 7F 79 72 50 22 03 06 3F CF FF FF FF FF FF FF
C2 04 60 04 00 64 00 04 CD 36 FF FF FF FF FF FF
0D C8 1D C8 1D C8 9C C9 EB DD FF FF FF FF FF FF
32 01 3C 03 34 9B C0 83 D3 EF FF FF FF FF FF FF
77 38 7B 30 70 33 70 30 BA 7D FF FF FF FF FF FF
3F 3F 3F 3F 1F 3F 3F 1F DF BF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 109 | CRC CALC: 28374 (110 214) | crc raw: 0 109 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FB F7 F1 E3 C8 E3 F2 E2 F6 FC F8 FC FD F9 FC F9
FF FF FF FF EF 1F 07 0F 0F 07 07 07 E7 F7 17 E7
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF F0 FF E0 F0 F0 E0 F0 E0
FF FF FF FF FF FF FF FF 07 FF 07 03 03 03 07 03
!DATA: length: 640 | CRC: 76 | CRC CALC: 19752 (77 40) | crc raw: 0 76 |Printer Status: Ready To Print, Checksum Error,  |
F8 FC FE FC FF FE E0 FF C0 E0 C0 E0 C0 E0 E1 FC
07 07 0F 07 1F 0F 47 BF 07 07 07 07 07 07 17 E7
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F0 E0 F0 E0 F0 E0 F1 E0 F0 E0 F0 E0 F0 E0 F0 E0
07 03 87 03 9B 83 E3 F3 07 03 87 43 DB 83 7B C3
F9 FC FC FC FE FC FE FE FF FF FC FE FC FC F8 FC
17 E7 07 07 0F 07 1F 0F 5F BF 07 07 07 07 07 07
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F0 E0 F0 E0 F0 E0 F3 E0 F0 E0 F0 E0 F0 E0 F0 E0
73 23 83 03 83 73 F3 63 73 03 D7 03 9B 83 53 83
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 88 | CRC CALC: 22528 (88 0) | crc raw: 0 88 |Printer Status: Ready To Print, Checksum Error,  |
F8 FC FD FC FE FC FC FC FC FC FE FD FE FF FE FC
07 FF FF FF 0F 07 07 07 07 07 FF 07 8F 7F 67 0F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0 F0 E0
53 23 67 03 13 E3 1B 83 93 63 07 03 F3 63 9B 83
FC FC FD F8 FD F9 F9 FC FC FC F8 FE FC FB F1 FB
67 07 77 27 67 37 57 27 07 07 07 0F EF 1F FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F0 E0 F0 E0 EF F0 FF FF F7 E7 E1 F3 F1 F8 F8 FC
73 83 8B 77 F7 0F FF FF FF FF FF FF FF FF 07 7F
!DATA: length: 640 | CRC: 84 | CRC CALC: 21566 (84 62) | crc raw: 0 84 |Printer Status: Ready To Print, Checksum Error,  |
E0 F0 E0 F0 E9 F0 F1 FB FF FA FE FC FC FC F8 FC
07 07 07 07 FF 07 0F FF FF 07 07 07 07 07 F7 0F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FA FC E9 F0 F1 E3 EE F0 F8 F0 F4 E3 F3 E7 F3 E7
07 03 3F C3 FF FF DF 3F 17 0F 43 87 F7 E3 E3 F3
FD FC FC FC FC FC FC FC FC FC FF FF D0 E4 C0 E4
FF FF 07 FF 07 07 07 07 07 07 FF FF 07 07 07 07
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
E5 F3 F9 F0 F8 FC FE FF F9 FF E0 F0 F4 E2 F4 E2
E3 F3 F7 03 0F 07 8F 7F 87 8F 23 07 77 23 63 73
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 83 | CRC CALC: 21536 (84 32) | crc raw: 0 83 |Printer Status: Ready To Print, Checksum Error,  |
C0 E4 E4 FF EC F3 C0 E0 C0 E0 D0 E0 FF FF FA FC
07 07 07 FF 17 EF 07 07 07 07 07 07 97 0F 1F 3F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F0 E0 EE F0 FF FF FF FF F7 FF E3 F7 F2 E7 F3 E6
73 03 03 03 97 E3 FF FF F7 FF E7 F3 A3 73 63 33
F0 F8 D1 E0 C0 E0 C0 E0 C0 E0 FF FF FF FF FF FF
7F FF F7 0F 07 07 07 07 07 07 FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F3 E6 F0 E0 EC F0 FE FF F0 FF F0 E0 E0 F0 F8 FC
63 33 03 33 03 03 1F E3 1B E7 03 03 C3 7F 5F 3F
!DATA: length: 640 | CRC: 85 | CRC CALC: 22064 (86 48) | crc raw: 0 85 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF F1 F8 E0 F0 F4 E2 F0 E6 E0 F0 F9 F0
FF FF FF FF 87 0F 07 07 37 67 37 67 27 07 87 0F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FE FC FF E8 F0 F0 E0 F4 F8 FC FF FF FF FF FF
1F 0F 83 07 37 0F BF 7F 6F 1F 87 03 C3 E3 F3 FF
FF FF FC FF F8 F0 E0 F0 F4 E3 F2 E7 F0 E0 F8 F0
FF FF 3F FF 17 0F 07 07 57 27 77 27 47 27 47 6F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F8 FF E0 F0 F0 E0 F4 F8 FD FE FF FF EB F7 F3 E7
C7 03 1B 07 3F 9F 9F 1F 1F 0F 47 83 F3 E3 D3 BF
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Ready To Print,  |
!DATA: length: 640 | CRC: 102 | CRC CALC: 26456 (103 88) | crc raw: 0 102 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FC FF F8 F0 E0 F0 F4 E3 F2 E7 F0 E0 F8 F0
FF FF 3F FF 17 0F 07 07 57 27 77 27 47 27 47 6F
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
F3 E7 F3 E7 E7 F3 E3 F3 F1 F9 FC F8 FE FC FE FF
83 87 93 83 E3 93 E3 F3 E7 F3 43 07 07 0F 5F BF
FF FF FF E0 F0 E0 E4 F3 FB F7 FF FF FF FF FF FF
FF FF FF 07 07 07 07 FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!DATA: length: 640 | CRC: 21 | CRC CALC: 5800 (22 168) | crc raw: 0 21 |Printer Status: Ready To Print, Checksum Error,  |
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FE C1 CC 80 DE 8C
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FB 47
FF FF FF FF FF FF FF FF FF FF FC FE FE FC FC 86
FF FF FF FF FF FF FF FF FF FF 7F FF FF 7F FB 4C
FF FF FF FF FF FF FF FF FF FF FF F3 E3 F3 B0 63
FF FF FF FF FF FF FF FF FF FF FD E0 C6 E0 C3 E6
FF FF FF FF FF FF FF FF FF FF 7E FF 3E 7F 5E 21
FF FF FF FF FF FF FF FF FF FF 3F 7F 7F 3F 37 68
FF FF FF FF FF FF FF FF FF FF FF CF C7 CF CB 86
FF FF FF FF FF FF FF FF FC FF F8 F0 F8 F0 FA 10
FF FF FF FF FF FF FF FF 03 FF 03 01 03 01 C3 01
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
C0 8C C0 80 C1 83 DF 8F DF 8F BF CF DF FF FF FF
43 81 31 99 B5 18 B0 19 1B 81 A7 C3 E7 FF FF FF
84 02 60 32 7C 3E 78 32 30 02 4C 86 CE FF FF FF
D8 08 3A 11 18 10 49 10 80 49 5A EC FE FF FF FF
00 21 A3 13 07 13 E7 33 A2 11 78 31 79 FF FF FF
C1 E6 C0 E0 C0 E1 C7 E7 C3 E7 FF E7 FF FF FF FF
61 00 6B 46 EF C6 EF C6 EF C6 E6 CF EF FF FF FF
20 20 22 24 2E 24 2E 24 2E 24 BD 66 7E FF FF FF
C2 04 C0 4C C0 4C C0 4C CC 40 C5 62 E7 FF FF FF
0A 13 12 C3 13 02 EB 12 9B 42 38 12 30 F8 F8 FF
43 81 03 01 03 01 03 01 03 01 03 01 01 03 03 FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF
!DATA: length: 0 | CRC: 4 | CRC CALC: 4 (0 4) | crc raw: 0 4 |Printer Status: Ready To Print,  |
!PRNT: 01 13 E4 40 | : length: 4 | CRC: 1 | CRC CALC: 380 (1 124) | crc raw: 0 1 |Printer Status: Print Reqested, Currently Printing, Checksum Error,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested, Currently Printing,  |
# Finished Pretending To Print for fun!
!INQY: length: 0 | CRC: 15 | CRC CALC: 15 (0 15) | crc raw: 0 15 |Printer Status: Print Reqested,  |
```

</details>
