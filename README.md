# TPM2-LUKS
Script for using a TPM2 to store a LUKS key and automatically unlock an encrypted system drive at boot.  
### Use at your own risk, I make no guarantees and take no responsibility for any damage or loss of data you may suffer as a result of running the script!

Based on:<br>
https://run.tournament.org.il/ubuntu-18-04-and-tpm2-encrypted-system-disk/<br>
and<br>
https://run.tournament.org.il/ubuntu-20-04-and-tpm2-encrypted-system-disk/<br>
Thanks etzion!

The script has been tested on Ubuntu 22.04 with full disk encryption on LVM.  It will likely work on earlier versions back to Ubuntu 18.04, but those haven't been tested.  Your drive must already be encrypted, this script will not do it for you!  If you ZFS encryption instead of LUKS this script will not work for you.

The script will create a new 64 character alpha-numeric random password, store it in the TPM2, add it to LUKS, and modify initramfs to pull it from the TPM2 automatically at boot.  The new key is in addition to the any already used for unlocking the drive.  If the TPM2 unlocks fails at boot, it will revert to asking you for the passphrase.  You can use either the original one you used to encrypt the drive, or the one that was supposed to be in the TPM2.

# Usage
Download the script, mark it as executable via the file properties or with the "chmod +x tpm2_luks_boot_unlock.sh" command.  Run the script and it will walk you through device selection.  Sudo rights are required, the script will prompt you for your password as needed.

If the drive unlocks as expected after using the script, you can optionally remove the original password used to encrypt the drive and rely completely on the random new one stored in the TPM2.  THIS IS NOT RECOMMENDED!  If you do this, you should keep a copy of the key somewhere saved on a DIFFERENT system, or printed and stored in a secure location so you can manually enter it at the prompt if something goes wrong. To get a copy of your key for backup purposes, run this command:
```
echo $(sudo tpm2-nvread 0x1500016)
```

### If you remove the original password used to encrypt the drive and don't have a copy of the key in then TPM2 then experience TPM2, motherboard, or another failure preventing auto-unlock, you WILL LOSE ACCESS TO EVERYTHING ON THE DRIVE!

If you are SURE you have a backup of the key you put in the TPM2, here is the command to remove the original password:
```
sudo cryptsetup luksRemoveKey <device name, e.g. /dev/sda3>
```

# Troubleshooting
If booting fails, press esc at the beginning of the boot to get to the grub menu.  Edit the Ubuntu entry and add .orig to end of the initrd line to boot to the original initramfs this one time. e.g.:
```
initrd /initrd.img-5.15.0-27-generic.orig
```
If that also fails, you may be able to boot to a previous kernel version under Advanced boot options.

# Known Issues
1) This only works for TPM 2.0 devices (including AMD fTPM and Intel PTT) and does NOT work for older TPM 1.2 devices
2) Just storing a value in the TPM isn't the best or most secure method.  It is a "good enough" method meant to protect from "normal" threats like a thief stealing your laptop and not a sophisticated attacker with physical and/or root access.  It should also be combined with protections like preventing USB booting and a BIOS password.  See # https://run.tournament.org.il/ubuntu-20-04-and-tpm2-encrypted-system-disk/#comment-501794 for further discussion on this from etzion.  If you know how to better use a TPM (e.g. with certificates and/or PCR registers) and would like to contribute, please reach out!

# To Do
1) Accept command line parameters
