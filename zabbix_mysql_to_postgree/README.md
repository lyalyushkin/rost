# Миграция базы данных Zabbix MySQL на Postgree
## Устанвливаем Необходимые пакеты
Из пакетов установить
```
$ sudo apt-get install postgresql
$ sudo apt-get install php8.1-pgsql
$ sudo systemctl restart php8.1-fpm.service
```
Скачать исходники Zabbix
```
$ mkdir zabbix-mysql-to-pgsql
$ cd zabbix-mysql-to-pgsql
$ wget -c https://cdn.zabbix.com/zabbix/sources/stable/6.4/zabbix-6.4.4.tar.gz
```
Скчать исходнки pgloader и скомпилировать [](https://github.com/pretix/pretix/issues/3109)
```
$ git clone https://github.com/dimitri/pgloader.git
$ cd pgloader/
$ sudo apt install sbcl
$ make
```
