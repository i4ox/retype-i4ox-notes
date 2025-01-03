# SSH

## Генерация ssh-ключей

```bash
ssh-keygen -t ed25519 -f ~/.ssh/key -C "comment"
```

## Подключение по SSH

```bash
ssh -i ~/.ssh/key user@host
```

## SSH-agent

ssh-agent - менеджер ключей для SSH. Он хранит ваши ключи и сертификаты в памяти, незашифрованные и готовые к использованию ssh. Это избавляет вас от необходимости вводить пароль каждый раз, когда вы подключаетесь к серверу. Он работает в фоновом режиме в вашей системе, отдельно от ssh, и обычно запускается при первом запуске ssh.

Чтобы добавить пароль от приватного ключа:

```bash
ssh-add ~/.ssh/key
```

## SSH-copy-id

Чтобы скопировать ключ на сервер используем комманду ниже:

```bash
ssh-copy-id -i ~/.ssh/key user@host
```

Не забываем про выдачу необходимых прав.

```bash
chown -R user:user /home/user/.ssh
chmod 700 ~/.ssh
chmod 644 ~/.ssh/*
chmod 600 ~/.ssh/id_* ~/.ssh/authorized_keys
```

## sshd_config

### Лучшие практики

1. Меняем порт с 22 на любой другой
2. PermitRootLogin no
3. PubkeyAuthentication yes
4. PasswordAuthencication no (исключительно после настройки ключей)
5. PermitEmptyPasswords no (не имеет смысла, если соблюдать 4 пункт)

## Настройки клиента

Клиент SSH настраивается через файл ~/.ssh/config.

Пример настроек:

```txt
Host server
        StrictHostKeyChecking no
        User sampleuser
        ForwardAgent yes
        IdentityFile /Users/username/.ssh/id_rsa
        IdentitiesOnly yes
        UserKnownHostsFile=/dev/null
        UseKeychain yes
        AddKeysToAgent yes
        ServerAliveInterval 60
        ServerAliveCountMax 1200
```

Далее делается подключение через команду ниже:

```bash
ssh server
```

## Пробрасывание портов

```bash
ssh -L 9000:localhost:8000 user@server
```

Командой выше мы создали подключение через SSH-туннель, с пробросом внешнего 8000 порта на локальный 9000 порт. И после этого ***НЕ ЗАКРЫВАЯ*** соединения, в другой вкладке терминала мы сможет обращаться по этому порту.

## Моя конфигурация клиента SSH

Ниже расположен пример моей конфигурации Linux

```txt
Host github
  HostName www.github.com
  Port 22
  User git
  IdentityFile ~/.ssh/github

Host forgejo
  HostName git.adevteam.com
  Port 22
  User git
  IdentityFile ~/.ssh/forgejo

Host *
  AddKeysToAgent yes
  IgnoreUnknown UseKeychain
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```
