# Администрирование Linux

## Краткая сводка по командам

### Пользователи и группы

- Создание пользователей: `useradd`, `passwd`
- Изменение пользователей: `usermod`
- Удаление пользователей: `userdel`
- Создание групп: `groupadd`
- Изменение групп: `groupmod`
- Удаление групп: `groupdel`
- Работа с sudo: `visudo`, `sudoers`

## Основы Linux

**Linux** — это операционная система с многозадачностью, многопользовательской поддержкой и строгими правами доступа.

### FHS - Filesystem Hierarchy Standard

В Linux все файлы организованы в виде древовидной структуры с корнем `/`.

Основные директории:

- `/` - корневой каталог, точка начала всей файловой системы.
- `/bin` - основные исполняемые файлы.
- `/sbin` - системные утилиты.
- `/boot` - загрузочные файлы.
- `/dev` - устройства, представленные как файлы.
- `/etc` - конфигурационные файлы.
- `/home` - домашние каталоги пользователей.
- `/lib`, `/lib64` - системные библиотеки.
- `/media`, `/mnt` - точки монтирования внешних устройств.
- `/opt` - файлы для дополнильных программ.
- `/proc` - виртуальная файловая система(информация о процессах и системе).
- `/root` - корневой каталог root-пользователя.
- `/run`, `/tmp` - временные файлы.
- `/srv` - сервисные файлы.
- `/sys` - системные файлы.
- `/usr` - программы и библиотеки для пользователей.
- `/var` - изменяемые данные(логи, кеши, почта)

### Права доступа (chmod, chown, unmask)

Каждый файл в Linux имеет владельца, группу и права.
При просмотре файлов можно увидеть их права:

```bash
ls -l
# -rw-r--r--. 1 alokhov alokhov  307 Jan 31 16:35  retype.yml
```

**Обозначения прав**:

| Символ | Значение |
|--------|----------|
|`-`|файл(если `d` - каталог)|
|`rw-`|владелец: чтение(`r`), запись(`w`), нет выполнения(`-`)|
|`r--`|группа: только чтение|
|`r--`|остальные пользователи: тоько чтение|

Чтобы изменить права доступа, используется команда `chmod`.

```bash
chmod u+rwx,g+r,o-rwx file.txt # При помощи символов
chmod 750 file.txt # В числовом формате: 111, 101, 000
```

Через команду `umask` можно установить какие права НЕ УСТАНАВЛИВАТЬ при создании файла.

```bash
umask # Получить текущую маску
umask 022 # То есть файлы будут с правами 755
```

Также можно сменить владельца файла, используя команду `chown`.

```bash
sudo chown user file.txt # Смена владельца файла
sudo chown user:group file.txt # Смена владельца и группы(пользователь, который состоит в данной группе)
```

### Основные комманды

Работа с файлами и каталогами:

```bash
ls -lah # Просмотр содержимого каталога
cd /path/to/directory # Переход в каталог
cp source.txt destination.txt # Копирование файлов
mv oldname.txt newname.txt # Перемешение/переименование
rm -rf mydir # Удаление файлов и каталогов
```

Поиск файлов:

```bash
find / -name "file.txt" # Поиск файла по имени
```

Работа с текстовыми файлами:

```bash
head -n 10 file.txt # Просмотр первых 10 строк
tail -f file.txt # Просмотр последних строк(обновляется в реальном времени)
grep "work" file.txt # Поиск строки в файле
awk '{print $1}' file.txt # Фильтрация текста (выводится лишь первый аргумент)
sed 's/old/new/g' file.txt # Замена слова внутри файла
```

Перенаправление вывода:

```bash
echo "Hello" > file.txt # Перезапись в файл
echo "World" >> file.txt # Добавление в файл
ls - l | grep ".txt" # Перенаправление вывода в ввод другой команды
ls -l | tee output.txt # Перехватывает вывод, записывает его в файл и выводит на экран
```

## Процессы

