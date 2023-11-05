

1: identify secondary hard drive(s)

        lsblk

2: Format secondary hard drives

- To format the disk using linux standard ext4.

        sudo mkfs -t ext4 /dev/sdb

###### NOTE: MKFS can format disks to either "ext4", "FAT32" or "NTFS"

- To format the disk using FAT32.

        sudo mkfs -t vfat /dev/sdb

- To format the disk using Windows Standard NTFS.

        sudo mkfs -t ntfs /dev/sdb

(Breakdown)

        sudo mkfs -t [format] [secondary drive address]

3: Create a directory to mount the disk volume into (created in /mnt directory).

        sudo mkdir /mnt/virtio-hdd

4: Mount the secondary disk into the new /mnt directory.

        sudo mount /dev/sdb /mnt/virtio-hdd

(Breakdown)

        sudo mount /[secondary drive] /[new mnt directory]

5: Check secondary drive has mounted successfully under the specified directory.

        lsblk

6: Set secondary drive to automatically mount to specified /mnt directory on boot.

        sudo nano /etc/fstab

7: Add secondary drive to the bottom of fstab.

        /dev/sdb        /mnt/virtio-hdd         ext4    defaults                0 2

###### NOTE: functions are seperated with TABS not with SPACES.

###### NOTE 2: In the event the host reboots into emergency mode, an error with the configuration of the command above has occurred and is causing boot issues. Repeat Step 6: and verify the host's fstab against Step 7: example.

(Breakdown)

        [secondary drive address]           [mount directory]           [disk filesystem type]          [options]           [dump]          [fsck]

###### NOTE 3: If you are unsure of the disk's file system, instead of specifying the filesystem (ie ext4 or FAT32) you can instead specify "auto" and fstab should automatically determine the file system.

###### NOTE 4: DO NOT REBOOT THE HOS UNTIL YOU HAVE COMPLETED STEP 8.

8: Verify the changes to fstab are formatted correctly.

        mount -a

###### NOTE: If the command runs without outputting any text, the command has run through successfully. If it returns a format error, something has gone wrong - recheck Step 7:.

- To check fstab without running the system call.

        mount -f

9: Reboot server

        sudo reboot

10: Ensure secondary drive has automatically mounted to the host.

        lsblk

![console view showing mounted drive](github.com)