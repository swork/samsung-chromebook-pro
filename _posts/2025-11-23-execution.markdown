---
title: Execution
---

Into developer mode: from power off, hold Esc and Refresh (fourth from left on top row) then tap Power and wait. That's Recovery Mode. From there it's Ctrl-D to get to the Developer Mode selection page, Enter to confirm, then a long wait while the machine gets wiped and reinstalled. I missed the counter at the top left but it's at 2 minutes to go as I write this last bit. It did not lie.

Then, on first boot, Ctrl-D selects booting into developer mode. Apparently waiting 30s does the same thing. The only obvious cue on the screen is to hit SPACE to return to normal-user mode (again via a wipe and reinstall). We of course hit Ctrl-D.

Ctrl-Alt-F2 (third key from left on top row, a right-arrow) pops from the very first screen to a "VT2 shell" at login. User "chronos" gets us in without a password. Verbiage at mrchromebox indicates there's no other way to have sudo actually get you root - there's a note that the "second way" to get a root-capable shell (via crosh, Ctrl-Alt-T) lost sudo capability at ChromeOS R117 and IDK if this machine is past that watershed or not.

## Hardware write protect

... involves removing a screw. Cool, they give a pic of the board so you know which screw - but here the screw is run into a heatsink from the other side! This requires surgery, and the battery is powered up. So in we go.

Battery out, a half-dozen cables disconnected (some with a cryptic ZIF latch), board unscrewed, screw removed. (Spoiler: I'm not paying enough attention.) Reassemble, run Mr. Chromebox's firmware-util.sh, see "WP Enabled" - nothing can be done. After a few more tries (isolating the screw hole from the heatsink with tape, removing the heatsink altogether) I wrote to Mr. Chromebox himself and got back a nice note indicating Yes, WP is just the one screw, there's a pull-up resistor on the flash chip so no worries about capacitance, and that he's seen solder blobs keeping it grounded.

Looking closer, I see the same mesh washer over the screw hole that I saw in the online picture. I assumed it is meant to be there after removing the screw, but in fact it serves to ensure one of the four quadrant solder blobs around the hole (which is soldered to the screw's through-hole via) is electrically connected to the other three - all of which go (through a SMD component, presumably a resistor) to the flash chip or its controller. Pulling the mesh washer off opened the ground connection and solved the problem: WP is no longer enabled at the hardware level.

## Completing the firmware replacement process

The rest of the full UEFI firmware install process was straightforward. Booted past the Dev Mode scary screen with Ctrl-D to the login screen, then Ctrl-Alt-(third top-row key from the left) gets to a shell, login without password as "chronos", run MrChromebox's util with "curl -LOfk https://mrchromebox.tech/firmware-util.sh && sudo bash ./firmware-util.sh". And follow instructions. I made a backup of the old firmware, then immediately and unintentionally destroyed it - but this was a one-way trip all along. (Glad I didn't have trouble half-way through; wish I still had the backup in case something comes up in the future.)

## OS install

Initial install was Ubuntu 25.10. More on that later.
