### Step 01: Downloading Arch Linux image

1. Go to Arch Linux downloads page https://archlinux.org/download/

2. Find **HTTP Direct Downloads** section and choose any download mirror.
   Select a mirror that is geographically closer to your location.

3. On the mirror page find archive named like `archlinux-YYYY.MM.DD-x86_64.iso` or `archlinux-x86_64.iso`
   or any other file with `.iso` suffix. Other files (like _.txt_, _.tar.gz_ and even _.iso.sig_)
   are not needed for installation process.

### Step 02: Preparing installation medium

1. Insert a USB-stick into your PC with at least 2Gb of space availavle on it.

2. Find corresponding block device for USB-stick in `/dev` folder. Usually it is `/dev/sdb`.

<dl><dd>
<b>IMPORTANT NOTE</b>: you need block device without a number on the end.
If you have for example <i>/dev/sdb</i>, <i>/dev/sdb1</i> and <i>/dev/sdb2</i> you need <i>/dev/sdb</i> !
</dd></dl>

3. Burn previously downloaded Arch Linux ISO-image on a USB-stick (in my case it is `/dev/sdb`):

<dl><dd>
<pre>
$ <b>sudo dd conv=fsync oflag=direct status=progress \
          if=./archlinux-YYYY.MM.DD-x86_64.iso of=/dev/sdb</b>
</pre>
</dd></dl>

### Step 03: Boot into Arch Linux medium

1. Insert the installation medium into the computer on which you are installing Arch Linux.

2. Power on your PC and press _boot menu key_. For _Lenovo ThinkPad X1 Carbon_ series laptop,
   this key is `F12`.

3. Boot from USB-stick and wait until boot process is finished.

<dl><dd>
<b>IMPORTANT NOTE</b>: not every device can run a system from USB-stick out of the box.
Many BIOS'es by default come with activated <i>Secure boot</i> option.You might need to
deactivate it in your BIOS.
</dd></dl>

### Step 04: Syncronize packages

1. [Optional] Connect to WiFi using `iwctl` and check connection is established:

<dl><dd>
<pre>
$ <b>iwctl</b>
[iwd]# <b>station wlan0 get-networks</b>
[iwd]# <b>station wlan0 connect &lt;Name of WiFi access point&gt;</b>
[iwd]# <b>exit</b>
$ <b>ping 1.1.1.1</b>
</pre>
</dd></dl>

2. Syncronize pacman packaes:

<dl><dd>
<pre>
$ <b>pacman -Syy</b>
</pre>
</dd></dl>

### Step 05: Disk partitioning

1. Partition main storage device using `fdisk` utility. You can find storage device name using `lsblk` command.

<dl><dd>
<pre>
$ <b>fdisk /dev/nvme0n1</b>
                <i>[repeat this command until existing partitions are deleted]</i>
Command (m for help): <b>d</b>
Command (m for help): <b>d</b>
Command (m for help): <b>d</b>
<span />
                <i>[create partition 1: efi]</i>
Command (m for help): <b>n</b>
Partition number (1-128, default 1): <b>Enter &crarr;</b>
First sector (..., default 2048): <b>Enter &crarr;</b>
Last sector ...: <b>+256M</b>
<span />
                <i>[create partition 2: main]</i>
Command (m for help): <b>n</b>
Partition number (2-128, default 2): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>-32G</b> <i>// double size of your RAM</i>
<span />
                <i>[create partition 3: swap]</i>
Command (m for help): <b>n</b>
Partition number (3-128, default 3): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>Enter &crarr;</b>
<span />
                <i>[change partition types]</i>
Command (m for help): <b>t</b>
Partition number (1-3, default 1): <b>1</b>
Partion typr or alias (type L to list all): <b>uefi</b>
Command (m for help): <b>t</b>
Partition number (1-3, default 2): <b>2</b>
Partion typr or alias (type L to list all): <b>linux</b>
Command (m for help): <b>t</b>
Partition number (1-3, default 3): <b>3</b>
Partion typr or alias (type L to list all): <b>swap</b>
<span />
                <i>[write partitioning to disk]</i>
Command (m for help): <b>w</b>
</pre>
</dd></dl>

2. Create filesystems on created disk partitions:

<dl><dd>
<pre>
$ <b>mkfs.fat -F 32 /dev/nvme0n1p1</b> <i># on EFI System partition</i>
$ <b>mkfs -t ext4 /dev/nvme0n1p2</b>   <i># on Linux filesystem partition</i>
$ <b>mkswap /dev/nvme0n1p3</b>         <i># on Linux swap partition</i>
</pre>
</dd></dl>

3. Correctly mount all filesystems to the `/mnt`:

<dl><dd>
<pre>
$ <b>mount /dev/nvme0n1p2 /mnt</b>
$ <b>mkdir -p /mnt/boot/efi</b>
$ <b>mount /dev/nvme0n1p1 /mnt/boot/efi</b>
$ <b>swapon /dev/nvme0n1p3</b>
</pre>
</dd></dl>

4. Install essential packages into new filesystem and generate fstab:

<dl><dd>
<pre>
$ <b>pacstrap -i /mnt base linux linux-firmware sudo nano amd-ucode</b>
$ <b>genfstab -U -p /mnt > /mnt/etc/fstab</b>
</pre>
</dd></dl>

### Step 06: Basic configuration of new system

1. Chroot into freshly created filesystem:

<dl><dd>
<pre>
$ <b>arch-chroot /mnt</b>
</pre>
</dd></dl>

2. Setup system locale and timezone, sync hardware clock with system clock:

