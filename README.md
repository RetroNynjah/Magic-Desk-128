# Magic Desk 128 (1MB)

<img src="rev1\images\render-top-names.png" alt="Render top" width="400"/><br/>
This is a Commodore 128 version of the popular Commodore 64 Magic Desk cartridge.
The Magic Desk cartridge is a simple bank switching ROM cartridge that uses IO1 ($de00) to select ROM banks. The difference between this C128 Magic Desk format and the C64 version is that the C128 version has 16kB ROM per bank instead of 8kB and therefore supports twice as much ROM. Theoretically it would support up to 4MB (256x16kB).  
This is the first version of the C128 Magic Desk cartridge and it has room for up to 1MB of ROM divided into 64 banks of 16kB.

The cartridge was designed with SST39 flash chips in mind but other 32-pin flash or EPROM/EEPROM chips can be used too as long as they are pin-compatible.

### Startup
After power-up or reset, ROM bank 0 (the first 16kB of the LOW chip) is active and accessible at $8000-$bfff.  
That first bank needs to contain the regular cartridge initiation bytes such as CBM magic word, autostart configuration and start vectors if you want your cartridge to autostart.

### Switching banks
Write an address of a ROM bank such as #00-#63 (#$00-#$3f in hex) to $de00 to switch bank. Set the value to 63 to switch to the last bank of the second ROM and value 0 to switch back to the first bank in first ROM or any bank in between.
The size and amount of ROM chips is flexible. It's even possible to mix and match chips of different sizes to match your needs but you need to be aware of the available bank numbers when doing so.
The LOW chip contains banks 0-31 and the HIGH chip contains 32-63. 
It doesn't make sense to add a HIGH chip if there's anything less than a 4Mbit/512kB (SST39SF040) chip in the LOW position but it is possible. You just have to skip some of the banks from the LOW range and skip to bank 32 to read from the HIGH chip.

### Precautions
ROM banks are switched whenever IO1 is asserted which is when any address between $de00 and $deff is accessed.
Writing to any of these addresses will switch the bank the same way as $de00 does.
Reading from any adress between $de00 and $deff corrupts the bank selection by switching bank to a somewhat random bank so if you for some reason would like to read from these addresses in your application you must make a proper bank selection afterwards.
I don't know of any program that reads from that area but if a program does that it can't be used on a multicart created with the Magic Cartridge Generator if you don't modify the program.

## ROM population examples
|LOW chip   |HIGH chip	|Banks      |Total size |
|----------	|----------	|-----	    |----------	|
|SST39SF010	|           |0-7        |128kB      |
|SST39SF010	|SST39SF010	|0-7, 32-39 |256kB		|
|SST39SF020	|           |0-15		|256kB	  	|
|SST39SF040	|           |0 -31      |512kB      |
|SST39SF040	|SST39SF020	|0-47		|768kB		|
|SST39SF040	|SST39SF040	|0-63		|1024kB	  	|

If you use a smaller chip such as the SST39SF010, the data in the eight available banks will be repated at bank address 8-15, 16-23, 24-31.

## BOM
|identifier |Part                 |Comments                         |
|----------	|----------	          |-----							|
|U1         |74LS273              |or 74HCT273/74AHCT273            |
|U1         |74LS02               |or 74HCT02/74AHCT02              |
|U3         |32-pin flash/EPROM   |First ROM. SST39SF or compatible |
|U4         |32-pin flash/EPROM   |Second ROM (optional)            |
|C1,C2,C3,C4|100nF ceramic        |5mm pitch                        |

## Software
### Volley for Two
The game Volley for Two is the reason that I developed this cartridge format in the first place. I started working on the cartrige when the programmer Jonas Hultén wanted a cartridge format to release the game on. He modified the game to load from a Magic Desk 128 cartridge.

You can download cartridge ROM file along with printable labels and more from https://kollektivet.nu/v42/  
The game needs a 256kB (SST39SF020) or larger flash chip.

### Magic Cartridge Generator
Zzarko has made a Python script and a menu program that can be used to generate menu-driven multi-cart images that can be put on this cartridge.  
Create a cartridge with all your favorite C128 utils or games. The requirement is that they consist of a single prg-file (no multi-load programs). 
Zzarko's Magic Cartridge Generator along with documentation and examples can be found at https://bitbucket.org/zzarko/magic-cartridge-generator

### Emulator Support
The Kernal64 emulator can emulate this cartridge.
At the moment you will have to mount the ROM image from the internal function ROM configuration and set the type to MAGICDESK128.  
The Kernal64 project can be found at https://github.com/abbruzze/kernal64

Vice will support emulation of the Magic Desk 128 from version 3.6.2  
Volley For two and images created by ZZarko's Magic Cartridge Generator works in r42534 and later builds. These builds can be found at https://github.com/VICE-Team/svn-mirror/releases  
The only method I currently know of for mounting the cartridge image is to start Vice with the command line parameter *-cartmd128 <romfile.bin>*  