### Просмотр процессов

```bash
ps aux # Показать все процессы
ps -ef # Показать все процессы с их ID
ps -ef | grep "systemd" # Показать все процессы, содержащие слово "systemd"
top # Мониторинг процессов
htop # Удобный мониторинг(нужно отдельно установить)
```

### Управление процессами

```bash
kill PID # завершить процесс с определенным ID
kill -9 PID # Принудительное завершение, игнорируя все закрывающие процессы
Ctrl + Z # приостановить процесс
fg # возобновить процесс
```

### Приоритет процессов (nice, renice)

```bash
nice -n 10 myscript.sh # Запуск с низким приоритетом
renice -n 5 -p PID # Повысить приоритет процесса с PID до 5
```

## Управление пользователями и группами

В Linux пользователи могут быть _обычными_ или _системными_.
Системные пользователи используются для служб-демонов.

**Группы** - компонент системы, который позволяет управлять правами доступа к файлам и ресурсам.

### Создание пользователя

Создать пользователя можно с помощью команды `useradd`.

```bash
sudo useradd user_name
```

!!!
По-умолчанию:
- Пользователь создается без домашней директории.
- Используется `/etc/skel` для шаблона конфигурации пользователя.
!!!

Чтобы создать пользователя с домашней директорией:

```bash
sudo useradd -m user_name
```

Ниже приведена полная команда со всеми популярными опциями.

```bash
sudo useradd -m -s /bin/bash -c "Arthur Lokhov" -g users -G sudo user_name
```

Опции:

- `-m` - создать домашнюю директорию для пользователя
- `-s` - указать путь к оболочке для пользователя
- `-c` - указать описание пользователя
- `-G` - указать группы пользователя
- `-g` - указать основную группу пользователя

### Смена пароля пользователя

После создания пользователя ему нужно задать пароль.

```bash
sudo passwd user_name
```

Если надо сменить текущему пользователю, то можно использовать просто `passwd`.

```bash
passwd
```

### Изменение параметров пользователя

Если нужно изменить уже существующего пользователя, используется команда `usermod`.

```bash
sudo usermod -l new_user_name user_name # изменение логина
sudo usermod -d /home/new_user_name -m user_name # изменение домашней директории
sudo usermod -s /bin/zsh user_name # изменение оболочки
sudo usermod -aG new_group_name user_name # добавление группы
```
!!!
Флаг `-a` используется для добавление пользователя в группу, без удаления из старых.
!!!

### Удаление пользователя

Если нужно удалить пользователя, используется команда `userdel`.

```bash
sudo userdel user_name
```

Правда этот вариант оставляет домашнюю директорию пользователя, чтобы удалить ее используется команда ниже.

```bash
sudo userdel -r user_name
```

### Управление группами

Создать новую группу:

```bash
sudo groupadd new_group_name
```

Изменить существующую группу:

```bash
sudo groupmod -n new_group_name old_group_name
```

Удалить группу:

!!!
Группу нельзя удалить, если она содержит пользователей.
!!!

```bash
sudo groupdel group_name
```

### Просмотр информации о группах и пользователях

Узнать основную группу пользователя:

```bash
id user_name
```

Проверить в каких группах состоит пользователь:

```bash
groups user_name
```

## Sudo и политика доступа

**Sudo** позволяет временно выполнять команды от имени суперпользователя.

### Добавление пользователя в sudo

Чтобы дать права суперпользователя пользователю:

```bash
sudo usermod -aG sudo user_name # Debian/Ubuntu
sudo usermod -aG wheel user_name # RHEL/CentOS
```

### Файл конфигурации sudoers

Настройка `sudo` хранится в файле `/etc/sudoers`. Изменять его вручную не рекомендуется, лучше использовать:

```bash
sudo visudo
```

Пример строки для добавления пользователя в sudo. Эта команда дает полный доступ к `sudo`.

```bash
user_name ALL=(ALL:ALL) ALL
```

