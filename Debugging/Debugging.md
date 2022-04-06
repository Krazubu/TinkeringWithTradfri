# <a>Using the debug interface over telnet and gdb</a>
Now that we know which regions contain data we can achieve various operations either using Telnet or GDB. GDB will allow to push things further for experienced users but we'll only cover basic operations here.
## <a>Using telnet</a>
Now that  OpenOCD is running as a server, we can't use that terminal window anymore, so leave it running and open a new one.
Initiate a telnet connection by typing:

```telnet localhost 4444```
If all is well you'll be in the telnet invite ```>``` and the server window will tell it is accepting a telnet connection.
We can check that the chip is connected correctly and is running by issuing the command ```targets``` :

```
    TargetName         Type       Endian TapName            State       
--  ------------------ ---------- ------ ------------------ ------------
 0* efm32s2.cpu        cortex_m   little efm32s2.cpu        running
```
efm32s2 tells us it is using our special driver for EFR32 series 2 and running state means the software inside is currently running fine.


There are several possible states :

* running : this is normal CPU operating mode
* halted : CPU is halted by the debugger
* external reset : reset was triggered by a software error or the reset pin
* handler hard fault : CPU does not boot, bootloader is corrupt

### Reading data from the MCU
Use ```flash banks``` to get a map of memory organization:

```
> flash banks
#0 : efm32s2.flash (efm32s2) at 0x00000000, size 0x00000000, buswidth 0, chipwidth 0
#1 : userdata.flash (efm32s2) at 0x0fe00000, size 0x00000000, buswidth 0, chipwidth 0
#2 : lockbits.flash (efm32s2) at 0x0fe04000, size 0x00000000, buswidth 0, chipwidth 0
```
Before going further we halt processor just by typing ```halt```:

```
> halt
target halted due to debug-request, current mode: Handler External Interrupt(50)
xPSR: 0x09000042 pc: 0x00004622 msp: 0x20014ea8
```
To get more infos about a bank use ```flash info``` followed by the bank number :

```
> flash info 1
#1 : efm32s2 at 0x0fe00000, size 0x00000800, buswidth 0, chipwidth 0
	#  0: 0x00000000 (0x800 2kB) protected
MG21A020, rev 30
```
We learn here that this bank size is 2Kb large, and that it's write protected. We'll come back to this last point later.

It is possible to quickly explore and display a part of the memory using the ```mdb``` command, followed by the adress and the size of the part we want to show.
For instance :  
```mdh 0x4000 200```  
This will display the first 200 bytes of the beginning of application region.

Before messing too much with the chip, one of the 1st things we'll want to do is to backup the current firmware in case we break things.  
We can do so with the command:  
```dump_image somefile start_address size_to_dump```  

Relying on the table showing the various data areas, the following command will allow us to save each segment in its own file:

```
dump_image FLASH.bin 0x0 0xfffff				;#backup the whole flash data in a single 1MB file
dump_image USERDATA.bin 0x0FF00000 0x400
dump_image DEVINFO.bin 0x0FE08000 0x400
dump_image CHIPCONFIG.bin 0x0FF000000 0x400
```
The FLASH.bin file will already contain a complete image of the main memory region, however for convenience we can also re-save some of its subregions separately:

```
dump_image bootloader.bin 0x0 0x4000 	  ;#save only the bootloader
dump_image application.bin 0x4000 0x40000 ;#save only the application, size may vary.
dump_image bootloader2.bin 0xFE270 0x120  ;#save that unknown bit of data that is mandatory for boot
```
Doing so will allow us to only restore useful parts of the region like the bootloader instead of writing the whole region.

### Writing data to MCU
To flash a region, we can use the ```program``` command. For instance, to flash a new application, we'll have to take care to not overwrite the bootloader, in this case we'll want to start flashing at adress 0x4000 using the following command:  
```program application.bin 0x4000```
When flash is done, you can reset the MCU by issuing the ``` reset``` command.
You can then ensure that the MCU is operating well by using ``` targets``` command.


## <a>Using GDB</a>
We can also connect with GDB, the name of the gdb executable may vary depending on the environment :  
```gdb-multiarch``` or ```arm-none-eabi-gdb```  
If none of them is present, you'll have to install it manually with your usual package manager.
You'll be welcomed by an invite :  
```
(gdb)
```

Now connect to OpenOCD GDB server :  
```target extended-remote localhost:3333```  
At any time, you can still send commands directly to OpenOCD like with telnet by using prefix ```mon```. For instance:
```mon targets```

### Reading data from the MCU
The ```dump``` command will save a region to a file. Note that unlike with telnet, you must give the ending address of the area you wish to dump and not its size.  
```dump memory somename.bin 0x00000000 0x000FFFFF```  


### Writing data to the MCU

It won't be possible to write a bin file directly using GDB, you'll need to convert it first to elf format. This format is more advanced and can contain metadatas.
The tool ```arm-none-eabi-objcopy``` will allow you to achieve this conversion with the following syntax. This tool should have been installed with ARM version of binutils:
```arm-none-eabi-objcopy --input-target=binary --output-target elf32-little inputfile.bin outputfile.elf```


Once done, write your newly converted file:  
```(gdb) load FLASH.elf 0x0```

The OpenOCD server window should immediately show something like this:

```
target halted due to debug-request, current mode: Thread
xPSR: 0x89000000 pc: 0x00012f88 msp: 0x200145b8
```

then after a while the GDB windows will show:

```
Loading section .data, size 0x100000 lma 0x0
```
just wait, it can be long as writing speed was 2Kb/s using arduino micro.
When it finally ends it should write:
```
Start address 0x00000000, load size 1048576
Transfer rate: 2 KB/sec, 16131 bytes/write.
```

Once done, reset MCU with ```mon reset run```  and check MCU state with ```mon targets```  
