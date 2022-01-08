# <a>How to use the EFR32 MG210 chip built in Ikea Trådfri Devices</a>
This github is an attempt to give a basis to start tinkering with the MGM210L Zigbee module found in some IKEA Tradfri accessories.
Those devices contain a fairly powerful MCU with embedded Zigbee/WiFi/Bluetooth 5 support that can be used as a basis for many DIY projects.

The subject has already been partly covered by several people like [basilfx](https://github.com/basilfx/TRADFRI-Hacking), [zw](https://github.com/zw/TRADFRI-Hacking) or also [MattWestb](https://github.com/MattWestb/IKEA-TRADFRI-ICC-A-1-Module), however they mainly focus on ICC-1 and ICC-A-1 versions and don't give much details about the MGM210 one.

We'll focus here on this one as it is slightly different in many regards (different memory regions for different purposes) and following instructions intended for those different chips will actually lead you to corrupt data in your chip.
We'll also try to give detailed step by step guides during all the process so that it's also accessible to people who are not familiar with the tools and devices involved.
This github is a WIP so bear with me if you find mistakes, and don't hesitate to correct me, require clarifications or give suggestions.


---

For the sake of clarity this github is divided in 5 subsections :

# [Teardown][Teardown]
[Teardown]:Teardown/Teardown.md</b>
Guide to dismantle a light bulb **non-destructively** and remove the MGM210 chip


# [Hardware considerations][Hardware]
[Hardware]:Hardware/Hardware.md</b>
Generalities about the chip and overview of available physical interfaces to be able to interact with the chip


# [Installing OpenOCD][OpenOCD]
[OpenOCD]:OpenOCD/OpenOCD.md
Guide to install the software that will allow you to communicate with the chip


# [Debugging][Debugging]
[Debugging]:Debugging/Debugging.md
Explanation of the basic operations you can do with the chip using telnet and GDB


# [Firmware][Firmware]
[Firmware]:Firmwares/Firmware.md
Informations related to firmware modding, extracting usable firmware images from OTA updates and various firmware related tools










---
## Other projects

**Doom on a lightbulb**  
You can see this amazing project to run [Doom game on the MGM210](https://github.com/marciopocebon/MG21DOOM)  
Here you'll find a github with all the resources : [lets-port-doom-to-an-ikea-tradfri-lamp](https://web.archive.org/web/20210614073810/https://next-hack.com/index.php/2021/06/12/lets-port-doom-to-an-ikea-tradfri-lamp/)
