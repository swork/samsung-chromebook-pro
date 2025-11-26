---
---
Keyboard mapping in Linux is messy. I want Ctrl where God intended, under my left pinkie skooched left, and it took
some digging to find udev - keyboard translation to kernel events just above the hardware.

Here's the file that went into `/etc/udev/hwdb.d/61-keyboard-local.hwdb` to effect this change both on the internal
keyboard and an external Apple wireless keyboard that I had lying around:

```
# Make the Samsung "Search" key act as left CONTROL modifier
evdev:input:b0011v0001p0001*
 KEYBOARD_KEY_db=key_leftctrl

# Same for the Apple bluetooth keyboard's CapsLock key
evdev:input:b0005v05ACp022C*
 KEYBOARD_KEY_70039=key_leftctrl

```

`evtest` was key to getting both the device identifier/selector strings and the key mapping directives worked out.
There's baroque documentation at `/usr/lib/udev/hwdb.d/60-keyboard.hwdb` and `/usr/lib/udev/hwdb.d/60-evdev.hwdb`, but I didn't get things right
following that as best as I could. [This blog resource](https://deskthority.net/viewtopic.php?t=24076) served me better.

Relevant bits quoted below for posterity:

```
15 Jun 2020, 00:05
This guide focuses on remapping keys using udev, which is the device manager for the linux. The advantages of this is that the rules are keyboard / device specific, so you avoid having to write your own keymap file and using it for every keyboard. HerbalNekoTea and I in the DT discord had some issues getting it to work correctly, so I wanted to write a step by step guide on how to do it.

Requires:

    linux based OS with systemd

    terminal knowledge

    the evtest program

I got the information on how to do key remapping from the following sources:

    https://wiki.archlinux.org/index.php/Ma ... o_keycodes

    https://wiki.archlinux.org/index.php/Ke ... _scancodes

    https://www.freedesktop.org/software/sy ... /hwdb.html

Also the files /usr/lib/udev/hwdb.d/60-keyboard.hwdb and /usr/include/linux/input-event-codes.h have a lot of useful info in them and come with nearly every OS.
I will be using arch linux but the links above should apply if you are on a different system. Most of what I will be doing here is as root (the sudo command) you can drop down into the root shell using sudo su -

The first step is to get the device "hardware match" string. The arch wiki had a really confusing way of doing this, so I came up with a more straight forward method. When I run the evtest command from the terminal, I get the following:

Code: Select all

# evtest
No device specified, trying to scan all of /dev/input/event*
Available devices:
/dev/input/event0:	Power Button
/dev/input/event1:	Power Button
/dev/input/event2:	SteelSeries SteelSeries Rival 110 Gaming Mouse
/dev/input/event3:	winkeyless.kr ps2avrGB
/dev/input/event4:	winkeyless.kr ps2avrGB System Control
/dev/input/event5:	winkeyless.kr ps2avrGB Consumer Control
/dev/input/event6:	PC Speaker
/dev/input/event7:	HDA NVidia HDMI/DP,pcm=3
/dev/input/event8:	HDA NVidia HDMI/DP,pcm=7
/dev/input/event9:	HDA NVidia HDMI/DP,pcm=8
/dev/input/event10:	HDA NVidia HDMI/DP,pcm=9
/dev/input/event11:	HDA NVidia HDMI/DP,pcm=10
/dev/input/event12:	HDA NVidia HDMI/DP,pcm=11
/dev/input/event13:	HDA NVidia HDMI/DP,pcm=12
/dev/input/event14:	HD-Audio Generic Front Mic
/dev/input/event15:	HD-Audio Generic Rear Mic
/dev/input/event16:	HD-Audio Generic Line
/dev/input/event17:	HD-Audio Generic Line Out
/dev/input/event18:	HD-Audio Generic Front Headphone
/dev/input/event19:	SINO WEALTH Gaming KB
/dev/input/event20:	SINO WEALTH Gaming KB  System Control
/dev/input/event21:	SINO WEALTH Gaming KB  Consumer Control
/dev/input/event22:	SINO WEALTH Gaming KB  Keyboard
Select the device event number [0-22]:

In this case I want to reassign some keys on my doomhammer (named "SINO WEALTH Gaming KB" in that list) so in this case I would then type 19 to start monitoring the events coming from that device. If you are unsure, select what you think is your keyboard, then start typing and see if the terminal outputting the keys you press. If you are getting no output its the wrong device.
Once you figure out what device you want to remap, you take the number that you used to monitor it (in my case 19) and type the following command to get the hardware match string for you keyboard

Code: Select all

cat /sys/class/input/event19/device/modalias

Take note on how the /sys/class/input/event19 matches to /dev/input/event19. The event19 in /dev/input is the actual device, the /sys/class/input/event19 is the folder for that device with the information you need. When I run the above command I get this:

Code: Select all

# cat /sys/class/input/event19/device/modalias
input:b0003v258Ap002Ae0111-e0,1,4,11,14,k71,72,73,74,75,79,7A,7B,7C,7D,7E,7F,80,81,82,83,84,85,86,87,88,89,8A,8C,8E,96,98,9E,9F,A1,A3,A4,A5,A6,AD,B0,B1,B2,B3,B4,B7,B8,B9,BA,BB,BC,BD,BE,BF,C0,C1,C2,F0,ram4,l0,1,2,3,4,sfw

Copy that output, then create a new file under /etc/udev/hwdb.d/ named 70-keyboard.hwdb and paste that output at the top of the file. The naming here is really important, because it did not work for me until I changed it to that filename. The hardware match here is pretty long, and can be longer, which isn't necessary to get your keys remapped. It can cause problems if its too specific, so I shortened mine to the following:

Code: Select all

input:b0003v258Ap002A*

The asterisk means that it will match any character after that part of the string, so essentially we are omitting that really long part at the end when we only need the values b0003, v258A, and p002A. To finish this part off we add evdev to the beginning of our string like so:

Code: Select all

evdev:input:b0003v258Ap002A*

Finally. we can now remap the keys. There's plenty of info up in the links I posted if you need the details but I am just going to post my configuration and a few rules of thumb when editing this file. udev is extremely picky, any syntax errors or anything a little off, it will refuse to set the rules you write in and just ignore them.

    indent the directives using a single space

    key reassignment syntax is KEYBOARD_KEY_<scancode>=<keycode> scancode comes from evtest, keycode is the name of a keycode as lowercase from the /usr/include/linux/input-event-codes.h file.

    /etc/udev/hwdb.d/70-keyboard.hwdb file ownership should be under user and group root and with -rw-r--r-- permissions. Double check using the ls -l command.

    removing a remap from the file will not remove the mapping for the key. You need to change the keycode back to what it was originally, then update, then you can remove the directive.

My file:

Code: Select all

evdev:input:b0003v258Ap002A*
#capslock -> left control
 KEYBOARD_KEY_70039=key_leftctrl
#pause -> numlock
 KEYBOARD_KEY_70048=key_numlock
#A -> B
 KEYBOARD_KEY_70004=key_b

The lines beginning with # are ignored by udev. I got the scancodes by running evtest and, for example, hitting the A key gives me the following output:

Code: Select all

Event: time 1592170729.749901, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70004
Event: time 1592170729.749901, type 1 (EV_KEY), code 30 (KEY_A), value 1
Event: time 1592170729.749901, -------------- SYN_REPORT ------------
aEvent: time 1592170729.852908, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70004
Event: time 1592170729.852908, type 1 (EV_KEY), code 30 (KEY_A), value 0
Event: time 1592170729.852908, -------------- SYN_REPORT ------------

As you can see, my scan code in this case is the part that says value 70004 after the code 4 (MSC_SCAN) part. You do this for every key that you want to remap. In my file I have already remapped the caps lock and pause keys and as you can see the scancode 70004 shows up in my file where I remap the A key to the B key. I recommend trying to remap one key first, see if it works, and then go from there so you don't write a big file of remapped keys and have it not work.
Once you have finished editing your file run the following command:

Code: Select all

systemd-hwdb update ; udevadm trigger --verbose --sysname-match='event19'

you can leave out the --verbose --sysname-match='event*' but that just confirms with an output which events were actually updated. To 100% confirm your remapping rules were actually added, run the following:

Code: Select all

 udevadm info /dev/input/by-path/*-usb-*-kbd | grep -i keyboard_key

From that output I get the following:

Code: Select all

# udevadm info /dev/input/by-path/*-usb-*-kbd | grep -i keyboard_key
E: KEYBOARD_KEY_70004=key_b
E: KEYBOARD_KEY_70039=key_leftctrl
E: KEYBOARD_KEY_70048=key_numlock

If I run switch hitter and press the capslock and pause key I get the left control and num lock keys respectively.
```
