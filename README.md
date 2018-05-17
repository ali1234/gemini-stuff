# Gemini PDA Notes

* Official Flashing Guide: http://support.planetcom.co.uk/index.php/Flashing_Guide
* Flashing tool for Linux: http://support.planetcom.co.uk/download/FlashToolLinux.tgz
* Firmware downloads: http://support.planetcom.co.uk/index.php/Gemini_Firmware
* Linux scatter generator: http://support.planetcom.co.uk/download/partitionTool.html

## Bootloader

Bootloader is controlled by the Escape key and the voice assistant key (the one and only hardware button.)

The key combinations required differ depending on firmware version. An up to date list is here: https://github.com/gemian/gemini-keyboard-apps/wiki/Bootloader

To correctly enter the keyboard combinations, start with the device off. If the combination requires you to hold the voice button, press and hold that first. Then press the escape key until the device vibrates. Then release the escape key, if the combination requires that.

So for example, to boot Linux, which currently requires you to hold only the voice button, do this:

1. Power off device.
2. Press and hold the voice button.
3. Press and hold the escape key until you feel the vibration, then immediately release it.
4. Continue to hold the voice button until you see the penguin.

## Recovery

There is a keyboard combination to access recovery, but it does not seem to be functional in the factory firmware. The device enumerates but is not recognized by adb or fastboot. There is no menu on screen, and no response to any user input. To escape from this mode, just unplug the USB cable and wait about a minute. The device will reset automatically.

After flashing the newer firmware the recovery image seems to work.

## Flashing

In order to successfully flash, you need to have read write access to the serial ports on your system, and you need to blacklist the Gemini's BROM serial port so that Modem Manager does not interrupt the flashing connection. If you do not do the latter the Gemini will be stuck in DA mode. See below for how to recover from that.

To configure the permissions, first add your user to the dialout group:

    sudo gpasswd -a <username> dialout

If you don't want to log out and log in again, use newgrp to log in to the new group for the current shell:

    newgrp dialout

Then copy the udev rules from this repository onto your system:

    sudo cp etc/udev/rules.d/* /etc/udev/rules.d/

Now reload the udev configuration:

    sudo udevadm control --reload

Start the flashing tool using the shell script:

    ./flash_tool.sh

Flashing can only be done from device power off state, and only for approximately 0.5 seconds after you plug in the cable. So you have to start the flashing tool before plugging in the Gemini.

If you forgot to blacklist Modem Manager the following will happen:

1. The flashing tool will upload the second stage bootloader (called Download Agent in the app.)
2. The device will re-enumerate on USB.
3. Modem manager will see a new serial port and attempt to probe it for modems.
4. The Download Agent will be confused by the AT commands and will crash, but the device will not reset.

You will see the following error on the flash tool console output:

```
USB port is obtained. path name(/dev/ttyACM0), port name(/dev/ttyACM0)
USB port detected: /dev/ttyACM0
BROM connected
Downloading & Connecting to DA...
connect DA end stage: 2, enable DRAM in 1st DA: 0
Failed to Connect DA: STATUS_PROTOCOL_ERR
Disconnect!
BROM Exception! ( ERROR : STATUS_PROTOCOL_ERR (-1073676283) , MSP ERROE CODE : 0x00. 
```

The device will not enumerate on USB and will just keep on running the Download Agent firmware until the battery goes flat. To escape from this you have to hold down both the escape key and the voice button for about 30 seconds, and then it will reboot.

If you managed to get Linux installed, hold the appropriate button combination, and you should see the Planet logo with a Penguin in the bottom right corner. It is quite big, you will not miss it. After this the screen will go black for a very long time. Wait, and eventually you will get to the Linux login screen.

## Linux

The current Debian image is extremely broken:

* There is no GPU driver and the screen redraw rate is excruciatingly slow - about 2 frames per second.
* Attempting to change the keyboard layout or session at the login screen will crash the device. Hold escape + voice button until it restarts.
* The rounded corners of the display hide parts of the panel.
* After you log in (the pasword is gemini) you will get another black screen which lasts for about a minute. Then a dialog will ask you which window manager you want. You get lx panel regardless of what you choose, so this seems a bit pointless.
* If you open QTerminal you will get a window where you can type things but nothing happens. UXTerm just gives a locale error. Xterm gives a working shell.
* The function key does not work, so you can't type any special characters like '-' and '/' which makes the shell completely useless anyway.

To possibly fix some of these problems you need to follow the instructions at: https://github.com/gemian/gemini-keyboard-apps/wiki/DebianTP

Note that you can only do this over ssh as you need to type at least one command containing '-'.

To avoid having to figure out what the device IP address is, you can just "ssh localhost.lan" (since the default firmware currently doesn't even have a hostname, it uses localhost.

