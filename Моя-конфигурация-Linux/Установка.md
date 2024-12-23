# Установка OpenSuse Tumbleweed

## Создание загрузочного USB накопителя

### Установка Live CD образа

```sh
wget https://download.opensuse.org/tumbleweed/iso/openSUSE-Tumbleweed-XFCE-Live-x86_64-Current.iso
```

!!!
Я буду ставить OpenSuse без установщика, то есть через консоль.
!!!

### Сжение(Burn) ISO образа

```sh
sudo dd bs=4M if=/path/to/iso of=/dev/sdX && sync
```

## Процесс установки

### Запуск Live CD

Как и с любой другой операционной системой или дистрибутивом Linux,
мы должны войти в загрузочный образ ISO.

После того как мы это сделаем перед нами появится XFCE рабочий стол.

![Запущенный Live CD](../uploads/opensuse_install_1.png)

После этого требуется лишь открыть браузер с этим руководством и терминал.

### (Опционально) Настройка SSH для доступа к системе

```sh
sudo passwd linux
sudo systemctl start sshd
```

Далее просто используем `ssh linux@x.x.x.x`.

### Обновление Live CD

Я считаю необходимым обновить весь инструментарий Live CD до последних версий.

```sh
sudo zypper refresh
sudo zypper update
```

### Установка локалей

Без настройки локалей будет возникать ряд ошибок во время установки.

```sh
sudo zypper install glibc-locale glibc-i18ndata glibc-gconv-modules-extra
sudo localedef -i en_US -f UTF-8 en_US.UTF-8
```

### Подготовка разделов на жестком диске

Для начала требуется узнать наименование необходимого жесткого диска.
Делается это при помощи команды `lsblk`.

![Запуск команды lsblk](../uploads/opensuse_install_2.png)

Далее при помощи утилиты `cfdisk` необходимо настроить разделы на данном диске.

```sh
sudo cfdisk /dev/sda
```

Нам предложат выбрать тип системы, для UEFI выбираем `gpt`.

![Выбор типа файловой системы](../uploads/opensuse_install_3.png)

После чего создаем новый раздел и вводим количество необходимой памяти.

![Создание раздела](../uploads/opensuse_install_4.png)

А также меняем тип раздела при необходимости.

![Выбор типа раздела](../uploads/opensuse_install_5.png)

!!!
Нам нужно три раздела:
1. 1G - EFI Filesystem
2. 4G - Linux swap
3. +100% - Linux filesystem
!!!

Далее выбираем `Write` подтверждаем создание разделов и выходим.

Теперь при помощи все той же команды `lsblk`,
мы можем увидеть, что разделы успешно создались.

![Созданные разделы](../uploads/opensuse_install_6.png)

### Форматирование разделов

Для **/** разделов я буду использовать `ext4` файловую систему.

!!!
Вероятно позже добавлю настройку `xfs` и `btrfs`.
Крайне сомнительно, но может быть появится `bcachefs`.
!!!

```sh
sudo mkfs.vfat -n EFI /dev/sda1
sudo mkfs.ext4 -L root /dev/sda3
sudo mkswap -L swap /dev/sda2
sudo swapon /dev/sda2
```

### Монтирование разделов

```sh
# Монтирование основных разделов
sudo mkdir -p /mnt/os
sudo mount /dev/sda3 /mnt/os
sudo mkdir -p /mnt/os/boot/efi
sudo mount /dev/sda1 /mnt/os/boot/efi

# Необходимый для операционной системы шлак
sudo mkdir /mnt/os/{proc,sys,dev,run}
sudo mount --types proc /proc /mnt/os/proc
sudo mount --rbind /sys /mnt/os/sys
sudo mount --make-rslave /mnt/os/sys
sudo mount --rbind /dev /mnt/os/dev
sudo mount --make-rslave /mnt/os/dev
sudo mount --bind /run /mnt/os/run
sudo mount --make-slave /mnt/os/run
```

### Установка базовой системы

```sh
sudo zypper --root /mnt/os ar --refresh https://download.opensuse.org/tumbleweed/repo/oss/ oss
sudo zypper --root /mnt/os in kernel-default grub2-x86_64-efi zypper bash man vim shadow util-linux
sudo zypper --root /mnt/os in --no-recommends NetworkManager
```

### Chrooting

```sh
sudo chroot /mnt/os /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

После чего перед нами будет терминал следующего вида.

![Запущенный chroot](../uploads/opensuse_install_7.png)

### Настройка fstab

Для начала нам надо получить UUID всех трех разделов.

```sh
echo "$(blkid)" >> /etc/fstab
```

После чего мы открываем файл `/etc/fstab` и редактируем его по примеру ниже.
Не забывая вставлять нужные UUID.

```sh
UUID=[...]   /boot/efi    vfat    defaults,noatime     0 2
UUID=[...]   none         swap    sw                   0 0
UUID=[...]   /            ext4    noatime              0 1
```

### Установка grub

```sh
dracut -f
grub2-install --efi-directory=/boot/efi --target=x86_64-efi --bootloader-id=opensuse
grub2-mkconfig -o /boot/grub2/grub.cfg
passwd
```

### Загрузка в систему

```sh
exit
umount -l /mnt/os/dev/{/shm,/pts,}
umount -R /mnt/os
reboot
sudo systemctl enable --now NetworkManager
```
