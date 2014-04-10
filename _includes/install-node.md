#Look Ma! No content here!
Теперь ставим NodeJS. Идем от лица свободного пользователя и ставим:

mkdir tmp && cd tmp
wget http://nodejs.org/dist/v0.10.26/node-v0.10.26.tar.gz
tar -zxvf node-v0.10.26.tar.gz
cd node-v0.10.26/
./configure
make
sudo make install

jk_cp -v -f -j /home/jail node
jk_cp -v -f -j /home/jail npm

jk_cp -v -f -j /home/jail env


или вовсе возьмем готовый бинарник

cd /home/jail/usr/local
sudo tar xzvf --strip-components=1 node-v0.10.26.tar.gz

теперь внутри jail

npm config set prefix /home/jailed_user/npm
npm install forever -g

возможно будет необходимо примонтировать служебные сисетмы
/dev
/proc
/sys


sudo aptitude install git
sudo aptitude install curl

jk_cp -v -f -j /home/jail git
jk_cp -v -f -j /home/jail curl
