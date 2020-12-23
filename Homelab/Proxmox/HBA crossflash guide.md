# HBA crossflash guide

Crossflashing an LSI 9211-8i from **IR** mode, to **IT** mode

## Details

- Some of these steps are specific to `Asrock Z370 Extreme4` motherboard
- The files required are in `/9211_files`

## Preperation

1. Format the usb drive into `FAT32`
2. Get [this](https://github.com/tianocore/edk2/blob/UDK2018/EdkShellBinPkg/FullShell/X64/Shell_Full.efi) EFI shell (or the one included here)
3. Rename the `Shell_Full.efi` to `shellx64.efi` and place it into the root of the usb drive
4. Get the firmware files from [here](https://docs.broadcom.com/docs/12350530) and place the following files onto the usb drive

    ```files
    Firmware/HBA_9211_8i_IT/2118it.bin
    sasbios_rel/mptsas2.rom
    ```
5. Download the efi flash tool from [here](https://docs.broadcom.com/docs/12350820) and place the following file onto the usb drive
    
    ```files
    sas2flash_efi_ebc_rel/sas2flash.efi
    ```

## The flashing process

1. Boot into the efi shell (this is motherboard dependant)
    1. Enter the UEFI Setup
    2. On the far right `Exit` tab -> `Launch EFI Shell from filesystem device`
2. Mount the filesystem that the files are on
    1. The `map` command lists the drives
    2. Use `mount fsX` to mount the filesystem where X is the corrent drive
    3. Use `fsX:` to switch to that drive
3. Use `sas2flash.efi -list` or `sas2flash.efi -listall` to see whether the HBA is detected
4. Use `sas2flash.efi -o -e 6` to erase the current BIOS and enter flashing mode **DO NOT REBOOT after this step or the device will be bricked**
5. Use `sas2flash.efi -o -f 2118it.bin -b mptsas2.rom` to flash the IT mode binary and the new BIOS
6. Repeat step 3 to see whether the device is now in the correct mode
7. Reboot

### Links

- https://digitalcardboard.com/blog/2014/07/09/flashing-it-firmware-to-the-lsi-sas-9211-8i-hba-2014-efi-recipe/
- https://www.truenas.com/community/threads/how-to-flash-lsi-9211-8i-using-efi-shell.50902/
- https://nguvu.org/freenas/Convert-LSI-HBA-card-to-IT-mode/
- http://brycv.com/blog/2012/flashing-it-firmware-to-lsi-sas9211-8i/
- https://serverfault.com/questions/679175/failed-to-initialize-pal-while-upgrading-an-lsi-9211-8i-to-it