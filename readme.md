# Format, manually mount & automatically mount secondary system drives & Shared Storage (NFS/SMB) in linux.

This guide runs through 3 different setups;

1. How to Format the secondary drive(s), manually mount them for the first time in the system, then set them to automatically mount on system reboot using fstab.
2. How to mount an SMB Share on a Linux Host, & set them to automatically mount on system reboot using fstab.
3. How to mount an NFS Share on a Linux Host, & set them to automatically mount on system reboot using fstab.

This guide was written for my VM's & servers running Ubuntu Server, as such YMMV following these commands on other non-debian/non-ubuntu OS's.

![cover photo Dell SAN](https://cf-images.dustin.eu/cdn-cgi/image/fit=contain,format=auto,width=828/image/d200001004166267/dell-storage-scv2020.jpg)

## 1- Secondary System Drive Setup

1. identify secondary hard drive(s)

        lsblk

2. Format secondary hard drives

###### NOTE: MKFS can format disks to either "ext4", "FAT32" or "NTFS"

- To format the disk using linux standard ext4.

        sudo mkfs -t ext4 /dev/sdb

- To format the disk using FAT32.

        sudo mkfs -t vfat /dev/sdb

- To format the disk using Windows Standard NTFS.

        sudo mkfs -t ntfs /dev/sdb

###### NOTE 2: Press enter when mkfs asks to specify block size (leave blank). This will automatically format the partition to the max block size.

(Breakdown)

        sudo mkfs -t [format] [secondary drive address]

3. Create a directory to mount the disk volume into (created in /mnt directory).

        sudo mkdir /mnt/virtio-hdd

4. Mount the secondary disk into the new /mnt directory.

        sudo mount /dev/sdb /mnt/virtio-hdd

(Breakdown)

        sudo mount /dev/[secondary drive] /mnt/[new mnt directory]

5. Check secondary drive has mounted successfully under the specified directory.

        lsblk

![console view showing mounted drive](https://github.com/NWSpitfire/Configure-Secondary-Drives-in-Linux/blob/2867f3a8c0afd1bbc6aa386030328809dcbf4f73/images/fstab-auto-mounted-drive.png)

6. Set secondary drive to automatically mount to specified /mnt directory on boot.

        sudo nano /etc/fstab

7. Add secondary drive to the bottom of fstab.

        /dev/sdb        /mnt/virtio-hdd         ext4    defaults                0 2

###### NOTE: functions are seperated with TABS not with SPACES.

###### NOTE 2: In the event the host reboots into emergency mode, an error with the configuration of the command above has occurred and is causing boot issues. Repeat Step 6: and verify the host's fstab against Step 7: example.

(Breakdown)

        [secondary drive address]           [mount directory]           [disk filesystem type]          [options]           [dump]          [fsck]

###### NOTE 3: If you are unsure of the disk's file system, instead of specifying the filesystem (ie ext4 or FAT32) you can instead specify "auto" and fstab should automatically determine the file system.

###### NOTE 4: DO NOT REBOOT THE HOST UNTIL YOU HAVE COMPLETED STEP 8.

8. Verify the changes to fstab are formatted correctly.

        mount -a

![mount -a command output](https://github.com/NWSpitfire/Configure-Secondary-Drives-in-Linux/blob/76823a3dd788ff4c799c67957c1cfc57c7fa7bd7/images/mount-a-output.png)

###### NOTE: If the command runs without outputting any text, the command has run through successfully. If it returns a format error, something has gone wrong - recheck Step 7:.

- To check fstab without running the system call.

        mount -f

![mount -f command output](https://github.com/NWSpitfire/Configure-Secondary-Drives-in-Linux/blob/76823a3dd788ff4c799c67957c1cfc57c7fa7bd7/images/mount-f-output.png)

9. Reboot server.

        sudo reboot

10. Ensure secondary drive has automatically mounted to the host.

        lsblk

![console view showing mounted drive](https://github.com/NWSpitfire/Configure-Secondary-Drives-in-Linux/blob/2867f3a8c0afd1bbc6aa386030328809dcbf4f73/images/fstab-auto-mounted-drive.png)


## 2- CIFS Remote Shared Drive Setup

This part of the guide is for mounting shared drives from a NAS or server using SMB protocol (with authentication). It is NOT for NFS Shares.

You must follow section i. in order **before** following section ii.

### i. Installation of CIFS & Manual Mounting of an SMB Share

1. Install CIFS-Utils from the package manager, this will allow us to mount our SMB share.

        sudo apt install cifs-utils

2. Create a directory in /mnt to use as the mount point for the SMB server folder.

        sudo mkdir /mnt/ER-10TB-SMB

###### NOTE: You can call this anything, but if you are mounting multiple SMB Shares you MUST create seperate folders for each mount.

3. Manually mount the SMB share to test the connection

        sudo mount -t cifs -o username=Server //10.20.3.10/SATA-10TB-HDD/Files/examples /mnt/ER-10TB-SMB/

(Breakdown)                
                
        sudo mount -t cifs -o username=[SMB share Username] //[SMB Server IP Address]/[SMB Mount Path] /mnt/[Specific mount directory]/

###### NOTE: This will bring up an prompt to enter the SMB Share's password. By manually mounting and inputting the password when prompted the password WILL NOT be persistent (ie after reboot the share will be unmounted). Follow Section ii. to create a persistent mount.

###### NOTE 2: If the SMB share path contains spaces then a specific format for entering the SMB path must be followed or the command will fail. See below.

3a. If the SMB share your trying to mount contains directories which contain spaces (not a problem for Windows, but a problem for Linux) then you will have to follow a specific format otherwise the command will fail.

- During manual mount the CIFS command will accept spaces if preceeded by a \ . For example if the directory I wanted to mount was called "My Album" in a dataset called "Main Storage Pool" the path would look like;

        //10.20.10.5/SATA-10TB-HDD/Main\ Storage\ Pool/My\ Album

- Alternatively, you can use the correct ASCII code for space which is **040**. This will be **required** later in Section ii., Step: 6 when we come to automatically mounting the share on boot using fstab. Using the same directory "My Album" the path would look like;

        //10.20.10.5/SATA-10TB-HDD/Main\040Storage\040Pool/My\040Album

4. If all steps above were followed correctly then the SMB Share should have mounted correctly. To verify this, check the mounted directories.

        df

### ii. Automatic Mounting of an SMB Share

Assuming you have successfully mounted the SMB Share using the information above, you now need to set the share to automatically mount on system boot using fstab.

1. Create a hidden credentials file under a system directory. I use /etc but you can use any other system directory such as /root. This credentials file will contain passwords in **PLAIN TEXT**, so it needs to be hidden AND have the correct file permissions for such a file.

        sudo nano /etc/.smbcreds

###### NOTE: The . before the filename means this file will be a hidden file. If you run a basic ls to list files the file will not be shown (although running an ls -a will show the hidden files).

2. Add the SMB share's credentials into the file taking care to use no spaces in any of the inputs.

        username=Server
        password=password1234

2a. If your network uses Active Directory, you will need to add the domain to the bottom of the credentials file.

        username=Server
        password=password1234
        domain=sitedomain

2b. If you have multiple shares, do not create multiple credential files (you can, but theres not really much point). Add them all to the same file and point each share to the same credentials file.

        username=Server
        password=password1234
        domain=sitedomain

        username=Server1
        password=password4321
        domain=sitedomain                

3. Once the credentials have been saved to the file, it needs to have its permissions restricted so only the root user can access the file.

        sudo chmod 600 /etc/.smbcreds

###### NOTE: BE VERY CAREFUL WHEN RUNNING THIS COMMAND THAT THE PATH TO YOUR FILE IS CORRECT. IF THE PATH IS WRONG OR SPECIFIES THE WRONG PATH, BY RUNNING THIS AS SUDO YOU WILL SET THE PERMISSIONS OF THE TARGETED FILE/FOLDER AS ROOT R/W only.

###### NOTE 2: This is a "Safe" way of storing the SMB servers credentials. However, if the server's root user gets compromised this file will be viewable, so it is best practice to create a user on the NAS/SMB Server that only has access restricted to the required share's for this server and not the whole server/other shares in case the server and thus the credentials are compromised. DON'T USE THE SMB SERVERS ROOT OR ADMIN CREDENTIALS AS AUTHENTICATION!

- For more on securely storing SMB credentials see [Mounting CIFS FSTAB Securely](https://help.ubuntu.com/community/MountCifsFstabSecurely).

4. Ensure the /.smbcreds file has successfully had the correct permissions applied to it. The permissions should be **rw-------**.

        ls -l /etc/.smbcreds

5. Edit fstab.

        sudo nano /etc/fstab

6. Add the path to your SMB server as well as the path to the mount folder created earlier as shown below.

        //10.20.3.4/SATA-10TB-HDD/Main\040Storage\040Pool/My\040Album           /mnt/ER-10TB-SMB        cifs            noperm,_netdev,credentials=/etc/.smbcreds       0       0

###### NOTE: As discussed in Section i., Step: 3a the SMB share path must not contain any spaces of any kind. As such the path must be written with spaces represented in ASCII format using \040.

###### NOTE 2: functions are seperated with TABS not with SPACES. Spaces will cause formatting errors.

(Breakdown)

        //[SMB Server IP Address]/[SMB Mount Path]         /mnt/[Specific mount directory]/        [connection protocol - cifs]            [skip permission check],[tell kernel to hold mount until after network is up],credentials=/path/to/credentials          0       0

7. After saving, verify the formatting of the new command in fstab is correct. The command should return no output if it has run successfully.

        sudo mount -a

###### NOTE: This command must be run with sudo, otherwise it will fail with a file permission error (only root is allowed to read credentials file).

###### NOTE 2: If the command returns a formatting error, go back and run Step 6: again. Most importantly check for hidden spaces.

8. Check SMB share is mounted by listing the mounted directories.

        df

9. Reboot server to test automatic mounting of the share.

        sudo reboot.

10. If all goes well you may see the SMB Share mount during system boot. It will show up as "[   OK   ] Mounted /mnt/ER-10TB-SMB. - If you don't see it, after the system has booted check the mounted directories.

        df

The share will now automatically mount on boot using the credentials file and the fstab path. 

### TROUBLESHOOTING - Using CIFS mounts with Docker mountpoints causing permissions issues with the container.

This is because the containers UID/GID does not match the CIFS mount permissions, this can be fixed by ammending the /etc/fstab info with the correct GID/UID

1. Identify the host mount permissions and owner.

        ls -ld /path/to/mounted/volume

###### NOTE: The permissions should list something like root:root (user:group) as the owner. This means that the user "root" and the group "root" own the share.

2. Once the mount owner has been identified, identify owners UID/GID.

        id username

###### NOTE: If the owner identified in the previous command was "root" for example, then "username" = "root".

3. Add the GID/UID to /etc/fstab

        //10.20.3.10/WD-10TB-SATA/Storage/Test           /mnt/EF-10TB-Sync        cifs            noperm,_netdev,credentials=/etc/.smbcreds,uid=1000,gid=1000,forceuid,forcegid       0       0

4. Reboot system to apply changes.

        sudo reboot

### iii. EXTRA: Manual mounting of an SMB Share using a credentials file.

This can be used if you dont want to remember your password but only want the SMB share mounted manually. This can be useful for verifying the credentials file works as expected.

1. Run through Section ii. Steps 1: - 4: to create and secure the credentials file.

2. To mount the SMB share manually run the manual mount command. There is no need to specify the share's username (as in the Section i. instructions), as it will be pulled from the credentials file

        sudo mount -t cifs -o credentials=/etc/.smbcreds //10.20.3.4/SATA-10TB-HDD/Main\ Storage\ Pool/My\ Album /mnt/ER-10TB-SMB/

###### NOTE: In this case the command will accept spaces in the share path, so long as it is preceeded with a \ . However where possible use the ASCII value for space (040).

3. If the command ran successfully it should output nothing, to check the share mounted correctly check mounted directories.

        df

### iv. Unmounting an SMB Share.

1. Identify the directory that needs unmounting.

        df

###### NOTE: This command will unmount normal directories so check your command before running Step 2:.

2. Unmount the SMB Share's mounted directory.

        umount /mnt/ER-10TB-SMB

###### NOTE: This command is NOT -UNMOUNT- it is actually -UMOUNT- with no "N".

## 3- NFS Remote Shared Drive Setup

This part of the guide is for mounting shared drives from a NAS or server using NFS protocol. It is NOT for SMB Shares.

You must follow this in order of Section i. ii. iii. 

### i. Setup of NFS Server & Infrastructure

1. First ensure you have a specific network that you want to use for NFS. Ideally this would be a protected VLAN that does not have internet access. This will allow you to have a specific network that is just for NFS traffic, and is also safe from internet related vulnerabilites.

For Example "VLAN 150" for NFS with a 172.10.0.0/24 range.

2. Ensure the network switch is setup to ensure VLAN membership on the ports both the server and the client(s) will be connected to.

###### NOTE: If you have issues later on with connecting to the NFS Share, try pinging the Server IP from Client, or vice versa. If this fails you have misconfigured your switch (ask me how i know...)

3. Make sure the server is setup with your NFS datastore and that datastore has the NFS share enabled. I am using TrueNAS so this is quite simple.

4. Ensure your client (in my case a VM) has a network adapter which is connected to the NFS VLAN. When setting this up, ENSURE it has a static IP (this is for security - see the next step).

5. Ensure you have resticted your NFS share to only the clients that will be using it. NFS does not use credentials, so if this is not set, then anyone on the NFS VLAN will be able to access it (which is why we want the traffic on an internet blocked VLAN!). 

For me in TrueNAS, I just added my VM's IP address to the "Hosts" tab. That means only that VM can access the share.

### ii. Installation of NFS Client & Manual Mounting of an NFS Share

1. Install NFS Client from the package manager, this will allow us to mount our NFS share.

        sudo apt update
        sudo apt install nfs-common

2. Create a directory in /mnt to use as the mount point for the NFS server folder.

        sudo mkdir /mnt/ER-10TB-NFS

###### NOTE: You can call this anything, but if you are mounting multiple NFS Shares you MUST create seperate folders for each mount.

3. Manually mount the NFS share to test the connection

        sudo mount -t nfs 172.10.0.50:/mnt/Seagate-8TB-HDD/NFS_SHARE /mnt/ER-4TB-NFS/

(Breakdown)                
                
        sudo mount -t nfs [NFS Server IP Address]:[NFS Mount Path] /mnt/[Specific mount directory]/

###### NOTE: This will create a non-persisting mount. Follow Section iii. to create a persistent mount.

###### NOTE 2: you must copy the full mount path from the server, ie. /mnt/Seagate-HDD/NFS_SHARE - Not just the /NFS_Share

4. If all steps above were followed correctly then the SMB Share should have mounted correctly. To verify this, check the mounted directories.

        df

5. Verify the server is setup correctly by creating a test folder on the server from the host machine.

        sudo mkdir /mnt/ER-4TB-NFS/

###### NOTE: If this comes up with a permission denied error, check the permissions on the remote server.

### iii. Automatic Mounting of an NFS Share

Assuming you have successfully mounted the NFS Share using the information above, you now need to set the share to automatically mount on system boot using fstab.

1. Edit fstab.

        sudo nano /etc/fstab

2. Add the path to your SMB server as well as the path to the mount folder created earlier as shown below.

        172.10.0.50:/mnt/Seagate-8TB-HDD/NFS_SHARE /mnt/ER-4TB-NFS/  nfs      defaults    0       0

###### NOTE: functions are seperated with TABS not with SPACES. Spaces will cause formatting errors.

(Breakdown)

        <file system>                                   <dir>      <type>    <options>  <dump>	<pass>

3. After saving, verify the formatting of the new command in fstab is correct. The command should return no output if it has run successfully.

        sudo mount -a

###### NOTE: If the command returns a formatting error, go back and run Step 2: again. Most importantly check for hidden spaces.

4. Check NFS share is mounted by listing the mounted directories.

        df

5. Reboot server to test automatic mounting of the share.

        sudo reboot.

6. If all goes well you may see the NFS Share mount during system boot. It will show up as "[   OK   ] Mounted /mnt/ER-10TB-NFS. - If you don't see it, after the system has booted check the mounted directories.

        df

The share will now automatically mount on boot. Ensure you have a static IP set as per Section i. Step 4. There are no credentials, a system has authority to access a share based upon its IP address. If that changes you will be disallowed from accessing the share!

## Credits

- [Cover Photo](https://cf-images.dustin.eu/cdn-cgi/image/fit=contain,format=auto,width=828/image/d200001004166267/dell-storage-scv2020.jpg)

- [CIFS FSTab](https://help.ubuntu.com/community/MountCifsFstab)

- [Mounting CIFS FSTAB Securely](https://help.ubuntu.com/community/MountCifsFstabSecurely).

- [Fixing Mount Permission issues with docker](https://community.bigbeartechworld.com/t/solving-docker-container-permission-issues-on-mounted-volumes/215)

- [NFS Share Mounting Linux](https://linuxize.com/post/how-to-mount-an-nfs-share-in-linux/)
