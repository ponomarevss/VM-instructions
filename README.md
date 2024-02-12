# Установка и настройка виртуальной машины
В этом руководстве рассматривается вариант установки и настройки виртуальной машины с операционной системой Ubuntu-22.04.3-live-server на хостовую Ubuntu-22.04.3-desktop. Для создания и управления виртуальной машиной используется стандартная утилита Virtual Machne Manager. После установки операционной системы, разворачивается пакет: OpenSSH, vsftpd, PostgreSQL, Docker, Git, Python, Redis.
## Установка и запуск Ubuntu-22.04.3-live-server
1. Загрузка образа.
   - Забрать образ операционной системы здесь: https://ubuntu.com/download/server
   - Сохранить образ ubuntu-22.04.3-live-server-amd64.iso
2. Создание ВМ
   - Стандартная утилита Virtual Machne Manager доступна для установки из портфеля Ubuntu Store. 
   - После ее запуска выбрать File/New Virtual Machine
   - Далее пункт Local install media (ISO image or CDROM)
   - Затем в окне Choose ISO install media выбрать сохраненный ранее образ ubuntu-22.04.3-live-server-amd64.iso. Проверить, что выбрана опция Automatically detect from the installation media
   - Следующий шаг: выбрать количество памяти (4ГБ) и ядер (2) для ВМ. 
   - Выбрать опцию Enable storage for this virtual machine. Выбрать пункт Create a disk for the virtual machine (25 ГБ)
   - Присвоить имя ВМ, проверить сводку по выбранным опциям и нажать Finish. После этого, начнется процесс установки ОС ubuntu-22.04.3-live-server
3. Установка ОС ubuntu-22.04.3-live-server
   - Выбрать язык (English)
   - Далее предложение обновить загрузочный файл (Continue wothout updating)
   - Предложение выбрать основу для инсталляции (Ubuntu Server)
   - Пропустить пункт про прокси, подождать когда прогрузится список зеркал
   - Принять Use an entire disk и далее предложенную сводку
   - Далее в диалоговом окне нажать Select после чего заполнить профиль
   - Пропустить предложение Ubuntu Pro
   - Выбрать Install OpenSSH server
   - Пропустить все предложенные далее сервисы
   - Дождаться окончания процесса и reboot. Ввести пользователя и пароль
## Установка OpenSSH
Если на шаге 3.8 в разделе "Установка и запуск Ubuntu-22.04.3-live-server" выбрана опция Install OpenSSH server, то уже сейчас можно подключиться, используя команду:
```python
	$ ssh your_user_name@your_server_ip
```
После этого остается только пароль для your_user_name
## Установка и настройка vsftpd
1. Установка vsftpd

Апдейт пакетов:
```python
	$ sudo apt update
```
Затем, установка демона vsftpd:
```python
	$ sudo apt install vsftpd
```
Бэкап изначального конфига:
```python
	$ sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.orig
```
2. Открыть Firewall
   
Проверить статус:
```python
	$ sudo ufw status
```
Вот такой вывод говорит, что пропускается только SSH:
```python
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```
Открыть порты 20, 21, и 990:
```python
	$ sudo ufw allow 20,21,990/tcp
```
Затем, открыть порты:
```python
	$ sudo ufw allow 40000:50000/tcp
```
Проверить статус:
```python
	$ sudo ufw status
```
Теперь правила для firewall выглядят так:
```python
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
20,21,990/tcp              ALLOW       Anywhere
40000:50000/tcp            ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
20,21,990/tcp (v6)         ALLOW       Anywhere (v6)
40000:50000/tcp (v6)       ALLOW       Anywhere (v6)
```
3. Подготовка директории пользователя

Добавить тестового пользователя:
```python
	$ sudo adduser sammy
```
Создать ftp папку:
```python
	$ sudo mkdir /home/sammy/ftp
```
Задать ее владельца:
```python
	$ sudo chown nobody:nogroup /home/sammy/ftp
```
Удалить возможность записи:
```python
	$ sudo chmod a-w /home/sammy/ftp
```
Проверить разрешения:
```python
	$ sudo ls -la /home/sammy/ftp
```
```python
Output
total 8
dr-xr-xr-x 2 nobody nogroup 4096 Sep 14 20:28 .
drwxr-xr-x 3 sammy sammy  4096 Sep 14 20:28 ..
```
Далее, создать директорию для загрузки файлов:
```python
	$ sudo mkdir /home/sammy/ftp/files
```
Назначить пользователя владельцем:
```python
	$ sudo chown sammy:sammy /home/sammy/ftp/files
```
Проверить разрешения, чтобы получить нижеследующий вывод:
```python
	$ sudo ls -la /home/sammy/ftp
```
```python
Output
total 12
dr-xr-xr-x 3 nobody nogroup 4096 Sep 14 20:30 .
drwxr-xr-x 3 sammy sammy  4096 Sep 14 20:28 ..
drwxr-xr-x 2 sammy sammy  4096 Sep 14 20:30 files
```
Наконец, добавить test.txt файл для тестирования:
```python
	$ echo "vsftpd test file" | sudo tee /home/sammy/ftp/files/test.txt
```
```python
Output
vsftpd test file
```
4. Настройка FTP доступа