<dl><dd>
<pre>
$ <b>vim /etc/locale.gen</b>   <i># uncomment your locales, i.e. `en_US.UTF-8` or `pt_BR.UTF-8`</i>
$ <b>locale-gen</b>
$ <b>echo "LANG=en_US.UTF-8" > /etc/locale.conf</b>                <i># choose your locale</i>
$ <b>ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime</b>   <i># choose your timezone</i>
$ <b>hwclock --systohc</b>
</pre>
</dd></dl>

3. Setup system hostname:

<dl><dd>
<pre>
$ <b>echo <i>yourhostname</i> > /etc/hostname</b>
$ <b>vim /etc/hosts</b>
    <i>127.0.0.1 localhost</i>
    <i>::1       localhost</i>
    <i>127.0.1.1 yourhostname</i>
</pre>
</dd></dl>

4. Add new users and setup passwords:

<dl><dd>
<pre>
$ <b>useradd -m -G wheel,storage,power,audio,video -s /bin/bash yourusername</i></b>
$ <b>passwd root</b>
$ <b>passwd <i>yourusername</i></b>
</pre>
</dd></dl>

5. Add wheel group to sudoers file to allow users to run sudo:

<dl><dd>
<pre>
$ <b>EDITOR=nano visudo</b>
    <i>[uncomment following line in file]</i>
    <i>%wheel ALL=(ALL) ALL</i>
</pre>
</dd></dl>

6. Install and configure GRUB:

<dl><dd>
<pre>
$ <b>pacman -S grub efibootmgr</b>
$ <b>grub-install /dev/nvme0n1</b>
$ <b>grub-mkconfig -o /boot/grub/grub.cfg</b>
</pre>
</dd></dl>

7. Setup networking stack:

<dl><dd>
<pre>
$ <b>pacman -S dhcpcd networkmanager resolvconf</b>
$ <b>systemctl enable dhcpcd</b>
$ <b>systemctl enable NetworkManager</b>
$ <b>systemctl enable systemd-resolved</b>
</pre>
</dd></dl>

8. Exit chroot, unmount all disks and reboot:

<dl><dd>
<pre>
$ <b>exit</b>
$ <b>umount /mnt/boot/efi</b>
$ <b>umount /mnt</b>
$ <b>reboot</b>
</pre>
</dd></dl>

<h1 align="center">
    Section 02: Configuring userspace after initial system setup &#127919;
</h1>

### Step 01: Basic configuration of userspace

1. Activate time syncronization using NTP:

<dl><dd>
<pre>
$ <b>timedatectl set-ntp true</b>
</pre>
</dd></dl>

2. [Optional] Connect to WiFi using `nmcli`:

<dl><dd>
<pre>
$ <b>nmcli device wifi connect &lt;Name of WiFi access point&gt; password &lt;password&gt;</b>
</pre>
</dd></dl>

3. Install a bunch of useful utilities:

<dl><dd>
<pre>
$ <b>sudo pacman -S dbus base-devel git zip unzip btop wget openssh bash-completion</b>
<div><div/>
$ <b>sudo pacman -S curl ripgrep fd gdb python grim slurp tar cmake make</b>
</pre>
</dd></dl>

4. Install essential system fonts:

<dl><dd>
<pre>
$ <b>sudo pacman -S ttf-dejavu ttf-freefont ttf-liberation ttf-droid terminus-font ttf-font-awesome</b>
$ <b>sudo pacman -S noto-fonts noto-fonts-emoji ttf-ubuntu-font-family ttf-roboto ttf-roboto-mono</b>
</pre>
</dd></dl>

5. Enable multilib:

<dl><dd>
Uncomment the [multilib] section and update pacman
<pre>
$ <b>sudo nano /etc/pacman.conf</b>
$ <b>sudo pacman -Sy</b>
</pre>
</dd></dl>

6. Enable sound support on your PC:

<dl><dd>
<pre>
$ <b>sudo pacman -S pipewire wireplumber pipewire-audio pipewire-alsa pipewire-pulse pipewire-jack lib32-pipewire</b>
</pre>
</dd></dl>

7. [Optional] Enable bluetooth support on your PC:

<dl><dd>
<pre>
$ <b>sudo pacman -S bluez bluez-utils blueman</b>
$ <b>sudo systemctl enable bluetooth</b>
</pre>
</dd></dl>

8. [Optional] Run service that will discard unused blocks on mounted filesystems. This is useful for
    solid-state drives (SSDs) and thinly-provisioned storage. More information on fstrim
    [can be found here](https://man7.org/linux/man-pages/man8/fstrim.8.html).

<dl><dd>
<pre>
$ <b>sudo systemctl enable fstrim.timer</b>
</pre>
</dd></dl>

9. Install AMD and vulkan drivers:

<dl><dd>
<pre>
$ <b>pacman -S mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon libva-mesa-driver libva-utils</b>
</pre>
</dd></dl>

10. Reboot to finalize installation:

<dl><dd>
<pre>
$ <b>reboot</b>
</pre>
</dd></dl>

### Install package manager for AUR (Arch User Repository)

<dl><dd>
<pre>
$ <b>git clone https://aur.archlinux.org/yay.git</b>
$ <b>cd yay</b>
$ <b>makepkg -si</b>
</pre>
</dd></dl>

### Install pwvucontrol (AUR)

<dl><dd>
<pre>
$ <b>yay -S pwvucontrol</b>
</pre>
</dd></dl>

### Hyprland

<dl><dd>
<pre>
$ <b>sudo pacman -S hyprland</b>
$ <b>sudo pacman -S swaync xdg-desktop-portal-hyprland qt5-wayland qt6-wayland</b>
$ <b>yay -S hyprpolkitagent</b>
$ <b>sudo pacman -S waybar rofi alacritty hyprlock hyprpaper</b>
</pre>
</dd></dl>

exec-once = waybar & hyprpaper
exec-once = swaync
exec-once = systemctl --user start hyprpolkitagent
