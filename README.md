# Arch Linux + KDE Plasma Installation Guide (Dualboot)
All credits to [edudscrc](https://github.com/edudscrc)

Learn how to install Arch Linux with Hyprland. Minimal installation.

<ol>
<li>Download Arch Linux ISO</li>
<li>leave an empty partition in Windows, with adequate size (>= 100GB)
</li>
<li>Burn it to a USB flash</li>
<li>Boot it (make sure <b>secure boot</b> is disabled).</li>
</ol>

### [Optional] Connect to Wi-Fi (on Ethernet, you don't have to do this):
<pre>
  $ iwctl
  [iwd]# station wlan0 get-networks
  [iwd]# station wlan0 connect <Network's name>
  [iwd]# exit
  $ ping archlinux.org
</pre>

### Synchronize packages:
<pre>
  $ pacman -Syy
</pre>

### Find your main disk partition:
<pre>
  $ lsblk
</pre>

### Disk partitioning (using cfdisk):
<pre>
  $ cfdisk /dev/nvme0n1
  <i>[First, find the empty partition, and then create the following partitions in thios space]</i>
  <i>[create partition 1: <b>efi</b>]</i>
  Partition Size: <b>1024M</b>

  <i>[create partition 2: <b>swap</b>]</i>
  Partition Size: <b>16G</b>

  <i>[create partition 3: <b>/</b>]</i>
  Partition Size: <b>Enter</b>

  <i>[Change the types of each partition]</i>
    - efi: EFI System</li>
    - swap: Linux swap</li>
    - root: Linux filesystem</li>

  <i>[Write changes to disk]</i>
</pre>

### Create filesystems on created disk partitions:
<pre>
  $ mkfs.fat -F 32 /dev/nvme0n1p1
  $ mkswap /dev/nvme0n1p2
  $ mkfs -t ext4 /dev/nvme0n1p3
</pre>

### Mount all filesystems to /mnt:
<pre>
  $ mount /dev/nvme0n1p3 /mnt
  $ mkdir -p /mnt/boot/efi
  $ mount /dev/nvme0n1p1 /mnt/boot/efi
  $ swapon /dev/nvme0n1p2
</pre>

### Install essential packages into new filesystem and generate fstab:
<pre>
  <i>[install amd-ucode for AMD chipset or intel-ucode for INTEL chipset]</i>
  $ pacstrap /mnt base linux linux-firmware amd-ucode base-devel grub efibootmgr nano networkmanager ntfs-3g fuse3 os-prober
  $ genfstab -U /mnt > /mnt/etc/fstab
</pre>

### Mount Main Windows file partition in /mnt/windows :
<pre>
  $ mkdir -p /mnt/windows
  $ mount /dev/[windows_partition] /mnt/windows
  <i>[Check if the partition is mounted with <b>lsblk</b>] </i>
</pre>

### Basic configuration of new system:
<pre>
  $ arch-chroot /mnt
  <i>[uncomment your locales, i.e. 'en_US.UTF-8' or 'pt_BR.UTF-8']</i>
  $ nano /etc/locale.gen
  $ locale-gen
  $ echo "LANG=en_US.UTF-8" > /etc/locale.conf
  $ ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
  $ hwclock --systohc
</pre>

### Setup hostname and usernames:
<pre>
  $ echo <i>yourhostname</i> > /etc/hostname
  $ nano /etc/hosts
      <i>127.0.0.1 localhost</i>
      <i>::1 localhost</i>
      <i>127.0.1.1 yourhostname</i>
  $ useradd -m -G wheel,storage,power,audio,video,uucp -s /bin/bash yourusername
  $ passwd root
  $ passwd yourusername
  $ EDITOR=nano visudo
      <i>[uncomment following line]</i>
      <i>%wheel ALL=(ALL) ALL</i>
</pre>

### Enable networking:
<pre>
  $ systemctl enable NetworkManager
</pre>

### Install and setup GRUB:
<pre>
  $ nano /etc/defaults/grub
    <i>[uncomment following line]</i>
      <i>GRUB_DISABLE_OS_PROBER=false</i>
  $ os-prober
  $ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --removable
  $ grub-mkconfig -o /boot/grub/grub.cfg
</pre>

### Exit chroot, unmount all disks and reboot:
<pre>
  $ exit
  $ umount -R /mnt
  $ reboot
</pre>

<h1 align="center">
    Setup Userspace and KDE
</h1>

### Activate time synchronization using NTP:
<pre>
  $ timedatectl set-ntp true
</pre>

### [Optional] Connect to Wi-Fi using nmcli:
<pre>
  $ nmcli device wifi connect &lt;Network's name&gt; password &lt;Network's password&gt;
</pre>

### Enable multilib (32-bit packages):
<pre>
  $ sudo nano /etc/pacman.conf
  <i>[uncomment [multilib] section]</i>
  $ sudo pacman -Sy
</pre>

### Install GPU drivers (and packages for gaming):
<b>AMD</b>
<pre>
  $ sudo pacman -S xf86-video-amdgpu mesa lib32-mesa
  $ sudo pacman -S vulkan-radeon lib32-vulkan-radeon
</pre>

<b>NVIDIA</b>
<pre>
  $ sudo pacman -S nvidia nvidia-utils lib32-nvidia-utils
</pre>

### [Optional] Run service that will discard unused blocks on mounted filesystems. This is useful for SSDs and thinly-provisioned storage:
<pre>
  $ sudo systemctl enable fstrim.timer
</pre>

### Install useful packages:
<pre>
  $ sudo pacman -S git btop wget fd curl unzip
  $ sudo pacman -S bash-completion openssh eza
  $ sudo pacman -S python python-gobject
  $ sudo pacman -S ripgrep fuse2
  curl -fsSL https://pyenv.run | bash
</pre>

### Install yay:
<pre>
  $ git clone https://aur.archlinux.org/yay.git
  $ cd yay
  $ makepkg -si
  $ cd ..
  $ rm -r yay
</pre>

### Install KDE Plasma and some useful packages:
<pre>
  $ sudo pacman -S xorg plasma plasma-wayland-session kde-applications 
  <i>[bluedevil is the KDE's Bluetooth tool]</i>
  $ sudo pacman -S bluedevil
  $ sudo systemctl enable bluetooth.service
  $ sudo systemctl start bluetooth.service
  $ systemctl enable sddm.service
</pre>

Install yay and a fast image viewer:
<pre>
  $ git clone https://aur.archlinux.org/yay.git
  $ cd yay
  $ makepkg -si
  $ cd ..
  $ rm -r yay
</pre>

### 19. Reboot:
<pre>
  $ reboot
</pre>