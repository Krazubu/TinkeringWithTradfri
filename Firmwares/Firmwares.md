
#<a>OTA updates</a>

##Downloading
You can find the links to grab the latest firmware updates by editing the json file located at [http://fw.ota.homesmart.ikea.net/feed/version_info.json][OTAjson].
[OTAjson]:http://fw.ota.homesmart.ikea.net/feed/version_info.json

This [database][fwdatabase] from zigpy's github might show useful to identify the files.
[fwdatabase]:https://github.com/zigpy/zigpy/discussions/660
Also, this huge [devices database][z2mdb] maintened by zigbee2mqtt will be precious to obtain useful infos about devices like names, model references and exposed features.
[z2mdb]:https://www.zigbee2mqtt.io/supported-devices/#v=IKEA

##Extracting image from OTA update
The downloaded firmwares won't be readily flashable as is from telnet or gdb. Instead, they'll come as files named with a ".signed" extension.
They only contain some added metadata and a signature to ensure file integrity, but there is no kind of compression or encryption involved.
It is then relatively easy to transform those files into flashable binaries using an hex editor:

* Find the "EBL" signature, at the beginning of the file, it should be followed by a string for the device model.  
* From there, move 58 bytes later (+0x3A to the current offset). This point is the beginning of the application image. You can verify that it's correct by researching the next 10 bytes. You should find the same chain about 0x201 or 0x209 bytes later.
* Remove all the data before the 1st occurence of the 10 bytes.
* Remove the last 256 bytes at the tail of the file.
* Flash this application at 0x4000

This is only an (awkward) empirical method, it's not based on parsing of the data contained in the various headers, there's probably a lot of improvement that can be done in this regard. Documentation for that seems to be available (btl_gbl_format.h), this is WIP and will be updated.

Most updates follow this scheme but some escape to it, this is for instance the case of the gateway firmware update which doesn't hold an EBL signature but seems instead to contain a readily flashable elf file. Some other files contain a GBL image.
I couldn't have all of them working so far but actually didn't try much things.
See below for a brief review of the available file formats commonly used here.


##File types
* s37 file is an ASCII file in a standard Motorola S-Record format. It can contain several images with metadatas like the location of the regions where to flash them. 
* EBL file is "ember bootloader". Legacy format. Designed for OTA updates.
* GBL file is "gecko bootloader". Current format. Designed for OTA updates. See [application note UG266 for details][UG266].
* ELF file is standard Executable and Linkable Format. Ready to be flashed in device. It can be parsed with readelf tool (in binutils or arm-none-eabi package)

[UG266]:https://www.silabs.com/documents/public/user-guides/ug266-gecko-bootloader-user-guide.pdf



##About the USERDATA region
This region is located at 0x0FE00000 and is used to store custom data that can't be erased or modified in a conventional way.
On TRADFRI devices, it's used to store the device model and it seems to be used to identify the device over the Zigbee neetwork. An attempt to wipe this region resulted in zigbee2mqtt finding an "unsupported device".

Modification of this region can only be done "from the inside" by uploading some code designed for this purpose.
A sample code to change the content of this region is available page 38 of [Application Note AN1303][AN1303].
[AN1303]:https://www.silabs.com/documents/public/application-notes/an1303-efr32-dci-swd-programming.pdf

Based on this sample, find below two compiled images to restore or erase this version :

* One to erase content of USERDATA : [erase_userdata.bin][erase_userdata.bin]
[erase_userdata.bin]:erase_userdata.bin
* One to restore factory content of USERDATA for the ligbulb LED2002G5 : [2002G5_userdata.bin][2002G5_userdata.bin]
[2002G5_userdata.bin]:2002G5_userdata.bin

As usual, this must be flashed at 0x4000.

It should also be possible to unlock and erase this region by issuing ```Erase Device Command``` (see page 15 of AN1303) 
through ```dap apreg``` commands in OpenOCD.