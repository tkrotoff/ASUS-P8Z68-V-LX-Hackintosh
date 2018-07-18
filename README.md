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
- Del: enter the BIOS setup
- F8: display the boot menu

- F5: reset to default settings (Optimized Defaults)
- Advanced > USB Configuration > EHCI Hand-off: Enabled (optional)
- Advanced > Onboard Devices Configuration > Serial Port Configuration > Serial Port: Disabled (optional)
- Boot > Full Screen Logo: Disabled (optional)
- Boot > Setup Mode: Advanced Mode (optional)

Do not touch option "Boot > PCI ROM Priority", in my case it crashes the BIOS and a clear CMOS is then needed

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
sudo mkdir /Volumes/EFI
sudo mount_msdos /dev/disk0s1 /Volumes/EFI
```

## Verbose mode

Add `-v` flag to file `/Volumes/EFI/EFI/CLOVER/config.plist`:
```XML
<key>Arguments</key>
<string>dart=0 -v</string>
```

## Enable trim

If you own a SSD you should enable TRIM support:

```Shell
sudo trimforce enable
```

Sources:
- [How to Enable TRIM on Third Party SSDs in Mac OS X with trimforce](http://osxdaily.com/2015/10/29/use-trimforce-trim-ssd-mac-os-x/)
- [Enable TRIM for Third-Party SSDs in OS X with a Terminal Command](http://lifehacker.com/enable-trim-for-third-party-ssds-in-os-x-with-a-termina-1714978260)

## Nvidia Web Driver

Can be disabled using `nv_disable=1` at boot time.

### OsxAptioFix2Drv-64.efi vs OsxAptioFixDrv-64.efi

When using Nvidia Web Driver with a [MSI GeForce GTX 970 GAMING 4G](https://www.msi.com/Graphics-card/GTX-970-GAMING-4G.html), I randomly get `Error allocating 0x800 pages` at boot time in verbose mode.
I couldn't fix this issue.

I've tried to replace `OsxAptioFix2Drv-64.efi` by `OsxAptioFixDrv-64.efi` without success: I then always get `Error - requested memory exceeds our allocated relocation block`.

Tested using Clover v2.3k r3766 (packaged with MultiBeast 9.0.1), r3974 and r3998.

See [issue #2](https://github.com/tkrotoff/ASUS-P8Z68-V-LX-Hackintosh/issues/2) for possible solutions.

Sources:
- [NVIDIA Releases Alternate Graphics Drivers for macOS Sierra 10.12.3](https://www.tonymacx86.com/threads/nvidia-releases-alternate-graphics-drivers-for-macos-sierra-10-12-3-367-15-10-35.213122/)
- [Big List of Solutions for El Capitan Install Problems](https://www.tonymacx86.com/threads/big-list-of-solutions-for-el-capitan-install-problems.173991/#CategoryFreeze)
- [ASUS P8Z68-V PRO/GEN3 Hackintosh - Prohibited Sign on Boot](https://github.com/DavidGoldman/ASUS-P8Z68-V-PRO-GEN3-Hackintosh/blob/1b4146189ed79fee06d1c3515e524af2fb5e792e/README.md#prohibited-sign-on-boot)

## BIOS boot entry

The ASUS P8Z68 UEFI BIOS will recognize a USB key configured with Clover "Install for UEFI booting only" but not a hard drive.

Using old Clover versions you could boot on using Clover USB key and add the boot entry for your hard drive into the BIOS using "Clover Boot Options" > "Add Clover boot options for all entries".
This is not possible anymore (why?), instead you will need to manually add the boot entry.

Open "Clover > Start UEFI Shell 64" and play with `bcfg`:

```Shell
map fs* # Show all partitions
fs0: # Switch to fs0, fs1, fs2... partition
ls # List the partition content
ls EFI/BOOT
bcfg boot dump # List current boot entries
bcfg boot add N EFI/BOOT/BOOTX64.EFI "Clover" # Add EFI/BOOT/BOOTX64.EFI as a boot entry labeled "Clover", N being 0 (first boot entry), 1 (second boot entry)...
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

Using Cinebench R15: OpenGL > 60 fps, CPU > 600 cb

## Other tools and links

