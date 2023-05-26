# TPM2-LUKS
Script for using a TPM2 to store a LUKS key and automatically unlock an encrypted system drive at boot.  
### Use at your own risk, I make no guarantees and take no responsibility for any damage or loss of data you may suffer as a result of running the script!

Based on:<br>
https://run.tournament.org.il/ubuntu-18-04-and-tpm2-encrypted-system-disk/<br>
and<br>
https://run.tournament.org.il/ubuntu-20-04-and-tpm2-encrypted-system-disk/<br>
Thanks etzion!

The script has been tested on Ubuntu 20.04 and 22.04 with full disk encryption on LVM.  Your drive must already be encrypted, this script will not do it for you!  If you selected ZFS and encryption during the Ubuntu 22.04 install, it will use ZFS native encryption and not LUKS, so this script will not work.

It will create a new 64 character alpha-numeric random password, store it in the TPM, add it to LUKS, and modify initramfs to pull it from the TPM automatically at boot.  The new key is in addition to the any already used for unlocking the drive.  If the TPM unlocks fails at boot, it will revert to asking you for a passphrase.  You can use either the original one you used to encrypt the drive, or the one that this script added the TPM, if you have a record of it.

# Usage

sudo ./tpm2-luks-unlock.sh [<device path>]
  
Examples:
sudo ./tpm2-luks-unlock.sh
  This will run the script against the first volume listed in /etc/crypttab, which is usually sufficient for booting
  
sudo ./tpm2-luks-unlock.sh /dev/sda3
  This will run the script against /dev/sda3

If desired, you can modify the KEYSIZE variable near the top of the size to change the size of the key stored in the TPM.

If the drive unlocks as expected after using the script, you can optionally remove the original password used to encrypt the drive and rely completely on the random new one stored in the TPM.  If you do this, you should keep a copy of the key somewhere saved on a DIFFERENT system, or printed and stored in a secure location on another system so you can manually enter it at the prompt if something goes wrong. To get a copy of your key for backup purposes, run this command:
```
echo `sudo tpm2_nvread 0x1500016 2\> /dev/null`
```

### If you remove the original password used to encrypt the drive and haven't backed up the key in the TPM, then a failure of the TPM, motherboard, or anything else prevents automatic unlocking, YOU WILL LOSE ACCESS TO EVERYTHING ON THE DRIVE!

If you are SURE you have a backup of the key you put in the TPM, here is the command to remove the original password.  Replace /dev/sda3 with the path to your target device:
```
sudo cryptsetup luksRemoveKey /dev/sda3
```

### Warning
If you run the script more than once on the same system, it will *add* a new key to LUKS for the device, leaving all existing keys in place.  However, the previous key stored in the TPM will be *overwritten*.  If you are running it on the same device (e.g. to change the key or key size) this should be fine.  If you are trying to run it against a different device, the key in the TPM for the first device will be overwritten and it will no longer automatically unlock.

# Troubleshooting
If booting fails, press esc at the beginning of the boot to get to the grub menu.  Edit the Ubuntu entry and add .orig to end of the initrd line to boot to the original initramfs this one time. e.g.:
```
initrd /initrd.img-5.15.0-27-generic.orig
```
If that also fails, you may be able to boot to a previous kernel version under Advanced boot options.

# Known Issues
1) This only works for TPM 2.0 devices (including AMD fTPM and Intel PTT) but does NOT work for TPM 1.2 devices
2) Just storing a value in the TPM isn't the best or most secure method.  It is a "good enough" method meant to protect from "normal" threats like a thief stealing your laptop and not a sophisticated attacker with physical and/or root access.  It should also be combined with protections like preventing USB booting and a BIOS password.  See https://run.tournament.org.il/ubuntu-20-04-and-tpm2-encrypted-system-disk/ for further discussion on this from etzion.  If you know how to better use a TPM (e.g. with certificates and/or PCR registers) and would like to contribute, please reach out!

# To Do
1) Detect if TPM area already allocated, if so ask to overwrite or reuse (e.g. for additional drives)
