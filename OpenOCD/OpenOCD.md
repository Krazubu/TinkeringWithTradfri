# <a>Installing and configuring OpenOCD</a>
### Install and patch OpenOCD
Now it's time to install [OpenOCD](https://openocd.org) and update it for proper EFR32 series 2 support.[^1]

The steps below will allow you to download the official OpenOCD then patch it for better support of EFR32 series 2.  

```
git clone https://github.com/ntfreak/openocd
cd openocd
git checkout 42a0bf3
git submodule init  
git submodule update
git clone https://github.com/knieriem/openocd-efm32-series2
cp openocd-efm32-series2/efm32s2/efm32s2.c src/flash/nor/
cp openocd-efm32-series2/efm32s2/efm32s2.cfg tcl/target
patch -p1 < openocd-efm32-series2/efm32s2/adjust_openocd.patch
```
This will add a new "efm32s2" driver, while the old driver will remain as "efm32".
We now have an improved source code with better support for the MCU.


---
###### Optionnal

To get a better description of memory areas, there's yet another patch that we can apply, ie lockbits and userdata regions. Thanks to users miceuz1 and Uncannier, a patch was posted in Silabs' forum. See the thread [here](https://community.silabs.com/s/question/0D51M00007xeK8ySAE/efm32-developmentdebug-on-raspberry-pi). Based upon all the hints in this thread, you can grab a diff file adapted to our new efm32s2 driver [here](patch_efm32s2.diff). Back in your OpenOCD root directory, copy the diff there and apply it with command
patch -p1 < patch_efm32s2.diff

Alas, this last patch is rather cosmetic and still won't allow us to modify lockbits and userdata regions from OpenOCD, but at least a correct memory description will appear when we'll use flash banks command

---

You can now finally run the usual process to compile and install OpenOCD :

```
./bootstrap
./configure
make
sudo make install
```
---

###### Notes for macOS:
If you're building on macOS you might encounter several issues.

* error on `ld: symbol(s) not found for architecture x86_64`
If you check upper in the process you'll see the core reason is  
`ld: warning: ignoring file libjim.a, file was built for archive which is not the architecture being linked (x86_64): libjim.a`
This is because 3rd party ranlib and ar tools took priority over the stock ones.  
To overcome the issue, remove "binutils" pkg or reorder the paths so the original ones come 1st.

* error about `raggedright` in path `./doc/openocd.texi`
install texinfo with brew then put it into path with  
`export PATH=/usr/local/opt/texinfo/bin:$PATH`

---

If you've reached this point, congrats, OpenOCD is now properly installed

### Configure your interface and target MCU
Once OpenOCD is installed, you'll have to configure 2 things :

* The hardware interface you're going to use
* The target MCU you want to communicate with

OpenOCD already comes with many readymade configuration files that should fit most use cases.
You can invoke OpenOCD from a command line by giving it at least 2 parameters, the interface and the target.
For instance, to launch openocd using an arduino micro, connected to our EFR32MG210 chip, we'll write :  
```
sudo openocd -f interface/CMSIS-DAP.cfg -f target/efm32s2.cfg
```
Here we don't specify wether we want to use SWD or JTAG, OpenOCD will then default on the default protocol defined in the interface cfg file.
If we want to force SWD, we can specify it this way :
```
sudo openocd -f interface/cmsis-dap.cfg "-c transport select SWD" -f target/efm32s2.cfg
```

Custom settings can be modified in interface and target cfg files.
For instance you can change the pinout in the target cfg, or you can define custom memory banks in the target cfg (be careful with this one).
It is also possible to use a configuration file that will be automatically used when invoking openocd with no parameter. This file has to be named openocd.cfg and to resider in the current directory. See OpenOCD documentation and guides linked above for details.




When all is set and working fine, when you launch OpenOCD, it should read something like this :

##### Example using Arduino micro :

```
Open On-Chip Debugger 0.11.0+dev-00225-g42a0bf3c3-dirty (2021-12-26-14:20)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "swd". To override use 'transport select <transport>'.
cortex_m reset_config sysresetreq

Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : CMSIS-DAP: SWD  Supported
Info : CMSIS-DAP: JTAG Supported
Info : CMSIS-DAP: FW Version = 1.0
Info : CMSIS-DAP: Serial# = 1234
Info : CMSIS-DAP: Interface Initialised (SWD)
Info : SWCLK/TCK = 0 SWDIO/TMS = 1 TDI = 1 TDO = 1 nTRST = 0 nRESET = 1
Info : CMSIS-DAP: Interface ready
Info : clock speed 1000 kHz
Info : SWD DPIDR 0x6ba02477
Info : efm32s2.cpu: hardware has 8 breakpoints, 4 watchpoints
Info : starting gdb server for efm32s2.cpu on 3333
Info : Listening on port 3333 for gdb connections
```
It should now sit there waiting for a client to connect.
Notice the line that reads  
`Info : SWD DPIDR 0x6ba02477`
this one is important, 0x6ba02477 is the address of debug port reported by the MCU.
For EFR32MGM210, this numer MUST be 0x6ba02477.  
If you get something else like 0xffffffff or a plain error, check the wiring and your settings.

Also note the two lines  
` Info : Listening on port 4444 for telnet connections`  
and  
`Info : Listening on port 3333 for gdb connections`

This means that OpenOCD is now waiting for a telnet or gdb client, this is the next step we're going to focus on

[^1]: As of 26/12/21, the last release of OpenOCD doesn't support series 2 MCUs, but commit 42a0bf3 by [knieriem](https://github.com/knieriem/openocd-efm32-series2) allows it to work properly with our EFR32MG21
