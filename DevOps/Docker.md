# Docker

## Установка Docker

Существует несколько способов установки Docker:

- Через пакетный менеджер, такие как apt-get, pacman, dnf, brew;
- Установить Docker Desktop и docker установится сам;
- Через скрипт с сайта [https://get.docker.com](https://get.docker.com).

Рекомендуется использовать последний способ, он и описан ниже:

```bash
curl https://get.docker.com > /tmp/install.sh
sudo chmod +x /tmp/install.sh
/tmp/install.sh
```

!!!
Не используйте sid для Debian, если хотите использовать способ выше.
!!!

Альтернативный способ:

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Запуск Docker без sudo

Нужно просто добавить пользователя в нужную группу, а затем перезапустить docker.

```bash
sudo usermod -aG docker $USER
sudo systemctl restart docker
```
