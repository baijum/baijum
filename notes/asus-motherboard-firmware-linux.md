# Updating ASUS Motherboard Firmware

Recently I updated my [PC's motherboard firmware](https://www.asus.com/in/motherboards-components/motherboards/csm/prime-z790-a-wifi-csm/helpdesk_bios/?model2Name=PRIME-Z790-A-WIFI-CSM).
The ASUS Motherboard firmware comes with a Microsoft Windows executable program to rename the firmware program file.
Originally it came with a generic lengthy name.
But that needs to be changed with a particular name and copied to a USB device.
The downloaded Zip file contains a Microsoft Windows executable program to rename it.
Since I use GNU/Linux and have no access to the Microsft Windows system, I used [Wine](https://www.winehq.org/) to execute the program.

```
wine BIOSRenamer.exe 
```

As a preparation, I need to format a USB device with the FAT32 file system.

Once inserted, I could see the [USB device in the system](https://www.baeldung.com/linux/check-for-usb-devices).
To format the drive, I used fdisk.

To see all the storage devices and existing partitions, I run `fdisk -l` command.
Then I ran the `fdisk` command with the device identifier as the argument, like this:

`fdisk /dev/sda`

After formating this is how my partition table looks like:

```
Disk /dev/sda: 29.3 GiB, 31457280000 bytes, 61440000 sectors
Disk model: Disk 2.0        
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3db87b5e

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1  *     2048 61439999 61437952 29.3G  c W95 FAT32 (LBA)
```

I used the partition type as `W95 FAT32 (LBA)` (Id: c)

To format the device, I ran this command:

```
mkfs.vfat /dev/sda1
```

Then mount:

```
mount /dev/sda1 /mnt/b
```

Then I copied my renamed driver file to the device

```
cp PZ790AW.CAP /mnt/b
```

After this, I rebooted the system and updated the BIOS from the BIOS settings.
