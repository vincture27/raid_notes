https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-ubuntu-22-04

RAID 01 (pay attention to size)

Create two RAID 1 arrays using the raw storage devices:

  --------> sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda /dev/sdb
  
  --------> sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdc /dev/sdd

Create a RAID 0 array using the RAID 1 arrays as members:

  --------> sudo mdadm --create --verbose /dev/md2 --level=0 --raid-devices=2 /dev/md0 /dev/md1

Create a filesystem on the RAID 01 array (/dev/md2):

  --------> sudo mkfs.ext4 -F /dev/md2

Create a mount point and mount the RAID 01 array:

  --------> sudo mkdir -p /mnt/md2
  
  --------> sudo mount /dev/md2 /mnt/md2

Save the array layout and update the initramfs:

  --------> sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
  
  --------> sudo update-initramfs -u

Add the new filesystem mount options to the /etc/fstab file for automatic mounting at boot:

  --------> echo '/dev/md2 /mnt/md2 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab

Now you have a RAID 01 setup instead of RAID 10. The rest of the tutorial can be followed as is.
