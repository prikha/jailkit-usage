#Установка NodeJS в Jail
##Теперь ставим NodeJS.

Идем от лица свободного пользователя и ставим:

`sudo aptitude install g++ build-essential`

Собираем и ставим ноду:

```sh
mkdir tmp && cd tmp
wget http://nodejs.org/dist/v0.10.26/node-v0.10.26.tar.gz
tar -zxvf node-v0.10.26.tar.gz
cd node-v0.10.26/
./configure
make
sudo make install
```

Теперь обеспечим себе полную работоспособность текущих установок.

```sh
jk_cp -v -f -j /home/jail node
jk_cp -v -f -j /home/jail npm
jk_cp -v -f -j /home/jail /usr/local/lib/dtrace/node.d
jk_cp -v -f -j /home/jail /usr/local/lib/node_modules
```

Тут оказалось что не лишним будет пробросить в джейл еще и `env`.

```sh
jk_cp -v -f -j /home/jail env
```

##Внутри Jail
Заходим в jail и настраиваем пути для установки модулей:

```sh
npm config set prefix /home/jailed_user/npm
```

теперь можно попробовать установить популярный супервизор `forever`.

```sh
npm install forever -g
```

Чтобы бинарник `forever` стал доступен нам через bash необходимо добавить новый путь в `$PATH`. Добавляем в конец `.bashrc`.

```
#jailed_user .bashrc
export PATH=~/npm/bin:$PATH
```

### Внимание!
Некоторые модули могут потребовать служебные директории:
```
/dev
/proc
/sys
```