- [RTL8111 Driver for OS X](https://github.com/Mieze/RTL8111_driver_for_OS_X): RealtekRTL8111.kext source code


## Clover

Download and run Clover:

- Installation Type > Change Install Location... - Select the disk where you want to install Clover
- Installation Type > Customize - Check "Install for UEFI booting only"
- Installation Type > Drivers64UEFI - OsxAptioFixDrv-64 (Fix memory problems for American Megatrends Inc. AMI Aptio UEFI BIOS)
- Installation Type > Customize - Check "Install RC scripts on target volume" (Only when installing on a hard drive, not needed when creating a bootable USB key)

Sources:
- [All in one guides for Hackintosh](http://www.insanelymac.com/forum/topic/298027-guide-aio-guides-for-hackintosh/)
- [Gigabyte Z77X UD5H Clover UEFI Install/Tweak guide](http://www.insanelymac.com/forum/topic/288829-guide-gigabyte-z77x-ud5h-clover-uefi-installtweak-guide/)

The ASUS P8Z68 has a buggy? UEFI BIOS: it will recognize a USB key configured with Clover "Install for UEFI booting only" but not a hard drive.
Boot on the Clover USB key and manually add the boot entry for your hard drive into the BIOS using "Clover Boot Options" > "Add Clover boot options for all entries".

Sources:
- [ASUS P8z68 won't boot without USB-stick](http://www.tonymacx86.com/yosemite-desktop-support/158425-asus-p8z68-wont-boot-without-usb-stick.html)
- [Clover just WILL NOT see OS X!](http://www.tonymacx86.com/yosemite-desktop-support/147375-clover-just-will-not-see-os-x-2.html#post913505)
- [Booting UEFI with Clover on Asus P8Z68-V/GEN3](https://www.reddit.com/r/hackintosh/comments/21ywt4/booting_uefi_with_clover_on_asus_p8z68vgen3/)

## Clover Configurator

- Acpi > IntelGFX
- Boot > Verbose (-v)
- ~~Boot > dart=0~~
- ~~Devices > Fake ID > IntelGFX 0x01168086~~
- ~~Graphics > Patch VBios~~
- Graphics > Inject Intel (for HD3000)
- SMBIOS > iMac12,2

## Mount the EFI system partition

(No need to download and install an application like EFI Mounter, `diskutil` will do it)

```Shell
diskutil list # List the partitions
diskutil umount EFI_IDENTIFIER # EFI_IDENTIFIER will be disk0s1 most of the time
diskutil mount EFI_IDENTIFIER
```

The Clover installer automatically mounts the partition at /Volumes/ESP instead of /Volumes/EFI

## HFSPlus.efi

Mandatory

```Shell
rm /Volumes/EFI/EFI/CLOVER/drivers64UEFI/VBoxHfs-64.efi
curl -OL https://github.com/JrCs/CloverGrowerPro/raw/master/Files/HFSPlus/X64/HFSPlus.efi
cp HFSPlus.efi /Volumes/EFI/EFI/CLOVER/drivers64UEFI/
```

## FakeSMC.kext

Mandatory

```Shell
curl -OL https://bitbucket.org/RehabMan/os-x-fakesmc-kozlek/downloads/RehabMan-FakeSMC-2015-0504.zip
open RehabMan-FakeSMC-2015-0504.zip
cp -r RehabMan-FakeSMC-2015-0504/*.kext /Volumes/EFI/EFI/CLOVER/kexts/10.10/
```

## DSDT and SSDT

If you have a custom DSDT or SSDT

```Shell
cp DSDT.aml /Volumes/EFI/EFI/CLOVER/ACPI/patched/
cp SSDT.aml /Volumes/EFI/EFI/CLOVER/ACPI/patched/
```

## Other popular kexts

### RealtekRTL8111.kext

Driver for the Realtek RTL8111/8168 family

```Shell
curl -OL https://bitbucket.org/RehabMan/os-x-realtek-network/downloads/RehabMan-Realtek-Network-v2-2015-0526.zip
open RehabMan-Realtek-Network-v2-2015-0526.zip
cp -r RehabMan-Realtek-Network-v2-2015-0526/Release/RealtekRTL8111.kext /Volumes/EFI/EFI/CLOVER/kexts/10.10/
```

### AtherosE2200Ethernet.kext

Driver for Qualcomm Atheros AR816x, AR817x and Killer E220x
Manually download it from http://www.insanelymac.com/forum/files/file/313-atherose2200ethernet/ then:

```Shell
open ~/Downloads/AtherosE2200Ethernet-V2.0.1.zip
cp -r ~/Downloads/AtherosE2200Ethernet-V2.0.1/Release/AtherosE2200Ethernet.kext /Volumes/EFI/EFI/CLOVER/kexts/10.10/
```

### Realtek ALCxxx onboard audio

Patch AppleHDA.kext for Realtek ALC Audio support

```Shell
curl -OL https://github.com/toleda/audio_CloverALC/raw/master/audio_cloverALC-110.command.zip
open audio_cloverALC-110.command.zip
./audio_cloverALC-110_v1.0f.command
[...]
Confirm Realtek ALC887 (y/n): y
Enable HD4600 HDMI audio (y/n): n
Clover Audio ID Injection (y/n): y
Use Audio ID: 1 (y/n): n
Audio IDs:
1 - 3/5/6 port Realtek ALCxxx audio
2 - 3 port (5.1) Realtek ALCxxx audio (n/a 885)
3 - HD3000/HD4000 HDMI and Realtek ALCxxx audio (n/a 885/1150 & 887/888 Legacy)
Select Audio ID: 3
[...]
```

## DSDT

### HD3000

####

```
into method label _DSM parent_adr 0x00020000 remove_entry;
into device name_adr 0x00020000 insert
begin
Method (_DSM, 4, NotSerialized)
{
If (LEqual (Arg2, Zero)) { Return (Buffer() { 0x03 } ) }
Return (Package()
{
"device-id", Buffer () { 0x26, 0x01, 0x00, 0x00 },
"hda-gfx", Buffer () { "onboard-1" },
"AAPL,snb-platform-id", Buffer () { 0x10, 0x00, 0x03, 0x00 }
})
}
```

####

```
Device (GFX0)
{
Name (_ADR, 0x00020000)
Method (_DSM, 4, NotSerialized)
{
Store (Package (0x04)
{
"device-id",
Buffer (0x04)
{
0x26, 0x01, 0x00, 0x00
},

"AAPL,snb-platform-id",
Buffer (0x04)
{
0x10, 0x00, 0x03, 0x00
}
}, Local0)
DTGP (Arg0, Arg1, Arg2, Arg3, RefOf (Local0))
Return (Local0)
}
```

##

#### Insert HDMI audio injection into device IGPU (HD3K HDMI audio - Part 1/2)

```
into scope label _DSM parent_adr 0x00020000 remove_entry;
into scope parent_adr 0x00020000 insert
begin
    Method (_DSM, 4, NotSerialized)\n
    {\n
      If (LEqual (Arg2, Zero)) { Return (Buffer() { 0x03 } ) }\n
      Return (Package()\n
      {\n
          "device-id", Buffer() { 0x26, 0x01, 0x00, 0x00 },\n
          "AAPL,snb-platform-id", Buffer() { 0x10, 0x00, 0x03, 0x00 },\n
          "hda-gfx", Buffer() { "onboard-1" },\n
      })\n
  }\n
end;
```

- [GA-Z68X-UD3H-B3 with HD3000 in clover installation?](http://www.tonymacx86.com/yosemite-desktop-support/148290-ga-z68x-ud3h-b3-hd3000-clover-installation.html#post922487)
- [Clover Intel HD 3000](http://www.tonymacx86.com/graphics/89945-clover-intel-hd-3000-a-3.html#post907803)
- [audio_hdmi_hd3000](https://github.com/toleda/audio_hdmi_hd3000/blob/master/Patches/AMI-BIOS-HD3000-6_series/sb3-hdmi_audio_ami_bios_hd3000-3.txt)

## Links

- [Clover EFI bootloader](http://sourceforge.net/projects/cloverefiboot/)
- [Clover Configurator](http://mackie100projects.altervista.org/)
- [CloverGrowerPro](https://github.com/JrCs/CloverGrowerPro)
- [FakeSMC - fork of HWSensors by RehabMan](https://github.com/RehabMan/OS-X-FakeSMC-kozlek)
- [Original HWSensors](https://github.com/kozlek/HWSensors)
- [OS X Realtek ALC onboard audio with Clover](https://github.com/toleda/audio_CloverALC)



## License

Do whatever you like, this is public domain.
