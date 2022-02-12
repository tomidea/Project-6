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