### Запрет на запрос пароля при использовании sudo

Можно запретить запрос пароля при использовании sudo:

```bash
user_name ALL=(ALL) NOPASSWD: ALL
```

!!!
Это может быть небезопасно!
!!!

### Ограничение команд для sudo

Можно ограничить команды, которые могут быть выполнены с помощью sudo:

```bash
user_name ALL=(ALL) NOPASSWD: /usr/sbin/service nginx restart
```

В данном примере без запроса пароля можно выполнить команду `sudo service nginx restart`.

## Управление пакетами в Linux

Linux использует **пакетные менеджеры** для устаноки, обновления и удаления программ.
В зависимости от дистрибутива используются разные системы управления пакетами.

### Debian-based(APT, DPKG)

**APT(Advanced Package Tool)** - это высокоуровневый менеджер пакетов для Debian, Ubuntu и их производных.
Он работает поверх `dpkg`, который является низкоуровневым инструментов установки `.deb` пакетов.

Работа с `apt`:

```bash
sudo apt update # Обновление списка пакетов
sudo apt install upgrade # Обновление всех установленных пакетов
apt search package_name # Поиск пакета по имени
sudo apt install package_name # Установка пакета
sudo apt remove package_name # Удаление пакета
sudo apt purge package_name # Удаление пакета вместе с конфигурацией
sudo apt autoremove # Удаление всех неиспользуемых пакетов
```

Работа с `dpkg`:

```bash
sudo dpkg -i package.deb # Установка .deb пакетов
sudo apt -f install # Исправление зависимостей после установки .deb через dpkg
sudo dpkg -r package_name # Удаление пакета
sudo -l | grep package_name # Просмотр информации о пакете
sudo dpkg --add-architecture i386 # Добавление пакетов определенной архитектуры
```

### RedHat-based(YUM, DNF, RPM)

**YAM(Yellowdog Updater, Modified)** и **DNF(Dandifined YUM)** используются в RHEL, CentOS и Fedora.

**RPM(RedHat Package Manager)** - это низкоуровневый инструмент работы с `.rpm` пакетами.

Работа с `dnf`(современный пакетный менеджер):

```bash
sudo dnf check-update # Обновление списка пакетов
sudo dnf upgrade # Обновление всех установленных пакетов
sudo dnf search package_name # Поиск пакета по имени
sudo dnf install package_name # Установка пакета
sudo dnf remove package_name # Удаление пакета
sudo dnf clean all # Очистка кэша пакетов
```

Работа с `yum`(старый пакетный менеджер, используется в CentOS 7 и RHEL 7):

```bash
sudo yum check-update # Обновление списка пакетов
sudo yum update # Обновление всех установленных пакетов
sudo yum search package_name # Поиск пакета по имени
sudo yum install package_name # Установка пакета
sudo yum remove package_name # Удаление пакета
sudo yum clean all # Очистка кэша пакетов
```

Работа с `rpm`:

```bash
sudo rpm -i package.rpm # Установка пакета
sudo rpm -U package.rpm # Обновление пакета
sudo rpm -e package.rpm # Удаление пакета
rpm -qa | grep "package_name" # Поиск пакета по имени
```

### Компиляция из исходников(make, gcc, tar, ./configure)

Если нужного пакета нет в репозитории, можно установить его из исходников.

```bash
tar -xvzf package.tar.gz # Распаковка архива с исходниками
tar -xvJf package.tar.xz # Распаковка архива с исходниками
cd package/ # Переход в каталог с исходниками

sudo apt install build-essential # Установка пакетов для компиляции (Debian)
sudo dnf groupinstall "Development Tools" # Установка пакетов для компиляции (RHEL)

./configure # Проверяет зависимости и готовит сборку
make # Компиляция исходников
sudo make install # Установка пакета
```

## Управление дисками и файловыми системами в Linux

Linux предоставляет мощные инструменты для разметки дисков, работы с файловыми системами, управления `LVM` и `RAID`, а также шифрования данных с помощью `LUKS`.

