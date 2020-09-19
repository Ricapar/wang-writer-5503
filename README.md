# Wang Writer 5503 Restoration Project
This is my adventure in trying to restore a word processor from 1981 to working condition. At this time, it is not currently working. It powers on, but it does not pass its power-on tests and does not currently get far enough to even attempt to load its system software from the floppy disks.

## Acquisition and a Mission
In early September 2020 I stumbled across a Facebook Marketplace post titled: "[Old computer setup processor monitor wang 40 dollars estate find](images/fb_marketplace.png)". I love old hardware, and already own a few other vintage systems, so I wasn't going to pass this one up. I've heard of the [Wang](https://en.wikipedia.org/wiki/Wang_Laboratories) company in the past, but I've never actually gotten my hands on one. (The "wang" jokes just write  themselves, don't they?)

I messaged the guy on Facebook Marketplace, and later that night I met him at his storage unit to pick it up. It was in serious need of a cleaning and smelled like it spent the last 30 years in a basement. But it came with _everything_. Cables, system disks, user and technical service manuals, etc.

What I couldn't find was any information about this on the internet. Searching Google for "Wang Writer 5503" returned nothing. (That search now returns my own Reddit and VCF posts)

Given the lack of information about this machine I'll be doing my best to document as much as I can about it as I can and hopefully get it working in the process. I'll be periodically updating the work log below as best I can. If I fall out of touch with this repo feel free to reach out to me on Twitter, Reddit, or where ever else you can find me.

## Contact Me, Please!
I can't seem to find any information about this machine on the internet beyond a single line-item in a PDF of a parts catalog.

If you've used this model of word processor, know anything about it, or have any suggestions on how I can better diagnose what may be wrong with it, I'd love to hear from you.

You can reach out to me via any of the accounts used in the posts under the _Links_ section below or via Twitter.

# Links
* [Photo Album (Google Photos)](https://photos.app.goo.gl/h5TvAR3F5gg16o3e6)
* [My post on /r/retrocomputing](https://www.reddit.com/r/retrocomputing/comments/imo90m/wang_writer_5503_is_anyone_familiar_with_this/)
* [My post on Vintage Computer Federation forums](http://www.vcfed.org/forum/showthread.php?76671-Wang-Writer-5503-In-search-of-information-details-history-etc)
* [@Ricapar on Twitter](https://twitter.com/Ricapar)

# Work Log

## 2020-09-19
Been spending some time the past few days going thorugh the service manual. I've got my bearings now, and can generally find where a component is located within the schematics without too much trouble.


### Power Supply Tests
I didn't note this in an earlier update, but I did go thorugh all of the power supply outputs and confirmed they are all up to spec. The service manual contained a nicely detailed table containing the expected voltages and tolerances for each pin going into the motherboard, and everything was up to spec.

```
+5V (4.75 -to- 5.25Vdc)          J1 Printer PCA pin 1, ground pin 3
+12V (11.40 -to- 12.60Vdc)       J1 Printer PCA pin 5, ground pin 3
-5V (-4.75 -to- -5.10Vdc)        J1 Printer PCA pin 7, ground pin 3
-12V (-11.76 -to- -12.24Vdc)     J1 Printer PCA pin 9, ground pin 3
+18V (+14.00 -to- 21.00Vdc)      J3 Printer PCA pin 1, ground pin 3
+36V (+28.00 -to- 42.00Vdc)      J3 Printer PCA pin 5, ground pin 3
```

### Chips' (testing) Challenge
The system bootup says that the error message is coming from CPU board, so most of my testing has been focused on there. Every removable chip has been removed and re-seated. Contacts have been cleaned as well. No changes there.

I'm slowly going through all of the chips on the CPU board, verifying at the very least that they're getting the expected voltage on the their +5V pins.


## 2020-09-15
Dumped the boot PROM from the middle motherboard. PROM chip is a Hitachi HN462716G; was able to get it dumped using XGPro with a TL866Plus EEPROM reader/writer. 

The PROM dump contained some strings that seem to indicate that there may have been errors in reading the ROM. It's a 2K ROM chip, but according to the manual the ROM is wired into the bus at memory addresses 0xC000 thorugh 0xC400, indicating that it may only be actually mapping the first 1K of data out of the chip. The error is presented a bit past the 1K mark, so I'm not entirely sure if I'm already dead in the water with a PROM that's lost some bits over the last 40 years.

```
00000720  5d 80 f1 e6 0f c6 90 27  ce 40 27 77 23 c9 4d 65  |]......'.@'w#.Me|
00000730  6d 6f 72 79 20 45 72 72  6f 72 20 61 74 20 20 20  |mory Error at   |
00000740  20 20 20 70 61 67 65 0d  45 78 70 65 63 74 65 64  |   page.Expected|
00000750  20 64 61 74 61 20 3d 0d  52 65 63 65 69 76 65 64  | data =.Received|
00000760  20 64 61 74 61 20 3d 0d  58 4f 52 20 64 61 74 61  | data =.XOR data|
```

Nevertheless, I was able to disassemble the ROM using [Mdz80](https://www.z80cpu.eu/78-data-articles/projects/76-mzt), a disassembler for the Z80 CPU.

```
mdz80 -a -f -v -xc000 rom_l59.bin
```
The `-xc000` flag was key and took me a few minutes to realize why I couldn't follow through the assembly that came out of the first dump I did. The Z80 CPU starts its program counter at memory address 0 but the ROM is wired into memory address 0xC000, so any addresses referenced within the disassembled code need to have 0xC000 added.

The binary and disassembled ROM can be found here:

* Binary: [roms/rom_l59.bin](roms/rom_l59.bin)
* Hexdump: [roms/rom_l59.hex](roms/rom_l59.hex)
* Disassembled: [roms/rom_l59.z80](roms/rom_l59.z80)

## 2020-09
Acquired! Cleaned and it's now taking over a table and half the floor in my TV/game room.

It powers on. No smoke or sparks. Doesn't boot up though.

According to the manual the 7-segment display on the back is used to indicate where the power-up diagnostic routine has identified faults.

The display reads a "7", indicating:

| Error Code | PCA      | FRU                                            |
|------------|----------|------------------------------------------------|
| 7          | CPU 7777 | CPU UART OR PIO, KEYBOARD CABLE, KEYBOARD UART |


# References

Datasheets and stuff. These are here mostly for my own reference.

## Datasheets
* [Z80A CPU](https://www.zilog.com/manage_directlink.php?filepath=docs/z80/um0080&extn=.pdf)
* [HN462716G EEPROM](https://pdf1.alldatasheet.com/datasheet-pdf/view/116083/HITACHI/HN462716G.html)
* [uPD416C-2 16K x 1-Bit RAM](https://www.datasheets360.com/pdf/1852544526355807954)
* [Intel P8272 Floppy Disk Controller](https://datasheetspdf.com/pdf-file/627488/IntelCorporation/8272/1)
