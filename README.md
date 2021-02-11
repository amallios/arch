##### updated for (2021.02) ISO on ASUS-U36SG: #UEFI #GPT #GRUB #KDE #Plasma

# Part 1. Arch Linux Installation

## a. Connect to WiFi and sync network time
iwctl

[iwd]# device list

[iwd]# station DEVICE scan

[iwd]# station DEVICE get-networks

[iwd]# station DEVICE connect SSID

[iwd]# exit

ping -c 3 archlinux.org

timedatectl set-ntp true

timedatectl status

## b. Partition and mount drives
fdisk /dev/sda (create new empty GPT partition table)

cfdisk (sda1: type=efi, size=550M; sda2: type=linux, size=200G; sda3: type=swap, size=32G)

mkswap /dev/sda3

swapon /dev/sda3

mkfs.ext4 /dev/sda2

mount /dev/sda2 /mnt

mkfs.fat -F32 /dev/sda1

mkdir /mnt/boot

mkdir /mnt/boot/efi

mount /dev/sda1 /mnt/boot/efi

## c. Base installation & configuration
pacstrap -i /mnt base base-devel linux linux-firmware nano

genfstab -U -p /mnt >> /mnt/etc/fstab

arch-chroot /mnt

ln -sf /usr/share/zoneinfo/REGION/AREA /etc/localtime

hwclock -w

nano /etc/locale.gen (#en_US.UTF-8)

locale-gen

echo LANG=en_US.UTF-8 > /etc/locale.conf

echo HOSTNAME > /etc/hostname

echo -e "127.0.0.1\tlocalhost\n::1\tlocalhost\n127.0.1.1\tHOSTNAME.localdomain\tHOSTNAME" >> /etc/hosts

passwd

nano /etc/pacman.conf (#[multilib])

useradd -m -g users -G wheel,storage,power -s /bin/bash USERNAME

passwd USERNAME

EDITOR=nano visudo (#%wheelALL=(ALL) ALL)

## d. Bootloader installation & configuration
pacman -Syu && pacman -S grub efibootmgr

grub-install --target=x86_64-efi --efi-directory=/boot/efi

nano /etc/default/grub (GRUB_TIMEOUT=0; GRUB_TIMEOUT_STYLE=hidden)

grub-mkconfig -o /boot/grub/grub.cfg

## e. Desktop Environment (Plasma) installation
pacman -Syu && pacman -S xorg-server plasma dolphin konsole chromium

## f. Further optimisation
#### CPU Cores
(lscpu | grep “CPU(s):” | grep -v NUMA)

grep “COMPRESSXZ=(xz” /etc/makepkg.conf && sudo sed -i ‘s/COMPRESSXZ=(xz -c -z -)/COMPRESSXZ=(xz -c -T 4 -z -)/g’ /etc/makepkg.conf && grep “COMPRESSXZ=(xz” /etc/makepkg.conf

#### Options to sub-MAKE
grep “#MAKEFLAGS=\”-j” /etc/makepkg.conf && sudo sed -i ‘s/#MAKEFLAGS=”-j2"/MAKEFLAGS=”-j9"/g’ /etc/makepkg.conf $$ grep “#MAKEFLAGS=\”-j” /etc/makepkg.conf

#### Colour to pacman
grep “Color” /etc/pacman.conf && sudo sed -i -e ‘s/#Color/Color/g’ /etc/pacman.conf && grep “Color” /etc/pacman.conf

## g. Enable Services & Reboot
systemctl enable NetworkManager sddm

exit

reboot

# Part 2. Install additional packages

## a. Install Pacman Packages
sudo pacman -Syu && sudo pacman -S ark bluez-utils cmatrix geogebra gimp git gpicview htop hunspell jdk-openjdk jre-openjdk kcron kdesdk-thumbnailers kdegraphics-thumbnailers ktorrent libreoffice lynx man-db man-pages mplayer neofetch okular openssh pdfarranger postgresql pulseaudio-alsa pulseaudio-bluetooth pydf reflector spectacle speedtest-cli thunderbird tor virtualbox vlc wget youtube-dl

## b. Generate OpenPGP certificate & fetch keys
gpg --full-gen-key

## c. Make Yay and install AUR packages
git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -sic && cd .. && rm -rf yay

yay -Syu && yay -S atom balena-etcher datagrip datagrip-jre dropbox exfat-utils-nofuse expressvpn google-chrome ifuse nordvpn-bin nvidia-390xx-dkms pycharm-professional realvnc-vnc-server realvnc-vnc-viewer slack-desktop spotify tor-browser webstorm whatsapp-nativefier zoom

## d. Install Anaconda & Exit
wget https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh && sh Anaconda3-2020.11-Linux-x86_64.sh && rm Anaconda3-2020.11-Linux-x86_64.sh

exit

# Part 3. Server Configurations

## a. Samba
smb://pi@192.168.195.136/share

## b. Postgresql
sudo -iu postgres

initdb --locale=en_US.UTF-8 -E UTF8 -D /var/lib/postgres/data

systemctl start postgresql.service

createuser --interactive USERNAME

exit

## c. NordVPN
sudo gpasswd -a USERNAME nordvpn

sudo systemctl enable --now nordvpn

nordvpn login

## d. Dropbox

## e. Enable Services & Reboot
systemctl enable bluetooth postgresql.service 

reboot