### Разметка дисков (fdisk, parted)

Перед использование любого диска его необходимо разметить, а затем создать файловую систему.

Чтобы вывести список дисков и разделов, можно использовать команду `lsblk`:

```bash
lsblk
```

А чтобы посмотреть информацию о дисках, можно использовать команду `fdisk -l`:

```bash
sudo fdisk -l
```

Разметка диска с помощью `fdisk`:

```bash
sudo fdisk /dev/sda # Запуск разметки диска для диска /dev/sda
```

Основные команды для `fdisk`:

- `n` - создать новый раздел
- `d` - удалить раздел
- `w` - записать изменения
- `p` - показать таблицу разделов
- `q` - выйти из разметки

Разметка диска с помощью `parted`:

```bash
sudo parted /dev/sda # Запуск разметки диска для диска /dev/sda

# Команды ниже выполняются в parted
mklabel gpt # Создать GPT-разделитель
mkpart primary ext4 0% 100% # Создать раздел ext4 с начала диска и длиной 100%
quit # Выйти из parted
```

### Файловые системы(ext4, xfs, btrfs, zfs)

Создание файловых систем:

```bash
sudo mkfs.ext4 /dev/sda1 # ext4 используется по-умолчанию в Linux(где-то уже перешли на btrfs)
sudo mkfs.xfs /dev/sda1 # xfs хорош для больших файлов
sudo mkfs.btrfs /dev/sda1 # btrfs, современная с поддержкой снапшотов

sudo apt install zfsutils-linux # Установка zfs для Debian/Ubuntu, в RHEL уже установлена
sudo zpool create mypool /dev/sda # zfs, производительная, надежная сжимаемая
```

Проверка и исправления файловой системы:

```bash
sudo fsck -f /dev/sda1 # Проверка файловой системы
sudo e2fsck -p /dev/sda1 # Проверка ext4
sudo xfs_repair /dev/sda1 # Проверка xfs
sudo btrfs check /dev/sda1 # Проверка btrfs
sudo zpool status # Проверка zfs
```

### Монтирование дисков и fstab

После создания файловой системы, диск нужно подключить в систему.

Временное монтирование диска делается с помощью команды `mount`:

```bash
sudo mount /dev/sda1 /mnt # Монтирование диска в папку /mnt
```

Для того, чтобы размонтировать временно монтированный диск, используется команда `umount`:

```bash
sudo umount /mnt # Размонтирование диска из папки /mnt
```

Если диск нужно примонтировать в систему на постоянной основе, то можно использовать файл `fstab`.
Но сначала надо опредилить UUID нужного нам диска. Это можно сделать рядом способов:

```bash
blkid /dev/sda1 # Получить UUID диска
lsblk -o +uuid,name # Получить UUID диска и имя диска
```

После чего добавить строку ниже в файл `/etc/fstab`:

```bash
UUID=xxxx-xxxx /mnt ext4 defaults 0 0
```

Чтобы применить изменения в fstab без перезагрузки системы:

```bash
sudo mount -a # Применить изменения в fstab
```
### RAID

**RAID** – это способ объединения нескольких дисков в один для более высокой производительности, или для того, чтобы они работали как один том.

Уровни RAID:

| RAID уровень | Описание | Надежность | Производительность | Минимальное количество дисков |
|---------------|----------|------------|---------------------|------------------------------|
| RAID 0 (Stripting) | Разделение данных между дисками, без избыточности | Нет защиты данных | Высокая(чтение и запись) | 2 |
| RAID 1 (Mirroring) | Полное зеркалирование данных | Высокая(один из дисков может выйти из строя) | Чуть ниже, чем одиночный диск | 2 |
| RAID 5 | Чередование данных с паритетом | Один диск можно потерять | Высокая(но запись медленнее из-за вычисления паритета) | 3 |
| RAID 6 | Двойной паритет(выдерживает отказ 2 дисков) | Очень высокая | Запись медленнее, чем RAID 5 | 4 |
| RAID 10 | Зеркалирование + чередование(RAID 0 + RAID 1) | Высокая | Отличная | 4 |

