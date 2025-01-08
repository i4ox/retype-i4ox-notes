# Настройка OpenSuse Tumbleweed после установки

## Подключение к сети

```sh
systemctl enable --now NetworkManager

# Если у вас WiFi
nmcli dev wifi
nmcli dev wifi connect "SSID" password "PASSWORD"
```

## Создание пользователя с правами администратора и запрет SSH-подключения для root

```sh
useradd -m $USER
usermod -aG users $USER
groupadd wheel
usermod -aG wheel $USER
zypper in sudo
passwd $USER
```

## Настройка zypper

```sh
sudo vim /etc/zypp/zypp.conf
    solver.onlyRequired = true
    solver.allowVendorChange = false
    solver.cleandepsOnRemove = false
    download.use_deltarpm = true
    download.max_concurrent_connections = 10
```

## Установка программ

### Установка необходимых утилит

```sh
sudo zypper in curl wget git gpg2 stow
```

### Терминальный эмулятор: Ghostty

**Ghostty** - это эмулятор терминала, который отличается тем, что он быстрый, многофункциональный и нативный.
Хотя существует множество отличных эмуляторов терминала, все они заставляют вас выбирать между скоростью, функциями или нативными пользовательскими интерфейсами.
Ghostty предоставляет все три.

```sh
sudo zypper addrepo https://download.opensuse.org/repositories/X11:/terminals/openSUSE_Factory/X11:terminals.repo
sudo zypper refresh
sudo zypper install ghostty
```

### Браузер: qutebrowser

```sh
sudo zypper in qutebrowser
```

### Оконный менеджер: bspwm

```sh
sudo zypper in xorg-x11-server xinit
sudo zypper in bspwm sxhkd picom polyber flameshot rofi dunst nitrogen
```
