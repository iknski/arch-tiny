## <b style="color:DodgerBlue">Установка ArchLinux шаг за шагом (KDE)</b>

---
><span style="color:FireBrick">Для автоматической установки выполните <kbd>`git clone`</kbd> данного репозитория<br></span>
><span style="color:FireBrick">Далее запустите <kbd>`arch-minimal`</kbd> и  следуйте подсказкам.<br></span>

Все что написано ниже, лишь пример для вашего понимания

<br>

### <b style="color:Gold">Первоначально:</b><br>

---
|что выполнить|что мы делаем|
|:---|:---|
|<kbd>`timedatectl set-ntp true`</kbd>|актуализируем системное время|
|<kbd>`timedatectl set-timezone <Region/City>`</kbd>|настраиваем временную зону (`Europe/Moscow` или взять нужную из вывода `timedatectl list-timezones`)|
|<kbd>`timedatectl status`</kbd>|проверяем статус синхронизации|
|<kbd>`pacman -Sy --noconfirm archlinux-keyring`</kbd>|установка ключей...|
|...|...|
|Вариант обновления ключей|Может занять пордолжительное время|
|<kbd>`pacman-key --init`</kbd>|инициализация ключей|
|<kbd>`pacman-key --populate archlinux`</kbd>|получение ключей из репозитория|
|<kbd>`pacman-key --refresh-keys`</kbd>|обновить ключи для системы|
|...|...|
|<kbd>`pacman -Syy`</kbd>|синхронизируем и обновляем пакетные базы|
|<kbd>`pacman -S reflector`</kbd>|установливаем [Reflector](https://wiki.archlinux.org/title/Reflector_(Русский)) чтобы актуализировать [зеркала](https://archlinux.org/mirrors/status/) доступные для скачивания|
|<kbd>`reflector --verbose --latest 10 --sort rate --save /etc/pacman.d/mirrorlist`</kbd>|отсортируем зеркала по новизне или скорости|
|<kbd>`pacman -Syy`</kbd>|синхронизируем и обновляем пакетные базы|
|||

<br>

### <b style="color:Gold">Подготовка диска для установки:</b>

---
<span style="color:MediumSpringGreen">Просмотр списка дисков:</span>

```
lsblk
```

><span style="color:FireBrick">_Результаты, оканчивающиеся на <kbd>`rom`, `loop`, `airoot`</kbd> можно игнорировать ..._</span>

<span style="color:MediumSpringGreen">Очистка файловой системы выбранного диска (вместо `sda` введите наименование вашего диска):</span>

```
wipefs --all /dev/sda
```

<span style="color:MediumSpringGreen">Создание файловой системы на выбранном диске:</span><br>

><span style="color:FireBrick">_Будем использовать утилиту [cfdisk](https://man.archlinux.org/man/cfdisk.8) имеющую текстовый пользовательский интерфейс_</span><br>
><span style="color:FireBrick">_Более расширенный функционал имеет утилита [fdisk](https://wiki.archlinux.org/title/Fdisk_(Русский)) которая входит в состав пакета `util-linux`_</span>

```
cfdisk /dev/sda
```

<span style="color:MediumSpringGreen">Тип таблицы разделов выбираем в зависимости от поддержки вашего железа:</span>

><span style="color:FireBrick">_<kbd>`gpt`</kbd> -- для UEFI (новые компьютеры - UEFI API)_</span><br>
><span style="color:FireBrick">_<kbd>`dos`</kbd> -- для MBR (старые компьютеры - BIOS)_</span><br>

<span style="color:MediumSpringGreen">Создаём таблицу разделов:</span>

|тип ФС|размер|
|:---|:---|
|EFI System|512M|
|SWAP|Этот раздел должен быть равен размеру вашей оперативной памяти (можем его не создавать, а создать своп-файл)|
|Linux filesystem (root)|Остальное пространство диска|
|||

><span style="color:FireBrick"><kbd>_`lsblk`</kbd> -- проверим результаты создания разделов..._</span>

<br>

---
><span style="color:DodgerBlue">_Перед тем как форматировать созданные разделы:_</span><br>
><span style="color:DodgerBlue">_В репозитории есть скрипт автоматического форматирования и монтирования для файловой системы BTRFS - `btrfs-helper`_</span><br>
><span style="color:DodgerBlue">_тогда все, что ниже можно пропустить и после работы скрипта уже приступить к установке системы_</span><br>
>
>```
>bash ~/arch-tiny/btrfs-helper
>```
>
><span style="color:DodgerBlue">_Но если вы решили все сделать вручную ..._</span><br>

---

<br>

<span style="color:MediumSpringGreen">Форматирование и монтирование созданных разделов:</span>

<span style="color:FireBrick"><kbd>_`mkfs.vfat -F32 -n BOOT /dev/sda1`</kbd> -- форматирование раздела EFI System с присвоением ему метки тома `BOOT`_</span>

<br>

---

<span style="color:DeepPink">Если хотим создать `swap-раздел`:</span>

<ul><span style="color:DeepPink">> создаём ...</span></ul>

<ul><ul><span style="color:DeepPink">>> включаем ...</span></ul>

</ul>

```
mkswap /dev/sda2
---
swapon /dev/sda2
```

<span style="color:DeepPink">!!! если вы создали `swap-раздел`, то следующий раздел будет уже `/dev/sda3` !!!</span>

<span style="color:DeepPink">Если будем в дальнейшем создавать `swap-файл`, пропускаем эту часть</span>

---

<br>
<br>
<br>

<span style="color:MediumSpringGreen">Остальное пространство отформатируем в файловую систему `btrfs`</span>

* [btrfs - официальная документация](https://btrfs.readthedocs.io/en/latest/index.html)

* [btrfs для самых маленьких](https://habr.com/ru/company/veeam/blog/458250/)

<span style="color:FireBrick"><kbd>_`mkfs.btrfs -L ROOT /dev/sda2`</kbd> -- форматирование раздела Linux filesystem с присвоением ему метки тома `ROOT`_</span>

<span style="color:MediumSpringGreen">> Монтируем корневой раздел

<ul><span style="color:MediumSpringGreen">>> Создаём подтома (<span style="color:Tomato"> subvolume </span>)</ul>

<ul><ul><span style="color:MediumSpringGreen">>>> Размонтируем корневой раздел</span></ul>

</ul>

><span style="color:FireBrick">_!!! Обратите внимание, что сабволум `<snapshots>` идёт без точки. Но каталог для него будет создаваться с точкой `<.snapshots>` !!!_</span>

```
mount /dev/sda2 /mnt
---
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg
btrfs subvolume create /mnt/@snapshots
---
umount -R /mnt
```

<span style="color:MediumSpringGreen">Монтируем корневой подтом:</span>

```
mount -o rw,noatime,compress=zstd:3,space_cache=v2,discard=async,ssd,ssd_spread,max_inline=256,nodatacow,subvol=@ /dev/sda2 /mnt
```

<span style="color:MediumSpringGreen">Создаем каталоги для подтомов (<span style="color:Tomato"> не забываем про точку </span>) и проверяем то, что создали:</span>

```
mkdir -p /mnt/{boot/efi,home,var/log,var/cache/pacman/pkg,.snapshots}
---
ls -la --color /mnt
```

<span style="color:MediumSpringGreen">- Монтируем разделы в подтома</span>

<ul><span style="color:MediumSpringGreen">-- Монтриуем <span style="color:Tomato">boot</span> раздел</span></ul>

<ul><ul><span style="color:MediumSpringGreen">--- проверяем результат:</span></ul>

</ul>

```
mount -o rw,noatime,compress=zstd:3,space_cache=v2,discard=async,ssd,ssd_spread,max_inline=256,nodatacow,subvol=@home /dev/sda2 /mnt/home
mount -o rw,noatime,compress=zstd:3,space_cache=v2,discard=async,ssd,ssd_spread,max_inline=256,nodatacow,subvol=@log /dev/sda2 /mnt/var/log
mount -o rw,noatime,compress=zstd:3,space_cache=v2,discard=async,ssd,ssd_spread,max_inline=256,nodatacow,subvol=@pkg /dev/sda2 /mnt/var/cache/pacman/pkg
mount -o rw,noatime,compress=zstd:3,space_cache=v2,discard=async,ssd,ssd_spread,max_inline=256,nodatacow,subvol=@snapshots /dev/sda2 /mnt/.snapshots
---
mount /dev/sda1 /mnt/boot/efi
---
df -h
```

---
<br>

### <b style="color:Gold">Переходим к установке операционной системы:</b>

```
pacstrap -K /mnt base base-devel linux-zen linux-firmware nano
```

<span style="color:MediumSpringGreen">После установки нужно сгенеририровать таблицу файловой системы ( `fstab` ) и проверим результат:</span>
><span style="color:FireBrick">_на этом этапе можно создать дополнительные папки в каталоге `/mnt` и примонтировать к ним нужные вам диски_</span>
><span style="color:FireBrick">_они будут автоматически монтироваться при загрузке системы_</span>

```
genfstab -U /mnt >> /mnt/etc/fstab
---
cat /mnt/etc/fstab
```

---
<br>

### <b style="color:Gold">Настройка системы:</b>

<span style="color:MediumSpringGreen">Перейдём в корневой каталог установленной системы:</span>

```
arch-chroot /mnt
```

<span style="color:MediumSpringGreen">Если не создавали `swap`-раздел, создадим `swap`-файл и внесем данные о своп-файле в файл `fstab` в конец строки</span>

```
touch /swapfile
chattr +C /swapfile
lsattr /swapfile  # ---------------c------
fallocate -l 32GB /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon
---
nano /etc/fstab
---
/swapfile none swap defaults 0 0
```

<span style="color:MediumSpringGreen">Зададим часовой пояс (<span style="color:Tomato"> ... /Europe/Moscow </span>):</span>

```
ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime
```

<span style="color:MediumSpringGreen">Учтановим системное время в соостветствии с аппаратными часами:</span>

```
hwclock --systohc
```

<span style="color:MediumSpringGreen">В файле `locale.gen` раскоментируем нужные локали: `en_US.UTF-8 UTF-8`, `ru_RU.UTF-8 UTF-8`</span>

```
nano /etc/locale.gen
```

<span style="color:MediumSpringGreen">Настроим язык системы:</span>

```
nano /etc/locale.conf
---
LANG=ru_RU.UTF-8
```

<span style="color:Tomato">или сразу одной строкой...</span>

```
echo "LANG=ru_RU.UTF-8" >> /etc/locale.conf
```

<span style="color:MediumSpringGreen">Настроим русский язык для правильного отображения языка в консоли:</span>

```
nano /etc/vconsole.conf
---
KEYMAP=ru
FONT=cyr-sun16
```

<span style="color:MediumSpringGreen">Сгенерируем локали:</span>

```
locale-gen
```

<span style="color:MediumSpringGreen">Зададим имя компьютеру в сети:</span>

```
nano /etc/hostname
```

<span style="color:Tomato">или сразу одной строкой...</span>

```
echo PC_NAME >> /etc/hostname
```

<span style="color:MediumSpringGreen">Настроим файл `hosts`:</span>

```
nano /etc/hosts
---
127.0.0.1    localhost
::1    localhost
127.0.1.1    PC_NAME.localdomain    PC_NAME
```

<span style="color:MediumSpringGreen">Настроим пароль для суперпользователя ( `root` ):</span>

```
passwd
```

<span style="color:MediumSpringGreen">Установим нужные пакеты:</span><br>

<br>

><span style="color:DodgerBlue">Есть хороший способ ускорить процесс путём установки пакетов из заранее заготовленного списка ...</span><br>
><span style="color:DodgerBlue">В репозитории есть списки созданные мной для окружения KDE, но лучшая идея исходить из собственного оборудования и предпочтений по софту ...</span>
>
>```
>pacman -S $(cat /путь_к_вашему_списку)
>```
>
><span style="color:DodgerBlue">Как должен выглядеть заранее заготовленный список ...</span>
>
>```
>grub
>efigrub
>dhcpcd
>dhclient
> .. итд итп
>```

<br>

<span style="color:FireBrick">_!!! для процессоров `intel` вместо `amd-ucode` нужно установить `intel-ucode` и `iucode-tool` !!!_</span> [<<<подробно о Microcode>>>](https://wiki.archlinux.org/title/Microcode_(Русский))

```
pacman -S dkms grub grub-btrfs efibootmgr efitools dkms base-devel linux-zen-headers amd-ucode xdg-utils xdg-user-dirs bash-completion dhcpcd dhclient networkmanager network-manager-applet iwd wireless_tools wpa_supplicant archlinux-keyring pipewire lib32-pipewire pipewire-alsa pipewire-jack pipewire-pulse gst-plugin-pipewire pipewire-audio libpulse wireplumber reflector rsync os-prober dialog pacman-contrib sudo polkit git openssh cups cups-pdf system-config-printer print-manager python-pip bluez bluez-utils smartmontools dosfstools e2fsprogs exfatprogs ntfs-3g mtools btrfs-progs mtpfs usbutils android-tools android-udev ark arj lzop lrzip unrar unzip unace p7zip squashfs-tools neovim python-pynvim man-db man-pages neofetch htop imagemagick ranger cmus links firefox hunspell hunspell-ru
```

<span style="color:MediumSpringGreen">Установим загрузчик системы:</span>

<br>

<span style="color:Fuchsia">Для `GRUB`:</span>

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchLinux
grub-mkconfig -o /boot/grub/grub.cfg
```

<span style="color:Fuchsia">Настройка `GRUB` в файле `/etc/default/grub`:</span>

```
GRUB_CMDLINE_LINUX_DEFAULT="rw quiet splash loglevel=6 nowatchdog rootfstype=btrfs rootflags=subvol=@ nvidia-drm.modeset=1"
```

---

<br>
<br>
<br>

<span style="color:Fuchsia">Для `SYSTEMD-BOOT`:</span>

```
bootctl --path=/boot install
```

<span style="color:Cyan">Настроим загрузчик системы отредактировав файл `loader.conf`:</span>

```
cd /boot
ls -la --color
cd loader
ls -la --color
nano loader.conf
```

<span style="color:Cyan">Приведём его к виду:</span>

```
timeout 10
console-mode max
editor no
default arch.conf
```

<span style="color:Cyan">Перейдём в папку `entries` и создадим в ней конфигурационный файл загрузчика:</span>

```
cd entries
ls -la
nano arch.conf
```

<span style="color:Cyan">Запишем во вновь созданный файл:</span><br>

<span style="color:Cyan">Причём, поля помеченные <span style="color:Red"><так></span> возьмём из этого вывода <kbd>`ls /boot -la --color`</kbd></span><br>

> <span style="color:FireBrick">!!! Вместо `amd-ucode` нужно прописать образ для своего процессора, он тоже будет в выводе выше и этот образ должен быть ПЕРВЫМ initrd при загрузке !!!</span>

```
title   Arch Linux
linux   /<vmlinuz-linux>
initrd  /amd-ucode.img
initrd  /<initramfs-linux.img>
options root="LABEL=ROOT" rw rootflags=subvol=@ nowatchdog loglevel=6 rootfstype=btrfs nvidia-drm.modeset=1
```

<span style="color:Cyan">Так же можно в `options root=` прописать <span style="color:Red">UUID</span> корневого раздела ( `root` ). Команда ниже запишет его в конфиг:</span>

```
blkid /dev/sda2 >> arch.conf
```

<span style="color:Cyan">Далее нужно отредактировать конфигурационный файл приведя его к виду:</span>

```
options root="UUID=uuid of the root partition" rw rootflags=subvol=@ nowatchdog loglevel=6 rootfstype=btrfs nvidia-drm.modeset=1
```

<span style="color:Cyan">Теперь нам нужно создать конфигурационный файл для `fallback`:</span>

```
cp arch.conf arch-fb.conf
nano arch-fb.conf
```

<span style="color:Cyan">Допишем куда нужно `fallback` ...</span>

```
title   Arch Linux Fallback
linux   /<vmlinuz-linux>
initrd  /amd-ucode.img
initrd  /<initramfs-linux-fallback.img>
options root="LABEL=ROOT" rw rootflags=subvol=@ nowatchdog loglevel=6 rootfstype=btrfs nvidia-drm.modeset=1
```

<span style="color:Cyan">Если хотим сделать `dual-boot` с `Windows` (<span style="color:Tomato">выбор между двух операционных систем при загрузке</span>):</span>

```
fdisk -l
---
mkdir /mnt/windows
mount /dev/nvme0n1p1 /mnt/windows
cp -r /mnt/windows/EFI/Microsoft /boot/EFI
```

---
<span style="color:MediumSpringGreen">Включим `NetworkManager`:</span>

```
systemctl enable NetworkManager.service
```

<span style="color:MediumSpringGreen">Включим `iwd`:</span>

```
systemctl enable iwd.service
```

<span style="color:MediumSpringGreen">Включим `Bluetooth`:</span>

```
systemctl enable bluetooth.service
```

<span style="color:MediumSpringGreen">Включим `Printers`:</span>

```
systemctl enable cups.service
```

<span style="color:MediumSpringGreen">Запустим `Pipewire`:</span>

```
systemctl --user enable pipewire-pulse.service
```

<span style="color:MediumSpringGreen">Включим [SSD-TRIM](https://wiki.archlinux.org/title/Solid_state_drive):</span>

```
systemctl enable fstrim.timer
```

<span style="color:MediumSpringGreen">Создадим своего пользователя:</span>

```
useradd -mG wheel <user_name>
```

<span style="color:MediumSpringGreen">Настроим для него пароль:</span>

```
passwd <user_name>
```

<span style="color:MediumSpringGreen">Отредактируем файл `sudoers`:</span>

```
nano /etc/sudoers
```

<span style="color:MediumSpringGreen">Раскомментируем в нём строчку:</span>

```
%whell ALL=(ALL:ALL) ALL
```

<span style="color:MediumSpringGreen">Размонтируем диски, перезагрузимся</span>

```
exit
umount -a
reboot
```

---
<br>

<span style="color:MediumSpringGreen">Войдём под своим пользователем:</span>

<span style="color:MediumSpringGreen">Если не подключены по сети запустим `iwd.service` для подключения к wi-fi сети:</span><br>

```
sudo systemctl enable iwd.service
---
sudo nano /etc/iwd/main.conf
---
[General]
EnableNetworkConfiguration=true
[Network]
RoutePriorityOffset=300
EnableIPv6=true
---
sudo dhcpcd
```

<span style="color:MediumSpringGreen">Для установки 32-битных пакетов нужен `Multilib`. Включим его отредактировав файл `pacman.conf`</span>

```
nano /etc/pacman.conf
```

<span style="color:MediumSpringGreen">раскоментируем секцию `[multilib]`</span>

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

<span style="color:MediumSpringGreen">так же раскоментируем и изменим значение:</span>

```
VerbosePkgLists
ParallelDownloads = 10
```

<span style="color:MediumSpringGreen">Обновим пакеты:</span>

```
pacman -Syu
```

<span style="color:MediumSpringGreen">Устанавим драйверы для своей видеокарты ( `данный пример для nvidia` ):</span><br>

<span style="color:MediumSpringGreen">Какие нужно установить смотрим на <kbd><https://wiki.archlinux.org/></kbd></span>

```
sudo pacman -S nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader opencl-nvidia lib32-opencl-nvidia libxnvctrl cuda python-cuda
```

<span style="color:MediumSpringGreen">Устанавим `X Window System`:</span>

```
sudo pacman -S xorg xorg-server xorg-xinit xorg-server-devel xorg-xwayland
```

<span style="color:MediumSpringGreen">Устанавливаем `Display manager` и включим его:</span>

```
sudo pacman -S sddm sddm-kcm
sudo systemctl enable sddm.service
```

<span style="color:MediumSpringGreen">Если хотим обойтись без sddm:</span>

```
nano ~/.xinitrc
---
export DESKTOP_SESSION=plasma
exec startplasma-x11
---
startx
```

<span style="color:MediumSpringGreen">Устанавливаем рабочее окружение ( `это пример для  KDE` ):</span>

```
sudo pacman -S plasma-desktop konsole kate kwrite dolphin dolphin-plugins okular spectacle kcolorchooser kolourpaint krita haruna elisa kcalc kget ktorrent kmix kompare okteta kdevelop-python ktimer kcron partitionmanager kdf ksystemlog plasma-systemmonitor kdeconnect discover flatpak packagekit-qt5 plasma-wayland-session egl-wayland breeze-grub
```

<span style="color:MediumSpringGreen">Добавление важных модулей в образы `initramfs`:</span><br>
<span style="color:FireBrick">!!! если у вас не `nvidia` и не `btrfs`, то не нужно это добавлять ...</span>

```
sudo nano /etc/mkinitcpio.conf
---
MODULES=(crc32c libcrc32c zlib_deflate btrfs) - для файловой системы btrfs
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm) - для nvidia
```

<span style="color:MediumSpringGreen">Установим [YAY](https://wiki.archlinux.org/title/AUR_helpers_(Русский)). Это надстройка над менеджером пакетов `pacman`. Он нам понадобится для удобной установки приложений из [AUR (Arch User Repository)](https://wiki.archlinux.org/title/Arch_User_Repository_(Русский)):</span><br>

<span style="color:MediumSpringGreen">Ниже пример установки timeshift с помощью `yay`. Нужно лишь название пакета в репозиториях `AUR`</span>

```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -sric PKGBUILD
cd
```

<span style="color:MediumSpringGreen">Установим [nvidia-tweaks](https://github.com/ventureoo/nvidia-tweaks):</span>

```
git clone https://aur.archlinux.org/nvidia-tweaks.git
cd nvidia-tweaks
nano PKGBUILD
makepkg -sric PKGBUILD
cd
```

<span style="color:MediumSpringGreen">Установим `mkinitcpio-firmware` чтобы убрать сообщения `WARNING: Possibly missing firmware for module:xxxx`:</span>

```
git clone https://aur.archlinux.org/mkinitcpio-firmware.git
cd mkinitcpio-firmware
makepkg -sric PKGBUILD
cd
```

<span style="color:MediumSpringGreen">Установим [Kcmsystemd](https://github.com/rthomsen/kcmsystemd). Модуль для настройки systemd:</span>

```
yay -S systemd-kcm
```

<span style="color:MediumSpringGreen">Установим [TimeShift](https://wiki.archlinux.org/title/Synchronization_and_backup_programs_(Русский)):</span>

```
yay -S timeshift
```

<span style="color:MediumSpringGreen">Сгенерируем образы `initramfs`:</span>

```
mkinitcpio -P
```

<span style="color:MediumSpringGreen">Закончим настройку и загрузимся в систему:</span>

```
reboot
```

<span style="color:MediumSpringGreen">Настройка `fish`:</span>

```
chsh -s /usr/bin/fish
---
set -U fish_greeting
---
curl -sL https://git.io/fisher | source && fisher install jorgebucaran/fisher
---
fisher install jorgebucaran/nvm.fish
---
fisher install IlanCosman/tide@v5
```

<span style="color:MediumSpringGreen">Для даулбута настроим синхронизацию часов:</span>

```
timedatectl set-local-rtc 1 --adjust-system-clock
```

<span style="color:MediumSpringGreen">Отключение отладочной информации `KDE`:</span>

```
kdebugdialog5
```