Упралять RAID можно с помощью команды `mdadm`:

```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda /dev/sdb # Создание RAID-1
sudo mdadm --detail /dev/md0 # Показать сведения о RAID
sudo mdadm --stop /dev/md0 # Остановить RAID
sudo mdadm --remove /dev/md0 # Удалить RAID
sudo mdadm --increment /dev/md0 # Увеличить или уменьшить RAID(добавить или удалить диски)
```

Чтобы просмотреть статусы RAID:

```bash
cat /proc/mdstat # Просмотр статусов RAID
```

### LVM

 **LVM** — это дополнительный слой абстракции от железа, позволяющий собрать кучи разнородных дисков в один, и затем снова разбить этот один именно так как нам хочется. Реализовано с помощью подсистемы `device-mapper`.

**PV(Physical Volume)** — физические тома(могут быть разделы или целые "неразбитые" диски).
**VG(Volume Group)** — группа томов(объединяем физические тома в группу, создаем единый диск, который дальше будет разбивать так, как нам хочется).
**LV(Logical Volume)** — логические разделы, собственно раздел нашего нового "единого диска", группы томов, который мы потом форматируем и используем как обычный раздел, обычного жесткого диска.

Создание физического тома:

```bash
sudo pvcreate /dev/sdb
```

Создание группы томов:

```bash
sudo vgcreate vg0 /dev/sdb
```

Создание логического тома:

```bash
sudo lvcreate -L 10G -n lv0 vg0
```

Форматирование и монтирование тома:

```bash
sudo mkfs.ext4 /dev/vg0/lv0
sudo mount /dev/vg0/lv0 /mnt
```

### Шифрование дисков с LUKS

**LUKS(Linux Unified Key Setup)** — стандартное решение для шифрования дисков в Linux.

Установка LUKS:

```bash
sudo apt install cryptsetup # Debian/Ubuntu
sudo dnf install cryptsetup # RHEL
```

Создание зашифрованного раздела LUKS:

!!!
Удалит все данные!
!!!

```bash
sudo cryptsetup luksFormat /dev/sdb # Подтведите шифрование(YES и пароль)
```

Открытие зашифрованного раздела LUKS:

```bash
sudo cryptsetup open /dev/sdb cryptdisk
```

!!!
Теперь зашифрованный раздел будет доступен под именем `/dev/mapper/cryptdisk`.
!!!

Форматирование и монтирование зашифрованного раздела:

```bash
sudo mkfs.ext4 /dev/mapper/cryptdisk
sudo mkdir /mnt/secure
sudo mount /dev/mapper/cryptdisk /mnt/secure
```

Автоматическое монтирование делается при помощи `/etc/crypttab` и `/etc/fstab`:

```bash
# /etc/crypttab
cryptdisk UUID=xxxx-xxxx none luks

# /etc/fstab
/dev/mapper/cryptdisk /mnt/secure ext4 defaults 0 0

# Применение изменений
sudo systemctl daemon-reexec
sudo mount -a
```

Закрытие зашифрованного раздела LUKS:

```bash
sudo umount /mnt/secure
sudo cryptsetup close cryptdisk
```

## Система инициализации и службы

В Linux управление службами и процессами играет ключевую роль в администрировании.
Основная система инициализации - `systemd`, но также существуют альтернативные системы(`SysVinit`, `Upstart`, `runut`, `OpenRC`, `dunit`, `s6` и т.д.).

### systemd

**systemd** - это основной системный менеджер инициализации в большинстве современных дистрибутивов.

Чтобы проверить используется ли systemd:

```bash
ps --pid 1
```

Основные команды systemd:

