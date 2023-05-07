https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-ubuntu-22-04
To modify the tutorial for creating a RAID 01 setup, you will need to create two RAID 1 arrays first and then create a RAID 0 array using those RAID 1 arrays as members. Here's the adjusted process:

1. Create two RAID 1 arrays using the raw storage devices:

```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda /dev/sdb
sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdc /dev/sdd
```

2. Create a RAID 0 array using the RAID 1 arrays as members:

```bash
sudo mdadm --create --verbose /dev/md2 --level=0 --raid-devices=2 /dev/md0 /dev/md1
```

3. Create a filesystem on the RAID 01 array (/dev/md2):

```bash
sudo mkfs.ext4 -F /dev/md2
```

4. Create a mount point and mount the RAID 01 array:

```bash
sudo mkdir -p /mnt/md2
sudo mount /dev/md2 /mnt/md2
```

5. Save the array layout and update the initramfs:

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

6. Add the new filesystem mount options to the /etc/fstab file for automatic mounting at boot:

```bash
echo '/dev/md2 /mnt/md2 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```

Now you have a RAID 01 setup instead of RAID 10. The rest of the tutorial can be followed as is.

_______________________________________________________________________________________________________________
**NFS Mount setup**

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-22-04
_____________________________________________________
Prerequisites:

1. Create the second virtual machine in VirtualBox:
   - Click on "New" and provide a name for the VM, choose "Linux" as the Type, and "Ubuntu (64-bit)" as the Version.
   - Allocate memory (RAM) according to your system's capacity; 1 GB or more is recommended for Ubuntu Server.
   - Choose "Create a virtual hard disk now" and click on "Create."
   - Select "VDI" as the hard disk file type and click "Next."
   - Choose "Dynamically allocated" and click "Next."
   - Set the virtual hard disk size (e.g., 20 GB) and click "Create."

2. Install Ubuntu 22.04 Server on both virtual machines:
   - Start each VM and, when prompted, select the downloaded Ubuntu Server 22.04 ISO file as the startup disk.
   - Follow the installation process, providing information such as language, keyboard layout, and network settings.
   - When prompted to create a user, create a non-root user with a strong password.
   - Complete the installation process and reboot the VMs.

3. Log in to both virtual machines using the non-root user you created during the installation.

4. Update the package index and upgrade the packages on both VMs:

```
sudo apt update
sudo apt upgrade
```

5. Set up UFW (Uncomplicated Firewall) on both VMs:

```
sudo apt install ufw
```

Configure UFW to allow SSH, NFS, and other required services:

```
sudo ufw allow ssh
sudo ufw allow 2049
```

Enable UFW:

```
sudo ufw enable
```

6. Set up private networking (if available) or use the host-only network in VirtualBox:
   - In VirtualBox, go to "File" > "Host Network Manager" and click on "Create."
   - Edit the settings for the new host-only network by clicking on the "Properties" button.
   - Configure the IPv4 address and mask (e.g., 192.168.56.1 and 255.255.255.0) and enable the DHCP server.
   - Save the changes and close the "Host Network Manager."
   - On each VM, go to "Settings" > "Network" > "Adapter 2," enable it, and choose "Host-only Adapter" with the newly created host-only network.
   - Save the changes and restart the VMs.

7. Update the network configuration on both VMs to use the host-only network:
   - Edit the `/etc/netplan/00-installer-config.yaml` file:

```
sudo nano /etc/netplan/00-installer-config.yaml
```

   - Add the configuration for the host-only network, like:

```
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
  version: 2
```

   - Save the changes and apply the new network configuration:

```
sudo netplan apply
```

Now you have two Ubuntu 22.04 virtual machines with non-root users, UFW firewall, and private networking (using the host-only network in VirtualBox). You can proceed to set up the NFS server and client as described in my previous response.

sudo ufw status verbose
ip addr show
__________________________________________________________________________
https://www.howtoforge.com/tutorial/how-to-setup-iscsi-storage-server-on-ubuntu-2004-lts

Here's a brief step-by-step guide for setting up the prerequisites for your iSCSI target and initiator VMs:

1. **Configure static IP addresses:**
   - iSCSI target VM:
     - Open the `/etc/netplan/00-installer-config.yaml` file:
       ```
       sudo nano /etc/netplan/00-installer-config.yaml
       ```
     - Modify the file to configure the static IP address `192.168.1.10`:
       ```yaml
       network:
         version: 2
         ethernets:
           enp0s3:
             dhcp4: no
             addresses: [192.168.1.10/24]
             gateway4: 192.168.1.1
             nameservers:
               addresses: [8.8.8.8, 8.8.4.4]
       ```
     - Save and exit the file, then apply the changes:
       ```
       sudo netplan apply
       ```
   - iSCSI initiator VM:
     - Follow the same steps, but set the IP address to `192.168.1.20` in the configuration file.

2. **Configure root password:**
   - On both the iSCSI target and initiator VMs, set the root password by running:
     ```
     sudo passwd root
     ```
   - Enter the desired password when prompted and confirm it.

3. **Create a 1GB external HDD for the iSCSI target VM:**
   - In VirtualBox, select the iSCSI target VM, and click "Settings."
   - Go to the "Storage" tab and click the "+" icon to add a new storage device.
   - Choose "Create a new disk" and select "VDI (VirtualBox Disk Image)" as the format.
   - Set the size to 1 GB and click "Create."

After completing these steps, you'll have two Ubuntu 20.04 virtual machines configured with the required prerequisites. You can now proceed with the iSCSI target and initiator configuration.
