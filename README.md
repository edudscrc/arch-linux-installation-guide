<ol>
<li>Download Arch Linux ISO</li>
<li>Burn it to a USB flash</li>
<li>Boot it (make sure <b>secure boot</b> is disabled).</li>
</ol>

### Step 01: Syncronize packages

1. [Optional] Connect to WiFi using `iwctl` and check connection is established:

<dl><dd>
<pre>
$ <b>iwctl</b>
[iwd]# <b>station wlan0 get-networks</b>
[iwd]# <b>station wlan0 connect &lt;Name of WiFi access point&gt;</b>
[iwd]# <b>exit</b>
$ <b>ping google.com</b>
</pre>
</dd></dl>

<b>On ethernet, you don't have to do anything</b>

2. Syncronize pacman packaes:

<dl><dd>
<pre>
$ <b>pacman -Syy</b>
</pre>
</dd></dl>

### Step 02: Disk partitioning

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

<b>Obs.: if you have intel CPU, instead of amd-ucode, install intel-ucode</b>

<dl><dd>
<pre>
$ <b>pacstrap -i /mnt base linux linux-firmware sudo nano amd-ucode</b>
$ <b>genfstab -U -p /mnt > /mnt/etc/fstab</b>
</pre>
</dd></dl>

### Step 03: Basic configuration of new system

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

<b><i>yourhostname</i> is like the name of your machine. It's the name that comes after the @ (youruser@yourhostname)</b>

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

### Step 04: Basic configuration of userspace

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
$ <b>sudo pacman -S dbus base-devel git zip unzip btop wget openssh bash-completion fuse2</b>
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

5. Enable multilib (this allows installing 32-bits packages that are essential for some apps to run):

<dl><dd>
Uncomment the [multilib] section and update pacman
<pre>
$ <b>sudo nano /etc/pacman.conf</b>
$ <b>sudo pacman -Sy</b>
</pre>
</dd></dl>

6. Install sound drivers and sound support (pipewire):

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

9. Install AMD and vulkan drivers (only needed for AMD, idk about nvidia/intel):

<dl><dd>
<pre>
$ <b>pacman -S mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon libva-mesa-driver libva-utils xf86-video-amdgpu</b>
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

### Install pwvucontrol (AUR) to control sound. (This is pavucontrol for pipewire (pw), but I think pavucontrol works too)

<dl><dd>
<pre>
$ <b>yay -S pwvucontrol</b>
</pre>
</dd></dl>

### Hyprland (and must-have software / recommended apps)

<dl><dd>
<pre>
$ <b>sudo pacman -S hyprland</b>
$ <b>sudo pacman -S swaync xdg-desktop-portal-hyprland qt5-wayland qt6-wayland</b>
$ <b>yay -S hyprpolkitagent</b>
$ <b>sudo pacman -S waybar rofi alacritty hyprlock hyprpaper cliphist thunar nwg-look</b>
</pre>
</dd></dl>

Buscar dotfiles

### Steam

<dl><dd>
<pre>
$ <b>sudo pacman -S ttf-liberation lib32-systemd </b>
Em alguns jogos com anticheat (elden ring, por exemplo), precisa colocar isso nos parâmetros de launch:
<b>env --unset=SDL_VIDEODRIVER %command%</b>
</pre>
</dd></dl>

### Discord

<dl><dd>
<pre>
Instale o Vencord e aplique o tema: https://github.com/refact0r/system24/blob/main/theme/flavors/catppuccin-mocha.theme.css
</pre>
</dd></dl>

### GTK Theme

<dl><dd>
<pre>
Instale o seguinte tema e aplique usando o nwg-look
https://github.com/catppuccin/gtk
https://github.com/catppuccin/papirus-folders
</pre>
</dd></dl>

### Firefox, alacritty, vscode, btop...

Todos esses apps tem temas do catppucin no github ofical deles. Apenas buscar e instalar

### Bug onde electron apps não abrem (vscode, discord, spotify, etc...)

Abre o nwg-look, vai na seção "Mouse cursor" e apenas aplique qualquer tema.

### Ubisoft Connect - Problema de conexão (testado pelo heroic games launcher)

<dl><dd>
<pre>
$ <b>sudo pacman -S lib32-gnutls </b>
</pre>
</dd></dl>