```bash
sudo systemctl start nginx # Запуск службы
sudo systemctl stop nginx # Остановка службы
sudo systemctl restart nginx # Перезапуск службы
sudo systemctl reload nginx # Перезагрузка службы без остановки
```

Проверка статуса службы:

```bash
sudo systemctl status nginx
```

Автозагрузка сервисов при старте системы:

```bash
sudo systemctl enable nginx # Включить автозапуск
sudo systemctl enable --now nginx # Включить автозапуск и запустить сервис
sudo systemctl disable nginx # Отключить автозапуск
```

Проверить включен ли автозапуск службы:

```bash
systemctl is-enabled nginx
```

Проверка всех запущенных служб:

```bash
systemctl list-units --type=serviced
```

### Написание собственных служб

Службы в `systemd` описываются в **unit-файлах**(`.service`).

Создание простого сервиса:

```bash
sudo nano /etc/systemd/system/my-service.service
```

Пример unit-файла:

```ini
[Unit]
Description=Мой тестовый сервис # Описание службы
After=network.target # После чего запускать службу

[Service]
ExecStart=/usr/bin/python3 /opt/myscript.py # Запуск скрипта
Restart=always # Когда служба перезапускается
User=nobody # От лица какого-то пользователя
Group=nogroup # От лица какой-то группы

[Install]
WantedBy=multi-user.target # Запускать службу при запуске системы
```

Включение службы в автозапуск:

```bash
sudo systemctl daemon-reload # Перезагрузить systemd
sudo systemctl enable my-service # Включить службу
sudo systemctl start my-service # Запустить службу
systemctl status my-service # Проверить статус службы
```

### Таймеры systemd

```bash
sudo nano /etc/systemd/system/my-timer.timer
```

Пример unit-файла:

```ini
[Unit]
Description=Мой тестовый таймер # Описание таймера

[Timer]
OnBootSec=10 # После загрузки системы запустить таймер через 10 секунд
OnUnitActiveSec=60 # После запуска службы запустить таймер через 60 секунд
Unit=my-service.service # Запускать таймер при запуске службы

[Install]
WantedBy=timers.target # Запускать таймер при запуске системы
```

Включение таймера в автозапуск:

```bash
sudo systemctl daemon-reload # Перезагрузить systemd
sudo systemctl enable my-timer # Включить таймер
sudo systemctl start my-timer # Запустить таймер
systemctl list-timers # Проверить статус таймера
```

## Резервное копирование и восстановление

**Резервное копирование** - важная часть администрирования системы. В Linux есть несколько инструментов.

| Инструмент | Описание |
|------------|----------|
| `rsync` | Инкрементальное копирование файлов по сети |
| `tar` | Архивация файлов и директорий |
| `dd` | Клонироваие дисков и разделов |
| `scp` | Копирование файлов по сети |
| `BorgBackup` | Эффективное дедуплицированное резервное копирование |

### rsync

```bash
rsync -av /home/user /backup/ # Простое копирование директории
rsync -av -e ssh /home/user user@remote:/backup/ # Копирование директории по сети
rsync -av --delete /home/user /backup/ # Копирование директории с удалением исходной директории
```

### tar

```bash
tar -cvzf backup.tar.gz /home/user # Архивирование директории
tar -xvzf backup.tar.gz -C /restore/ # Распаковка архива
```

### dd

```bash
dd if=/dev/sda of=/backup/disk.img bs=4M status=progress # Создание полного образа диска
dd if=/backup/disk.img of=/dev/sdb bs=4M status=progress # Восстановление из образа
```

### scp

```bash
scp -r /home/user user@remote:/backup/ # Копирование директории по сети
```

### BorgBackup

Установка:

```bash
sudo apt install borgbackup # Debian/Ubuntu
sudo dnf install borgbackup # RHEL
```

Использование:

```bash
borg init --encryption=repokey /backup/repo # Инициализация репозитория
borg create --compression=lz4 /backup/repo::backup-$(date +%Y-%m-%d) /home/user # Создание резервной копии
borg extract /backup/repo::backup-2024-02-07 # Восстановление резервной копии
```

