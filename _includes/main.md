##Bootstrapping
###Ставим чистую виртуалку Debian 7.4
Воспользуемся [Vagrant](http://vagrantup.com)

```sh
$ mkdir debian_with_jail
$ cd debian_with_jail/
$ vagrant init InFog/debian74_x64
$ vagrant up
```

Теперь нам необходимо слегка поправить Vagrantfile:
important! это надо разделить на два и своевременно вкинуть.

```ruby
...
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  ...
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.ssh.forward_agent = true
end
```

###Готовим jail на удаленном сервере

Заходим под обычным пользователем и ставим [jailkit](http://olivier.sessink.nl/jailkit/).

```sh
vagrant@debian$ mkdir tmp && cd tmp
vagrant@debian$ wget http://olivier.sessink.nl/jailkit/jailkit-2.17.tar.gz
vagrant@debian$ tar -zxvf jailkit-2.17.tar.gz
vagrant@debian$ cd jailkit-2.17
vagrant@debian$ ./configure
vagrant@debian$ make
vagrant@debian$ make install
```

Теперь нам нужна папка для подсистемы:

```
$ mkdir /home/jail
$ chown root:root /home/jail
```

Можно начинать наполнение системы. Есть примеры уже готовых конфигов в [/etc/jailkit/jk_init.ini](/jk_init.html)

пример из документации с пояснениями:

```sh
[jk_lsh]
comment = Jailkit limited shell
paths = /usr/sbin/jk_lsh, /etc/jailkit/jk_lsh.ini
users = root
groups = root
need_logsocket = 1
includesections = uidbasics

[sftp]
comment = ssh secure ftp with Jailkit limited shell
paths = /usr/lib/sftp-server
includesections = netbasics, uidbasics
devices = /dev/urandom, /dev/null
emptydirs = /svr
```


`paths` entry specifies which files and directories need to be copied into the jail. Executables and libraries are checked for any required libraries, and these requirements are copied too. All files are created with user root as owner.

`paths_w_owner` entry specifies which paths need to be copied with their current ownership. This can be used to copy files that need to be writable by a server process that does not run as user root (for example database files). The users and groups entries specify which users and groups that need to be present in jail /etc/passwd.

`need_logsocket` entry is set to "1" the jk_socketd.ini file is modified to include a /dev/log socket in this jail.

`devices` entry specifies which devices are required in the jail.

`includesections` entry specifies which other sections need to be processed as well when processing the current section. In the above example, the jk_lsh section is automatically included if the sftp section is processed.

`emptydirs` entry specifies which directories to create as empty directories. This can be useful to create for example mountpoints in the jail.


###Ставим нужные пакеты
С соответствующими правами:

```sh
$ jk_init -v /home/jail basicshell
$ jk_init -v /home/jail netutils
$ jk_init -v /home/jail editors
```

###Создаем пользователя

```sh
$ useradd -d /home/jailed_user -m jailed_user -s /bin/bash
$ passwd jailed_user
```

После чего мы проверяем `/etc/passwd`:

```sh
jailed_user:x:1001:1001::/home/jail/./home/jailed_user:/usr/sbin/jk_chrootsh
```

###Теперь прячем его в Jail

Тут начинается ***МАГИЯ***

```sh
$ jk_jailuser -m -j /home/jail jailed_user
```

`-m` говорит, чтобы скрипт пернес данные пользователя внурть тюрьмы

`-j` указывает в какой именно джейл его сажать

###Дадим нормальную консоль


проверяем и правим `/home/jail/etc/passwd`

```
jailed_user:x:1001:1001::/home/jailed_user:/bin/bash
```

проверяем и правим `/home/jail/etc/group`

```
jailed_user:x:1001:
```

###Недостающие папки смело создаем

```sh
$ mkdir /home/jail/tmp
$ chmod a+rwx /home/jail/tmp
```

```
$ mkdir /home/jail/opt
```

###Пытаемся войти

`ssh jailed_user@127.0.0.1 -p 2222`

и если дело валится то смотрим

`cat /var/log/auth.log`