Открыть конфиг файл:
```python
	$ sudo nano /etc/vsftpd.conf
```
Раскомментировать, отредактировать и добавить следующие инструкции:
```python
. . .
# Allow anonymous FTP? (Disabled by default).
anonymous_enable=NO
#
# Uncomment this to allow local users to log in.
local_enable=YES
. . .
. . .
write_enable=YES
. . .
. . .
chroot_local_user=YES
. . .
. . .
user_sub_token=$USER
local_root=/home/$USER/ftp
. . .
pasv_min_port=40000
pasv_max_port=50000
. . .
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```
Наконец, добавить пользователя в /etc/vsftpd.userlist. Использовать флаг -a для аппенда в файл:
```python
	$ echo "sammy" | sudo tee -a /etc/vsftpd.userlist
```
Проверить, что все добавлено:
```python
	$ cat /etc/vsftpd.userlist
```
```python
Output
sammy
```
Рестарт демона для загрузки изменений конфигурации:
```python
	$ sudo systemctl restart vsftpd
```
Дальнейшие шаги по настройке опциональны и описаны здесь:  
https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-a-user-s-directory-on-ubuntu-20-04
## Установка и настройка PostgreSQL
1. Установка PostgreSQL

Обновить индекс пакета:
```python
	$ sudo apt update
```
Установить пакет Postgres с пакетом -contrib, который дает дополнительные утилиты и функционал:
```python
	$ sudo apt install postgresql postgresql-contrib
```
2. Использование ролей и базданных PostgreSQL

Один способ - переключиться в postgres аккаунт:
```python
	$ sudo -i -u postgres
```
Быстрое переключение в Postgres prompt:
```python
	$ psql
```
Для выхода из PostgreSQL prompt набрать:
```python
	postgres=# \q
```
Для выхода в терминал пользователя набрать:
```python	
	postgres@server:~$ exit
```
Другой способ управления postgres:
```python
	$ sudo -u postgres psql
```
3. Создание новой роли

Находясь в аккаунте postgres, новую роль создать командой:
```python
	postgres@server:~$ createuser --interactive
```
Или, используя команду sudo, командой:
```python
	$ sudo -u postgres createuser --interactive
```
```python
Output
Enter name of role to add: sammy
Shall the new role be a superuser? (y/n) y
```
4. Создание новой базы

Находясь в аккаунте postgres, новую базу создать командой:
```python
	postgres@server:~$ createdb sammy
```
Или, используя команду sudo, командой:
```python
	$ sudo -u postgres createdb sammy
```
5. Открытие Postgres Prompt из новой роли

Если нет пользователя, создать командой:
```python
	$ sudo adduser sammy
```
Когда доступен новый аккаунт, набрать:
```python
	$ sudo -i -u sammy
	$ psql
```
Или
```python
	$ sudo -u sammy psql
```
Чтобы соединиться с другой базой:
```python
	$ psql -d postgres
```
Проверить текущее соединение:
```python
	sammy=# \conninfo
```
```python
Output
You are connected to database "sammy" as user "sammy" via socket in "/var/run/postgresql" at port "5432".
```
## Установка и настройка Docker
Апдейт пакетов:
```python
	$ sudo apt update
```
Установка нескольких необходимых пакетов чтобы дать возможность apt использовать пакеты через HTTPS:
```python
	$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
Добавить GPG ключ для официального Docker репозитория в систему:
```python
	$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Добавить репозиторий Docker в APT ресурсы:
```python
	$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Снова обновить существующие пакеты, чтобы распознавались изменения:
```python
	$ sudo apt update
```
Убедиться, что установка прошла с репозитория Docker, а не с Ubuntu:
```python
	$ apt-cache policy docker-ce
```
Будет показан следующий вывод:
```python
Output of apt-cache policy docker-ce

docker-ce:
  Installed: (none)
  Candidate: 5:20.10.14~3-0~ubuntu-jammy
  Version table:
     5:20.10.14~3-0~ubuntu-jammy 500
        500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
     5:20.10.13~3-0~ubuntu-jammy 500
        500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
```
Docker еще не установлен, но кандидат из репозитория Docker для Ubuntu 22.04 (jammy).

Наконец, установить Docker:
```python
	$ sudo apt install docker-ce
```
Docker установлен, демон запущен. Проверить работу:
```python
	$ sudo systemctl status docker
```
Вывод будет таким:
```python
Output
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-04-01 21:30:25 UTC; 22s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 7854 (dockerd)
      Tasks: 7
     Memory: 38.3M
        CPU: 340ms
     CGroup: /system.slice/docker.service
             └─7854 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
Следующие шаги по работе с Docker описаны здесь:  
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