### Снапшоты(LVM, ZFS, Btrfs)

LVM снапшоты:

```bash
lvcreate -L 1G -s -n my_snapshot /dev/vg0/my_volume # Создание снапшота
lvremove -f /dev/vg0/my_snapshot # Удаление снапшота
```

ZFS снапшоты:

```bash
zfs snapshot pool/dataset@backup # Создание снапшота
zfs list -t snapshot # Просмотр снапшотов
zfs rollback pool/dataset@backup # Откат к снапшоту
zfs destroy pool/dataset@backup # Удаление снапшота
```

BTRFS снапшоты:

```bash
btrfs subvolume snapshot /home /backup/home_snapshot # Создание снапшота
btrfs subvolume delete /backup/home_snapshot # Удаление снапшота
```

## Безопасность в Linux

Основные аспекты безопасности в Linux:

- Контроль доступа: `SELinux`, `AppArmor`
- Защита от атак: `fail2ban`
- Аудит системы: чек-листы безопасности

### SELinux(Security-Enhanced Linux)

```bash
setstatus # Проверка SELinux
sudo setenforce 0 # Временно отключить SELinux
sudo nano /etc/selinux/config # Изменить режим SELinux
> SELINUX=permissive # enforcing / permissive / disabled
```

Режимы SELinux:

- `enforcing` - принудительно включен
- `permissive` - включен, но принудительно отключен
- `disabled` - выключен

### AppArmor

```bash
sudo aa-status # Проверка AppArmor
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx # Отключить AppArmor для nginx
sudo aa-disable # Отключить все AppArmor
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx # Включить AppArmor для nginx
sudo aa-enforce # Включить все AppArmor
```

### Fail2ban - защита от брутфорса

Fail2ban блокирует IP-адреса после нескольких неудачных попыток входа.

```bash
sudo apt install fail2ban # Debian/Ubuntu
sudo dnf install fail2ban # RHEL
```

Настройка fail2ban(`/etc/fail2ban/jail.local`):

```ini
[sshd]
enabled = true
maxretry = 3
bantime = 3600
```

Перезапуск fail2ban:

```bash
sudo systemctl restart fail2ban
```
Просмотр заблокированных IP-адресов:

```bash
sudo fail2ban-client status sshd
```

### Чек-листы безопасности

1. Обновление системы.
   ```bash
   sudo apt update && sudo apt upgrade -y # Debian/Ubuntu
   sudo dnf upgrade -y # RHEL
   ```
2. Ограничение sudo доступа(`visudo`).
3. Отключение ненужных служб.
4. Защита паролей:
   ```bash
   passwd -l root # Заблокировать root пароль
   ```
5. Мониторинг логов.

## Журналирование и мониторинг

Журналирование и мониторинг помогают ослеживать состояние системы, выявлять проблемы и анализировать производительность.

### Логирование

**journald** - системный журнал `systemd`. Он хранит логи в бинарном формате и управляется через `journalctl`.

```bash
journalctl # Просмотр всех логов
journalctl -u nginx  # Логи конкретного сервиса
journalctl -b # Логи после перезагрузки системы
journalctl --since "1 hour ago"  # Логи за последний час
journalctl --vacuum-time=7d # Очистка журнала с логами старше 7 дней
```

**syslog** и **rsyslog** - традиционное логирование. Они записывают логи в текстовые файлы, такие как `/var/log/syslog` и `/var/log/auth.log`.

```bash
tail -f /var/log/syslog # Просмотр логов
tail -f /var/log/auth.log # Просмотр логов аутентификации
```

Настройка филтрации логов в `/etc/rsyslog.conf`.

```conf
:programname, isequal, "sshd" /var/log/ssh.log
& stop
```

!!!
Запись выше отправляет логи с процесса `sshd` в файл `/var/log/ssh.log`.
!!!

После изменения настроек надо перезапустить службу `rsyslog`.

