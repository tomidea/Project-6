# LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”

##### Step 1 — Prepare a Web Server
- Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
-img
img

- Attach all three volumes one by one to your Web Server EC2 instance
img

- Use *lsblk* command to inspect what block devices are attached to the server.
img

- Use *df -h* command to see all mounts and free space on your server
img

- Use gdisk utility to create a single partition on each of the 3 disks
*sudo gdisk /dev/xvdf*
img
*sudo gdisk /dev/xvdg*
img
*sudo gdisk /dev/xvdh*
img

- Use *lsblk* utility to view the newly configured partition on each of the 3 disks.
img 

- Install lvm2 package using *sudo yum install lvm2*. 
img
- Run *sudo lvmdiskscan* command to check for available partitions.
img

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
img

- Verify that your Physical volume has been created successfully by running *sudo pvs*
img

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
*sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1*

- Verify that your VG has been created successfully by running *sudo vgs*
img

- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
*sudo lvcreate -n apps-lv -L 14G webdata-vg*
*sudo lvcreate -n logs-lv -L 14G webdata-vg*
img

- Verify that your Logical Volume has been created successfully by running sudo lvs
img

- Verify the entire setup
*sudo vgdisplay -v* #view complete setup - VG, PV, and LV
img

*sudo lsblk*

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem
*sudo mkfs -t ext4 /dev/webdata-vg/apps-lv*
*sudo mkfs -t ext4 /dev/webdata-vg/logs-lv*
img

- Create /var/www/html directory to store website files
*sudo mkdir -p /var/www/html*

- Create /home/recovery/logs to store backup of log data
*sudo mkdir -p /home/recovery/logs*

- Mount /var/www/html on apps-lv logical volume
*sudo mount /dev/webdata-vg/apps-lv /var/www/html/*

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
*sudo rsync -av /var/log/. /home/recovery/logs/*

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. 
*sudo mount /dev/webdata-vg/logs-lv /var/log*

- Restore log files back into /var/log directory
*sudo rsync -av /home/recovery/logs/. /var/log*

- Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file
*sudo blkid*
img

*sudo vi /etc/fstab*
img

- Test the configuration and reload the daemon
 *sudo mount -a*
 *sudo systemctl daemon-reload*
 
 - Verify your setup by running df -h, 
 img
 
 ## Step 2 — Prepare the Database Server
 
