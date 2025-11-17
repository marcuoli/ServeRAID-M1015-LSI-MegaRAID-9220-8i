# ServeRAID-M1015-LSI-MegaRAID-9220-8i

ServeRAID M1015/LSI MegaRAID 9220-8i Firmware IT mode

> ⚠️ **Disclaimer:** Flash the firmware entirely at your own risk. These steps may brick the controller or other hardware if applied incorrectly. Double-check every command and ensure you have backups; the authors of this guide are not responsible for any damage or data loss.

[GA-Z77N-WIFI (rev. 1.0)](https://www.gigabyte.com/br/Motherboard/GA-Z77N-WIFI-rev-10)

[Rufus Download](https://rufus.ie/pt_BR/#download) [Rufus 4.11p](https://github.com/pbatard/rufus/releases/download/v4.11/rufus-4.11p.exe)

[LSI-9211-8i](https://www.mediafire.com/download/6mtie10d9ud6675/LSI-9211-8i.zip)

[LSI-Utils (megarec.exe & sbrempty.bin)](https://anthony-paul.com/wp-content/uploads/2016/10/LSI-Utils.zip)

Create a FreeDOS bootable pen drive with Rufus

1. Plug an empty USB stick (at least 512 MB) into the PC where you will run Rufus.
2. Launch Rufus and select the USB device in the Device dropdown.
3. Click *SELECT* only if you want to pick a specific FreeDOS image. Otherwise leave it blank because Rufus can generate FreeDOS automatically.
4. Set *Boot selection* to `FreeDOS`, keep Partition scheme as `MBR`, and Target system as `BIOS (or UEFI-CSM)`.
5. Optionally rename the volume label (for example `FREEDOS_FLASH`). Leave File system as `FAT32` and Cluster size as `4096 bytes` (default).
6. Click *START*, confirm when prompted to download the FreeDOS files, and wait for Rufus to finish writing the drive.
7. Once complete, copy the firmware utilities (`megarec`, `sas2flash`, `.bin` files) to the root of the USB drive so they are available during boot.

Boot the target machine from this USB drive and run the commands below.

```bash
megarec -writesbr 0 sbrempty.bin

megarec -cleanflash 0
```

Power the machine down before proceeding.

If the GA-Z77N-WIFI board only allows flashing utilities from an UEFI environment, add the required shell and boot through UEFI:

1. Download `shellx64_v1.efi` from [archive.org](https://archive.org/download/shellx64_v1/shellx64_v1.efi) (this version is confirmed to work with this board's firmware).
2. Rename the downloaded file to `BOOTX64.EFI`.
3. On the USB drive, create the folders `EFI` and `EFI/BOOT`, then place `BOOTX64.EFI` inside `EFI/BOOT/`.
4. Leave `sas2flash.efi` and the firmware `.bin` files in the root of the USB drive for easy access.
5. On the target machine, enter the firmware setup, disable Secure Boot/CSM if required, and choose the USB drive's *UEFI* boot entry so it loads this shell.
6. In the shell prompt, switch to the USB filesystem (usually `fs0:`) and run the flashing commands.

```bash
# Only if necessary
# sas2flash.efi -o -e 6 or sas2flash.efi -o -e 7 

# sas2flash.efi -o -f 2118it.bin -b mptsas2.rom
sas2flash.efi -o -f 2118it.bin
```

Record the adapter's SAS address before running the final command:

1. Find the white sticker on the PCB (usually near the PCI bracket) labeled `SAS Address` or `WWN`. ![LSI SAS 9220-8i IBM ServeRAID M1015](images/LSI%20SAS%209220-8i%20IBM%20ServeRaid%20M1015.png)
2. Note the 16-character hexadecimal value that typically starts with `500605B`.
3. Substitute that value for the placeholder in the command below so the controller keeps its factory-assigned address.

```bash
# sas2flash.efi -o -sasadd 500605bxxxxxxxxx

# Example value taken from the label shown above
sas2flash.efi -o -sasadd 500605b002BDC5A0
```

Reboot the server.

From linux:

```bash
root@localhost:~# lspci -knn | grep -i 'sas\|scsi\|raid'
01:00.0 Serial Attached SCSI controller [0107]: Broadcom / LSI SAS2008 PCI-Express Fusion-MPT SAS-2 [Falcon] [1000:0072] (rev 03)
        Kernel driver in use: mpt3sas
        Kernel modules: mpt3sas
```

<!-- markdownlint-disable MD033 -->
<pre><code>root@localhost:~# dmesg | grep -e 'Protocol\|FWVersion\|port enable:'
[    1.931784] mpt2sas_cm0: LSISAS2008: FWVersion(20.00.07.00), ChipRevision(0x03)
[    1.932158] mpt2sas_cm0: Protocol=(<mark><em>Initiator</em></mark>), Capabilities=(Raid,TLR,EEDP,Snapshot Buffer,Diag Trace Buffer,Task Set Full,NCQ)
[   20.375045] mpt2sas_cm0: port enable: SUCCESS
</code></pre>
<!-- markdownlint-enable MD033 -->