```bash
sudo systemctl restart rsyslog
```

### Мониторинг ресурсов

| Команда | Описание |
|---------|----------|
| `htop` | Графический просмотр процессов |
| `iostat` | Нагрузка на диски |
| `vmstat` | Использование CPU, памяти |
| `free` | Свободная память |
| `sar` | Сбор статистики нагрузки |

Примеры:

```bash
htop                # Запуск интерактивного мониторинга процессов
iostat -x 1 5       # Статистика по диску (обновляется 5 раз)
vmstat 2 5          # Мониторинг CPU и памяти (2 сек, 5 раз)
free -h             # Проверка использования памяти
```

### Инструменты мониторинга

`Nagios`:

- Мониторинг серверов и сетевого оборудования.
- Проверка доступности (ping, HTTP, SSH).
- Оповещения (email, Telegram, Slack).

`Zabbix`:

- Сбор данных (CPU, память, диск).
- Графики и триггеры.
- Агенты для Windows/Linux.

`Prometheus + Grafana`:

- Сбор метрик(CPU, память, сети).
- Интеграция с Kubernetes, Docker.
- Визуализация данных через Grafana.

## Виртуализация и контейниризация

**Виртуализация** и **контейниризация** - это технологии, позволяющие запускать несколько изолированных сред на одном физическом сервере, но у них разные принципы работы.

| Параметр | Виртуализация | Контейниризация |
|----------|---------------|------------------|
| Изоляция | Полноценная ОС с ядром | Использует ядро хоста |
| Ресурсы | Требует больше CPU/RAM | Легковесная |
| Запуск | Минуты(из-за загрузки ОС) | Секунды |
| Гибкость | Можно запускать разные ОС | Все контейнеры используют одно ядро |
| Безопасность | Высокая(отдельные ядра) | Меньше изоляции, но можно использовать `SELinux`, `AppArmor` |

### Виртуализация

**QEMU/KVM** - нативная виртуализация на ядре Linux.

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo # Проверка поддержки виртуализации
sudo apt install qemu-kvm libvirt-daemon-system virt-manager # Debian/Ubuntu
sudo dnf install qemu-kvm libvirt-daemon virt-manager # RHEL

virt-install --name ubuntu-vm --ram 2048 --vcpus 2 --disk size=10 --cdrom ubuntu.iso # Создание виртуальной машины
```

**Proxmox VE** - мощная виртуализация на базе KVM.
**Proxmox** - платформа для управления виртуальными машинами и контейнерами.

Установка Proxmox VE:

1. Скачиваем ISO с официального сайта.
2. Устанавливаем на сервер.
3. Управляем через web-интерфейс.

**VMWare ESXi** - коммерческий гипервизор для виртуализации в дата-центрах.

**Hyper-V** - виртуализация от Microsoft для Windows Server.

**VirtualBox** - простая виртуализация для тестов

### Контейнеризация

**Docker** - основной инструмент контейнеризации.

!!!
Контейнеризация и Docker разберу отдельно.
!!!

```bash
sudo apt install docker.io # Debian/Ubuntu
sudo dnf install docker # RHEL

docker run -d -p 8080:80 nginx # Запуск контейнера
docker ps # Просмотр списка контейнеров
docker rm -f container-id # Удаление контейнера
```

**Podman** - альтернатива Docker без демона.

```bash
sudo apt install podman # Debian/Ubuntu
sudo dnf install podman # RHEL

podman run -d -p 8080:80 nginx # Запуск контейнера
```

**Kubernetes** - оркестрация контейнеров. Он управляет кластерами контейнеров, балансировкой нагрузки и масштабированием.

!!!
Kubernetes разберу отдельно.
!!!

```bash
# Установка локального кластера
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube

minikube start # Запуск локального кластера
kubectl create deployment nginx --image=nginx # Создание деплоя
kubectl expose deployment nginx --port=80 --type=NodePort # Направление порта
```
