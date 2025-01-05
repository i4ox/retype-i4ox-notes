# Настройка OpenSuse Tumbleweed после установки

## Подключение к сети

```sh
systemctl enable --now NetworkManager

# Если у вас WiFi
nmcli dev wifi
nmcli dev wifi connect "SSID" password "PASSWORD"
```

## Настройка SSHD

```sh
zypper install openssh
vim /usr/etc/ssh/sshd_config
# PermitRootLogin yes
```

## Создание пользователя с правами администратора и запрет SSH-подключения для root

```sh
useradd -m $USER
usermod -aG users $USER
groupadd wheel
usermod -aG wheel $USER
zypper in sudo
passwd $USER

vim /usr/etc/ssh/sshd_config
# PermitRootLogin no
systemctl restart sshd
```

## Установка основы в виде минимального KDE

```sh
sudo zypper in xorg-x11-server xinit
sudo zypper in --no-recommends -t pattern kde
sudo zypper in plasma6-session-x11
sudo reboot
```
