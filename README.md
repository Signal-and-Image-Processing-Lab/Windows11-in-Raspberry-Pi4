# Windows11-in-Raspberry-Pi4
Installation Process of Windows11 in Raspberry Pi4
1. Prepare WoR
--------------

Download the latest version of WoR from https://worproject.com/downloads

Run WoR as Administrator.

Enable advanced mode so the Raspberry Pi model can be selected explicitly:

[WoR Configuration File]
WizardBasicMode = 0
SaveConfigOnExit = 0

In WoR, select:

Device: Raspberry Pi 4 / 400 [ARM64]
Partition scheme: GPT
Image: Windows ARM64 ESD/WIM/ISO
Drivers: Raspberry Pi 4 ARM64 drivers
UEFI: Raspberry Pi 4 UEFI firmware

2. Let WoR Create The USB
-------------------------

WoR should create a layout like this:

BOOT partition: FAT32, small EFI/boot partition
MSR partition: 16 MB
Windows partition: NTFS, main partition

Example placeholders:

<BootLetter>:     FAT32 BOOT partition
<WindowsLetter>:  NTFS Windows partition

WoR should then:

Apply the Windows ARM64 image to <WindowsLetter>:
Install Raspberry Pi 4 drivers into the offline Windows image
Copy Raspberry Pi 4 UEFI boot files to <BootLetter>:
Create EFI boot files
Create or update the BCD store

3. If WoR Fails At BCD Creation
-------------------------------

If Windows image and drivers finished, but WoR fails while creating boot files or BCD,
repair it manually from an Administrator PowerShell:

bcdboot.exe <WindowsLetter>:\Windows /s <BootLetter>: /f UEFI /offline /c

Then apply Raspberry Pi-friendly BCD settings:

bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} device partition=<WindowsLetter>:
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} osdevice partition=<WindowsLetter>:
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} path \Windows\System32\winload.efi
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} systemroot \Windows
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} detecthal Yes
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} testsigning on
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} nointegritychecks on
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} recoveryenabled No
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {default} bootstatuspolicy IgnoreAllFailures
bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /set {bootmgr} timeout 10

Verify the BCD entry:

bcdedit.exe /store <BootLetter>:\EFI\Microsoft\Boot\BCD /enum {default}

4. Required Boot Files
----------------------

The BOOT partition should contain at least:

<BootLetter>:\config.txt
<BootLetter>:\RPI_EFI.fd
<BootLetter>:\EFI\Microsoft\Boot\BCD
<BootLetter>:\EFI\Microsoft\Boot\bootmgfw.efi
<BootLetter>:\EFI\BOOT\BOOTAA64.EFI

If fallback boot is missing, run:

New-Item -ItemType Directory -Force <BootLetter>:\EFI\BOOT
Copy-Item <BootLetter>:\EFI\Microsoft\Boot\bootmgfw.efi <BootLetter>:\EFI\BOOT\BOOTAA64.EFI -Force


5. Raspberry Pi 4 config.txt
----------------------------

Base Pi 4 config:

arm_64bit=1
arm_boost=1
enable_uart=1
uart_2ndstage=1
enable_gic=1
armstub=RPI_EFI.fd
disable_commandline_tags=1
disable_overscan=1
device_tree_address=0x3e0000
device_tree_end=0x400000
dtoverlay=miniuart-bt
dtoverlay=upstream-pi4

If the Pi boots but the display is blank, add:

hdmi_force_hotplug=1
hdmi_force_hotplug:0=1
hdmi_group=2
hdmi_group:0=2
hdmi_mode=82
hdmi_mode:0=82
config_hdmi_boost=7


Summary
-------

WoR prepares the Windows ARM64 image and Raspberry Pi drivers.

If WoR's BCD step fails, manual bcdboot plus the bcdedit settings above repair the boot path.

Always confirm the BOOT and Windows partition letters before running bcdboot or bcdedit.
