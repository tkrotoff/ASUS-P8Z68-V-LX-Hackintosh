# Asus-P8Z68-V-LX-Hackintosh

Hackintosh for [ASUS P8Z68-V LX](https://www.asus.com/Motherboards/P8Z68V_LX/) motherboard using macOS 10.12 Sierra.
This is a minimal guide that fits my hardware configuration.

Intel Z68 chipset, LGA1155 socket.
Supports 2nd gen. ([32 nm - Sandy Bridge](http://en.wikipedia.org/wiki/Sandy_Bridge)) Intel Core CPUs.

Onboard devices:
- Realtek 8111E Gigabit LAN controller
- Realtek ALC887 8-Channel High Definition Audio CODEC
- ASMedia USB 3.0 controller (the 2 blue USB ports at the back panel)

## BIOS Settings

Latest stable BIOS: version [4105 (2013/07/23 update, 2013/07/01 build date)](https://www.asus.com/Motherboards/P8Z68V_LX/HelpDesk_Download/)
- Del: enter BIOS
- F8: list the boot entries
- F5: Optimized Defaults
- Boot > Setup Mode - Advanced Mode

## Patched BIOS to avoid kernel panic with AppleIntelCPUPowerManagement.kext

You can use NullCPUPowerManagement.kext (no Intel SpeedStep and no sleep) or patch the BIOS using [UEFIPatch](https://github.com/LongSoft/UEFITool):
```Shell
curl -O http://dlcdnet.asus.com/pub/ASUS/mb/LGA1155/P8Z68-V_LX/P8Z68-V-LX-ASUS-4105.zip
open P8Z68-V-LX-ASUS-4105.zip
curl -OL https://github.com/LongSoft/UEFITool/releases/download/0.20.5/UEFIPatch_0.3.5_osx.zip
open UEFIPatch_0.3.5_osx.zip
cd UEFIPatch_0.3.5_osx
./UEFIPatch ../P8Z68-V-LX-ASUS-4105.ROM # Generates P8Z68-V-LX-ASUS-4105.ROM.patched
```

/!\ This operation can brick your motherboard, do it at your own risk /!\
Flash your BIOS using "ASUS EZ Flash 2 Utility" and file `P8Z68-V-LX-ASUS-4105.ROM.patched`.

Sources:
- [Power Management on Asus 1155 Motherboards](http://www.tonymacx86.com/bios-uefi/43486-asus-1155-patched-bios-repository.html)
- [[UEFIPatch] UEFI patching utility](http://www.insanelymac.com/forum/topic/285444-uefipatch-uefi-patching-utility/)

## DSDT

My tests (sleep, wake, shutdown...) have concluded that there is no need to generate a patched `DSDT.aml` file.

Sources:
- [How to edit your own DSDT with MaciASL](http://www.macbreaker.com/2014/03/how-to-edit-your-own-dsdt-with-maciasl.html)
- [Creating a DSDT using MaciASL](http://pjalm.com/forums/index.php?topic=3.0)
- [Fork of MaciASL by RehabMan](https://github.com/RehabMan/OS-X-MaciASL-patchmatic)
- [ASUS DSDT patches repository for MaciASL by PJALM](http://maciasl.sourceforge.net/pjalm/asus/) ([available files](http://maciasl.sourceforge.net/pjalm/asus/.maciasl))
- [olarila.com - DSDT patches by motherboard](http://olarila.com/forum/packs.php)
- [Beta Asus Sandy Bridge Sleep/Wake Fix](http://www.tonymacx86.com/dsdt/50036-beta-asus-sandy-bridge-sleep-wake-fix.html)

## MultiBeast

Using version 9.x

- Quick Start > UEFI Boot Mode
- Drivers > Audio > Realtek ALCxxx > ALC887/888b
- Drivers > Misc > FakeSMC
- Drivers > Network > Realtek > RealtekRTL8111
- Drivers > USB > 3rd Party USB 3.0
- Bootloaders > Clover UEFI Boot Mode
- Customize > System Definitions > iMac > iMac 12,2
- Customize > SSDT Options > Sandy Bridge Core i7 (or Core i5 depending on your CPU)

Sources:
- [ASUS P8Z68-V PRO/GEN3 Hackintosh - Multibeast 8.x](https://github.com/DavidGoldman/ASUS-P8Z68-V-PRO-GEN3-Hackintosh/blob/1b4146189ed79fee06d1c3515e524af2fb5e792e/README.md#multibeast-8x)

## Mount EFI partition

```Shell
diskutil list
mkdir /Volumes/EFI
sudo mount_msdos /dev/disk0s1 /Volumes/EFI
```

## Verbose mode

Add `-v` flag to file `/Volumes/EFI/EFI/CLOVER/config.plist`:
```XML
<key>Arguments</key>
<string>dart=0 -v</string>
```

## Enable trim

`sudo trimforce enable`

## Nvidia Web Driver

Can be disabled using `nv_disable=1` at boot time.

### OsxAptioFix2Drv-64.efi vs OsxAptioFixDrv-64.efi

When using Nvidia Web Driver with a [MSI GeForce GTX 970 GAMING 4G](https://www.msi.com/Graphics-card/GTX-970-GAMING-4G.html), I randomly get `Error allocating 0x800 pages` at boot time in verbose mode.
I couldn't fix this issue.

I've tried to replace `OsxAptioFix2Drv-64.efi` by `OsxAptioFixDrv-64.efi` without success: I then always get `Error - requested memory exceeds our allocated relocation block`.

Tested using Clover v2.3k r3766 (packaged with MultiBeast 9.0.1), r3974 and r3998.

Sources:
- [NVIDIA Releases Alternate Graphics Drivers for macOS Sierra 10.12.3](https://www.tonymacx86.com/threads/nvidia-releases-alternate-graphics-drivers-for-macos-sierra-10-12-3-367-15-10-35.213122/)
- [Big List of Solutions for El Capitan Install Problems](https://www.tonymacx86.com/threads/big-list-of-solutions-for-el-capitan-install-problems.173991/#CategoryFreeze)
- [ASUS P8Z68-V PRO/GEN3 Hackintosh - Prohibited Sign on Boot](https://github.com/DavidGoldman/ASUS-P8Z68-V-PRO-GEN3-Hackintosh/blob/1b4146189ed79fee06d1c3515e524af2fb5e792e/README.md#prohibited-sign-on-boot)

## BIOS boot entry

The ASUS P8Z68 UEFI BIOS will recognize a USB key configured with Clover "Install for UEFI booting only" but not a hard drive.

Using old Clover versions you could boot on using Clover USB key and add the boot entry for your hard drive into the BIOS using "Clover Boot Options" > "Add Clover boot options for all entries".
This is not possible anymore (why?), instead you will need to manually add the boot entry.

Open "Clover > Start UEFI Shell" and play with `bcfg`:

```Shell
map fs* # Show all partitions
fs0: # Switch to fs0, fs1, fs2... partitions
ls # List the content
ls /efi/boot
bcfg boot dump # List current boot entries
bcfg boot add N bootx64.efi "Clover" # Add /efi/bootbootx64.efi as a boot entry labeled "Clover", N being 0 (first boot entry), 1 (second boot entry)...
bcfg boot rm N # Remove a boot entry given its number N in the list
exit # Get back to Clover main screen
reset # Restart the computer
```

Sources:
- [[GUIDE] ASRock H97 Pro4 Yosemite with Clover UEFI Installation](http://www.insanelymac.com/forum/topic/302041-guide-asrock-h97-pro4-yosemite-with-clover-uefi-installation/)
- [ASUS P8z68 won't boot without USB-stick](http://www.tonymacx86.com/yosemite-desktop-support/158425-asus-p8z68-wont-boot-without-usb-stick.html)
- [Booting UEFI with Clover on Asus P8Z68-V/GEN3](https://www.reddit.com/r/hackintosh/comments/21ywt4/booting_uefi_with_clover_on_asus_p8z68vgen3/)
- [[How To] Remove Extra Clover BIOS Boot Entries & Prevent Further Problems](http://www.insanelymac.com/forum/topic/308637-how-to-remove-extra-clover-bios-boot-entries-prevent-further-problems/)

## Performance

Using [Geekbench](http://www.primatelabs.com/geekbench/) with an Intel Core i7-2700K @ 3.50 GHz
- Version 3 32bits: > 3000 (single-core), > 11000 (multi-core)
- Version 4 64bits: > 3600 (single-core), > 11000 (multi-core)

Using [Unigine Valley Benchmark 1.0](https://unigine.com/products/benchmarks/valley/) in 1920x1080/medium with a GTX 970, you should get a score > 2900

## Other tools and links

- [RTL8111 Driver for OS X](https://github.com/Mieze/RTL8111_driver_for_OS_X): RealtekRTL8111.kext source code

## License

Do whatever you like, this is public domain.
