Модуль Б. (Настройка технических и программных средств информационно-коммуникационных систем)

Время на выполнение модуля 5 часов.
Доступ к ISP вы не имеете!!

image
Название устройства 	ОС
RTR-HQ 	Eltex vESR
RTR-BR 	Eltex vESR
SRV-HQ 	Альт Сервер 10
SRV-BR 	Альт Сервер 10 (Допустима замена) Debian
CLI-HQ 	Альт Рабочая станция 10
CLI-BR 	Альт Рабочая станция 10 (Допустима замена)
SW-HQ 	Альт Сервер 10
SW-BR 	Альт Сервер 10 (Допустима замена)
CICD-HQ 	Альт Сервер 10
HQ
Подсеть/VLAN 	Префикс 	Диапазон 	Broadband 	Размер
10.0.10.0 / vlan100 	/27 	10.0.10.1-30 	10.0.10.31 	30
10.0.10.32 / vlan200 	/27 	10.0.10.33-62 	10.0.10.63 	30
10.0.10.64 / vlan300 	/27 	10.0.10.65-94 	10.0.10.95 	30
BR
Подсеть/VLAN 	Префикс 	Диапазон 	Broadband 	Размер
10.0.20.0 / vlan100 	/27 	10.0.20.1-30 	10.0.20.31 	30
10.0.20.32 / vlan200 	/27 	10.0.20.33-62 	10.0.20.63 	30
10.0.20.64 / vlan300 	/27 	10.0.20.65-94 	10.0.20.95 	30
Имя 	NIC 	IP 	Default Gateway
RTR-HQ 	ISP 	11.11.11.2/30 	11.11.11.1
	vlan100 	10.0.10.1/27 	
	vlan200 	10.0.10.33/27 	
	vlan300 	10.0.10.65/27 	
RTR-BR 	ISP 	22.22.22.2/30 	22.22.22.1
	vlan100 	10.0.20.1/27 	
	vlan200 	10.0.20.33/27 	
	vlan300 	10.0.20.65/27 	
SW-HQ 	vlan300 	10.0.10.66/27 	
SW-BR 	vlan300 	10.0.20.66/27 	
SRV-HQ 	vlan100 	10.0.10.2/27 	
SRV-BR 	vlan100 	10.0.20.2/27 	
CLI-HQ 	vlan200 	10.0.10.32/27 	
CLI-BR 	vlan200 	10.0.20.32/27 	
CICD-HQ 	vlan200 	10.0.10.3/27 	
Внеполосное управление виртуалками в Proxmox
ТЫКНИ
1. Базовая настройка

a) Настройте имена устройств согласно топологии
  a. Используйте полное доменное имя
  b. Сконфигурируйте адреса устройств на свое усмотрение. Для офиса HQ выделена сеть 10.0.10.0/24, для офиса BR выделена сеть 10.0.20.0/24. Данные сети необходимо разделить на подсети для каждого vlan.
  c. На SRV-HQ и SRV-BR, создайте пользователя sshuser с паролем P@ssw0rd
    1. Пользователь sshuser должен иметь возможность запуска утилиты sudo без дополнительной аутентификации.
    2. Запретите парольную аутентификацию. Аутентификация пользователя sshuser должна происходить только при помощи ключей.
    3. Измените стандартный ssh порт на 2023.
    4. На CLI-HQ сконфигурируйте клиент для автоматического подключения к SRV-HQ и SRV-BR под пользователем sshuser. При подключении автоматически должен выбираться корректный порт. Создайте пользователя sshuser на CLI-HQ для обеспечения такого сетевого доступа.
ТЫКНИ
RTR-HQ

Полное доменное имя:

config
hostname rtr-hq.company.prof
do commit
do confirm

Команды для просмотра интерфейсов:

sh ip int
sh int stat

Настройка ip-адреса (ISP-HQ):

interface te1/0/2
ip address 11.11.11.2/30
description ISP
exit

Параметры DNS:

domain name company.prof
domain name-server 11.11.11.1

Настройка шлюза:

ip route 0.0.0.0/0 11.11.11.1

Проверка доступа в Интернет:

image

Настройка подынтерфейсов (Router-on-a-stick) vlan100:

interface te1/0/3.100
ip firewall disable
description vlan100
ip address 10.0.10.1/27
exit

vlan200:

interface te1/0/3.200
ip firewall disable
description vlan200
ip address 10.0.10.33/27
exit

vlan300:

interface te1/0/3.300
ip firewall disable
description vlan300
ip address 10.0.10.65/27
exit

Сохранение:

do commit
do confirm

image

image
RTR-BR

То же самое, кроме IP-адреса:

image

image

image
SW-HQ | SRV-HQ | CLI-HQ | CICD-HQ | SW-BR | SRV-BR | CLI-BR

Полное доменное имя:

hostnamectl set-hostname <VM_NAME>.company.prof;exec bash

Настройка интерфейсов:

sed -i 's/DISABLED=yes/DISABLED=no/g' /etc/net/ifaces/ens18/options
sed -i 's/NM_CONTROLLED=yes/NM_CONTROLLED=no/g' /etc/net/ifaces/ens18/options
echo 10.0.20.2/24 > /etc/net/ifaces/ens18/ipv4address
echo default via 10.0.20.1 > /etc/net/ifaces/ens18/ipv4route
systemctl restart network
ping -c 4 8.8.8.8

SRV-HQ | SRV-BR

Создание пользователя sshuser:

adduser sshuser
passwd sshuser
P@ssw0rd
P@ssw0rd
echo "sshuser ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
usermod -aG wheel sshuser
sudo -i

SSH

На серверах, к которым подключаемся:

mkdir /home/sshuser

chown sshuser:sshuser /home/sshuser

cd /home/sshuser

mkdir -p .ssh/

chmod 700 .ssh

touch .ssh/authorized_keys

chmod 600 .ssh/authorized_keys

chown sshuser:sshuser .ssh/authorized_keys

На машине с которого подключаемся к серверам:

ssh-keygen -t rsa -b 2048 -f srv_ssh_key

mkdir .ssh

mv srv_ssh_key* .ssh/

Конфиг для автоматического подключения .ssh/config:

Host srv-hq
        HostName 10.0.10.2
        User sshuser
        IdentityFile .ssh/srv_ssh_key
        Port 2023
Host srv-br
        HostName 10.0.20.2
        User sshuser
        IdentityFile .ssh/srv_ssh_key
        Port 2023

chmod 600 .ssh/config

Копирование ключа на удаленный сервер:

ssh-copy-id -i .ssh/srv_ssh_key.pub sshuser@10.0.10.2

ssh-copy-id -i .ssh/srv_ssh_key.pub sshuser@10.0.20.2

На сервере /etc/ssh/sshd_config:

AllowUsers sshuser
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
AuthorizedKeysFile .ssh/authorized_keys
Port 2023

systemctl restart sshd

Подключение:

ssh srv-hq

2. Настройка дисковой подсистемы

a) На SRV-HQ настройте зеркалируемый LVM том
  a. Используйте два неразмеченных жестких диска.
  b. Настройте автоматическое монтирование логического тома.
  c. Точка монтирования /opt/data.
b) На SRV-BR сконфигурируйте stripped LVM том.
  a. Используйте два неразмеченных жестких диска.
  b. Настройте автоматическое монтирование тома.
  c. Обеспечьте шифрование тома средствами dm-crypt. Диск должен монтироваться при загрузке ОС без запроса пароля.
  d. Точка монтирования /opt/data.
ТЫКНИ
SRV-HQ

Создаем разделы:

gdisk /dev/sdb

o - y
n - 1 - enter - enter - 8e00
p
w - y
То же самое с sdc
Инициализация:

pvcreate /dev/sd{b,c}1

В группу:

vgcreate vg01 /dev/sd{b,c}1

Логический том-зеркало RAID1:

lvcreate -l 100%FREE -n lvmirror -m1 vg01

Создаём ФС:

mkfs.ext4 /dev/vg01/lvmirror

Директория:

mkdir /opt/data

Автозагрузка /etc/fstab:

echo "UUID=$(blkid -s UUID -o value /dev/vg01/lvmirror) /opt/data ext4 defaults 1 2" | tee -a /etc/fstab

Монтаж:

mount -av

Проверка:

df -h

SRV-BR

Создаем разделы:

gdisk /dev/sdb

o - y
n - 1 - enter - enter - 8e00
p
w - y
То же самое с sdc
Инициализация:

pvcreate /dev/sd{b,c}1

В группу:

vgcreate vg01 /dev/sd{b,c}1

Striped том:

lvcreate -i2 -l 100%FREE -n lvstriped vg01

    -i2 - количество полос
    -l 100%FREE - занять всё место
    -n lvstriped - имя 'lvstriped'
    vg01 - имя группы

ФС:

mkfs.ext4 /dev/vg01/lvstriped

Создание ключа:

dd if=/dev/urandom of=/root/ext2.key bs=1 count=4096

Шифрование тома с помощью ключа:

cryptsetup luksFormat /dev/vg01/lvstriped /root/ext2.key

Открытие тома ключом:

cryptsetup luksOpen -d /root/ext2.key /dev/vg01/lvstriped lvstriped

ФС на расшифрованном томе:

mkfs.ext4 /dev/mapper/lvstriped

Директория монтирования:

mkdir /opt/data

Автозагрузка /etc/fstab

echo "UUID=$(blkid -s UUID -o value /dev/mapper/lvstriped) /opt/data ext4 defaults 0 0" | tee -a /etc/fstab

Монтаж:

mount -av

Автозагрузка зашифрованного тома с помощью ключа /etc/crypttab:

echo "lvstreped UUID=$(blkid -s UUID -o value /dev/vg01/lvstriped) /root/ext2.key luks" | tee -a /etc/crypttab

reboot, df -h, lsblk
3. Настройка коммутации

a) В качестве коммутаторов используются SW-HQ и SW-BR.
b) В обоих офисах серверы должны находиться во vlan100, клиенты – во vlan200, management подсеть – во vlan300.
c) Создайте management интерфейсы на коммутаторах.
d) Для каждого vlan рассчитайте подсети, выданные для офисов. Количество хостов в каждой подсети не должно превышать 30-ти.
ТЫКНИ
SW-HQ
Внимание! в интерфейсах ens... не должно быть прочих файлов, кроме options, иначе порты не привяжутся

Временное назначение ip-адреса (смотрящего в сторону rtr-hq):

ip link add link ens18 name ens18.300 type vlan id 300
ip link set dev ens18.300 up
ip addr add 10.0.10.66/27 dev ens18.300
ip route add 0.0.0.0/0 via 10.0.10.65
echo nameserver 8.8.8.8 > /etc/resolv.conf

Обновление пакетов и установка openvswitch:

apt-get update && apt-get install -y openvswitch

Автозагрузка:

systemctl enable --now openvswitch

ens18 - rtr-hq
ens19 - cicd-hq - vlan200 ens20 - srv-hq - vlan100 ens21 - cli-hq - vlan200

Создаем каталоги для ens19,ens20,ens21:

mkdir /etc/net/ifaces/ens{19,20,21}

Для моста:

mkdir /etc/net/ifaces/ovs0

Management интерфейс:

mkdir /etc/net/ifaces/mgmt

Не удалять настройки openvswitch:

sed -i "s/OVS_REMOVE=yes/OVS_REMOVE=no/g" /etc/net/ifaces/default/options

Мост /etc/net/ifaces/ovs0/options:

image

    TYPE - тип интерфейса, bridge;
    HOST - добавляемые интерфейсы в bridge.

mgmt /etc/net/ifaces/mgmt/options:

image

    TYPE - тип интерфейса (internal);
    BOOTPROTO - статически;
    CONFIG_IPV4 - использовать ipv4;
    BRIDGE - определяет к какому мосту необходимо добавить данный интерфейс;
    VID - определяет принадлежность интерфейса к VLAN.

Поднимаем интерфейсы:

cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens19/options
cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens20/options
cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens21/options

Назначаем Ip, default gateway на mgmt:

echo 10.0.10.66/27 > /etc/net/ifaces/mgmt/ipv4address

echo default via 10.0.10.65 > /etc/net/ifaces/mgmt/ipv4route

Перезапуск network:

systemctl restart network

Проверка:

ip -c --br a

ovs-vsctl show

ens18 - rtr-hq делаем trunk и пропускаем VLANs:

ovs-vsctl set port ens18 trunk=100,200,300

ens19 - tag=200

ovs-vsctl set port ens19 tag=200

ens20 - tag=100:

ovs-vsctl set port ens20 tag=100

ens21 - tag=200

ovs-vsctl set port ens21 tag=200

Включаем инкапсулирование пакетов по 802.1q:

modprobe 8021q

Проверка включения

image

image
4. Установка и настройка сервера баз данных

a) В качестве серверов баз данных используйте сервера SRV-HQ и SRV-BR
b) Разверните сервер баз данных на базе Postgresql
  a. Создайте базы данных prod, test, dev
    i. Заполните базы данных тестовыми данными при помощи утилиты pgbench. Коэффицент масштабирования сохраните по умолчанию.
  b. Разрешите внешние подключения для всех пользователей.
  c. Сконфигурируйте репликацию с SRV-HQ на SRV-BR
  d. Обеспечьте отказоустойчивость СУБД при помощи HAProxy.
    i. HAProxy установите на SW-HQ.
    ii. Режим балансировки – Hot-Standby: Активным необходимо сделать только SRV-HQ. В случае отказа SRV-HQ активным сервером должен становится SRV-BR.
    iii. Выбор standby режима (RO/RW) остается на усмотрение участника.
    iv. Обеспечьте единую точку подключения к СУБД по имени dbms.company.prof
ТЫКНИ
SRV-HQ

Установка postgresql:

apt-get install -y postgresql16 postgresql16-server postgresql16-contrib

Развертываем БД:

/etc/init.d/postgresql initdb

Автозагрузка:

systemctl enable --now postgresql

Включаем прослушивание всех адресов:

nano /var/lib/pgsql/data/postgresql.conf

изображение

Перезагружаем:

systemctl restart postgresql

Проверяем:

ss -tlpn | grep postgres

Заходим в БД под рутом:

psql -U postgres

Создаем базы данных "prod","test","dev":

CREATE DATABASE prod;

CREATE DATABASE test;

CREATE DATABASE dev;

Создаем пользователей "produser","testuser","devuser":

CREATE USER produser WITH PASSWORD 'P@ssw0rd';

CREATE USER testuser WITH PASSWORD 'P@ssw0rd';

CREATE USER devuser WITH PASSWORD 'P@ssw0rd';

Назначаем на БД владельца:

GRANT ALL PRIVILEGES ON DATABASE prod to produser;

GRANT ALL PRIVILEGES ON DATABASE test to testuser;

GRANT ALL PRIVILEGES ON DATABASE dev to devuser;

Заполняем БД с помощью pgbench:

pgqbench -U postgres -i prod

pgqbench -U postgres -i test

pgqbench -U postgres -i dev

Проверяем

изображение

Настройка аутентификации для удаленного доступа:

nano /var/lib/pgsql/data/pg_hba.conf

изображение

Перезапускаем:

systemctl restart postgresql

SRV-BR

Установка:

apt-get install -y postgresql16 postgresql16-server postgresql16-contrib

image
Репликация
SRV-HQ

Конфиг

nano /var/lib/pgsql/data/postgresql.conf

wal_level = replica
max_wal_senders = 2
max_replication_slots = 2
hot_standby = on
hot_standby_feedback = on

    wal_level указывает, сколько информации записывается в WAL (журнал операций, который используется для репликации); max_wal_senders — количество планируемых слейвов; max_replication_slots — максимальное число слотов репликации; hot_standby — определяет, можно ли подключаться к postgresql для выполнения запросов в процессе восстановления; hot_standby_feedback — определяет, будет ли сервер slave сообщать мастеру о запросах, которые он выполняет.

systemctl restart postgresql

SRV-BR

Останавливаем:

systemctl stop postgresql

Удаляем каталог:

rm -rf /var/lib/pgsql/data/*

Запускаем репликацию (может длиться несколько десятков минут):

pg_basebackup -h 10.0.10.2 -U postgres -D /var/lib/pgsql/data --wal-method=stream --write-recovery-conf

image

Назначаем владельца:

chown -R postgres:postgres /var/lib/pgsql/data/

Запускаем:

systemctl start postgresql

SRV-HQ

Создаем тест:

psql -U postgres

CREATE DATABASE testik;

SRV-BR

image
HAPROXY
SW-HQ (test, not prod)

Установка haproxy:

apt-get install haproxy

Автозагрузка:

systemctl enable --now haproxy

Конфиг /etc/haproxy/haproxy.cfg:

listen stats
    bind 0.0.0.0:8989
    mode http
    stats enable
    stats uri /haproxy_stats
    stats realm HAProxy\ Statistics
    stats auth admin:toor
    stats admin if TRUE

frontend postgre
    bind 0.0.0.0:5432
    default_backend my-web

backend postgre
    balance first
    server srv-hq 10.0.10.2:5432 check
    server srv-br 10.0.20.2:5432 check

5. Настройка динамической трансляции адресов

a) Настройте динамическую трансляцию адресов для обоих офисов. Доступ к интернету необходимо разрешить со всех устройств.
ТЫКНИ

rtr-hq:

config
security zone public
exit
int te1/0/2
security-zone public
exit
object-group network COMPANY
ip address-range 10.0.10.1-10.0.10.254
exit
object-group network WAN
ip address-range 11.11.11.11
exit
nat source
pool WAN
ip address-range 11.11.11.11
exit
ruleset SNAT
to zone public
rule 1
match source-address COMPANY
action source-nat pool WAN
enable
exit
exit
commit
confirm

rtr-br:

config
security zone public
exit
int te1/0/2
security-zone public
exit
object-group network COMPANY
ip address-range 10.0.20.1-10.0.20.254
exit
object-group network WAN
ip address-range 22.22.22.22
exit
nat source
pool WAN
ip address-range 22.22.22.22
exit
ruleset SNAT
to zone public
rule 1
match source-address COMPANY
action source-nat pool WAN
enable
exit
exit
commit
confirm

6. Настройка протокола динамической конфигурации хостов

a) Настройте протокол динамической конфигурации хостов для устройств в подсетях CLI - RTR-HQ
  i. Адрес сети – согласно топологии
  ii. Адрес шлюза по умолчанию – адрес маршрутизатора RTR-HQ
  iii. DNS-суффикс – company.prof
b) Настройте протокол динамической конфигурации хостов для устройств в подсетях CLI RTR-BR
  i. Адрес сети – согласно топологии
  ii. Адрес шлюза по умолчанию – адрес маршрутизатора RTR-BR
  iii. DNS-суффикс – company.prof
ТЫКНИ
RTR-HQ

DNS-сервер - srv-hq

configure terminal
ip dhcp-server pool COMPANY-HQ
network 10.0.10.32/27
default-lease-time 3:00:00
address-range 10.0.10.33-10.0.10.62
excluded-address-range 10.0.10.33                      
default-router 10.0.10.33 
dns-server 10.0.10.2
domain-name company.prof
exit

Включение DHCP-сервера:

ip dhcp-server

image

image

image

image
7. Настройка DNS для SRV-HQ и SRV-BR

i. Реализуйте основной DNS сервер компании на SRV-HQ
  a. Для всех устройств обоих офисов необходимо создать записи A и PTR.
  b. Для всех сервисов предприятия необходимо создать записи CNAME.
  c. Создайте запись test таким образом, чтобы при разрешении имени из левого офиса имя разрешалось в адрес SRV-HQ, а из правого – в адрес SRV-BR.
  d. Сконфигурируйте SRV-BR, как резервный DNS сервер. Загрузка записей с SRV-HQ должна быть разрешена только для SRV-BR.
  e. Клиенты предприятия должны быть настроены на использование внутренних DNS серверов
ТЫКНИ
SRV-HQ

Установка bind и bind-utils:

apt-get update && apt-get install -y bind bind-utils

Конфиг:

nano /etc/bind/options.conf

listen-on { any; };
allow-query { any; };
allow-transfer { 10.0.20.2; }; 

изображение

Включаем resolv:

nano /etc/net/ifaces/ens18/resolv.conf

systemctl restart network

Автозагрузка bind:

systemctl enable --now bind

Создаем прямую и обратные зоны:

nano /etc/bind/local.conf

изображение

Копируем дефолты:

cp /etc/bind/zone/{localhost,company.db}

cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/10.0.10.in-addr.arpa.db

cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/20.0.10.in-addr.arpa.db

Назначаем права:

chown root:named /etc/bind/zone/company.db

chown root:named /etc/bind/zone/10.0.10.in-addr.arpa.db

chown root:named /etc/bind/zone/20.0.10.in-addr.arpa.db

Настраиваем зону прямого просмотра /etc/bind/zone/company.prof:

изображение

Настраиваем зону обратного просмотра /etc/bind/zone/10.0.10.in-addr.arpa.db:

изображение

Настраиваем зону обратного просмотра /etc/bind/zone/20.0.10.in-addr.arpa.db:

изображение

Проверка зон:

named-checkconf -z

image

image

image
SRV-BR

Конфиг

vim /etc/bind/options.conf

image

Добавляем зоны

image

Резолв /etc/net/ifaces/ens18/resolv.conf:

search company.prof
nameserver 10.0.10.2
nameserver 10.0.20.2

Перезапуск адаптера:

systemctl restart network

Автозагрузка:

systemctl enable --now bind

SLAVE:

control bind-slave enabled

image

Разрешение имени хоста test
SRV-HQ

image

Копируем дефолт для зоны:

cp /etc/bind/zone/{localdomain,test.company.db}

Задаём права, владельца:

chown root:named /etc/bind/zone/test.company.db

Настраиваем зону:

vim /etc/bind/zone/test.company.db

image

image

Перезапускаем:

systemctl restart bind

SRV-BR

Добавляем зону /etc/bind/local.conf:

image

Задаём права, владельца:

chown root:named /etc/bind/zone/test.company.db

Редактируем зону /etc/bind/zone/test.company.db:

image

Перезапускаем:

systemctl restart bind

8. Настройка узла управления Ansible

a) Настройте узел управления на базе SRV-BR
  a. Установите Ansible.
b) Сконфигурируйте инвентарь по пути /etc/ansible/inventory. Инвентарь должен содержать три группы устройств:
  a. Networking
  b. Servers
  c. Clients
c) Напишите плейбук в /etc/ansible/gathering.yml для сбора информации об IP адресах и именах всех устройств (и клиенты, и серверы, и роутеры). Отчет должен быть сохранен в /etc/ansible/output.yaml, в формате ПОЛНОЕ_ДОМЕННОЕ_ИМЯ – АДРЕС
ТЫКНИ

Установка:

apt-get install -y ansible sshpass

nano /etc/ansible/inventory.yml

Networking:
  hosts:
    rtr-hq:
      ansible_host: 10.0.10.1
      ansible_user: admin
      ansible_password: P@ssw0rd
      ansible_port: 22
    rtr-br:
      ansible_host: 10.0.20.1
      ansible_user: admin
      ansible_password: P@ssw0rd
      ansible_port: 22
Servers:
  hosts:
    srv-hq:
      ansible_host: 10.0.10.2
      ansible_ssh_private_key_file: ~/.ssh/srv_ssh_key
      ansible_user: sshuser
      ansible_port: 2023
    srv-br:
      ansible_host: 10.0.20.2
      ansible_ssh_private_key_file: ~/.ssh/srv_ssh_key
      ansible_user: sshuser
      ansible_port: 22
Clients:
  hosts:
    cli-hq:
      ansible_host: 10.0.10.34
      ansible_user: root
      ansible_password: P@ssw0rd
      ansible_port: 22
    cli-br:
      ansible_host: 10.0.20.34
      ansible_user: root
      ansible_password: P@ssw0rd
      ansible_port: 22

nano /etc/ansible/ansible.cfg

image

nano /etc/ansible/gathering.yml

---
- name: show ip and hostname from routers
  hosts: Networking
  gather_facts: false
  tasks:
  - name: show ip
    esr_command:
      commands: show ip interfaces

- name: show ip and fqdn
  hosts: Servers, Clients
  tasks:
  - name: print ip and hostname
    debug:
      msg: "IP address: {{ ansible_default_ipv4.address }}, Hostname {{ ansible_hostname }}"
  - name: create file
    copy:
      content: ""
      dest: /etc/ansible/output.yml
      mode: '0644'

  - name: save output
    copy:
      content: "IP address: {{ ansible_default_ipv4.address }}, Hostname {{ ansible_hostname }}"
      dest: /etc/ansible/output.yml

cd /etc/ansible

image

image
9. Между маршрутизаторами RTR-HQ и RTR-BR сконфигурируйте защищенное соединение

a) Все параметры на усмотрение участника.
b) Используйте парольную аутентификацию.
c) Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса.
d) Для обеспечения динамической маршрутизации используйте протокол OSPF.

https://sysahelper.gitbook.io/sysahelper/main/telecom/main/vesr_greoveripsec
ТЫКНИ

tunnel gre 1
  ttl 16
  ip firewall disable
  local address 11.11.11.11
  remote address 22.22.22.22
  ip address 172.16.1.1/30
  enable
exit

security zone-pair public self
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
  rule 2
    description "GRE"
    action permit
    match protocol gre
    enable
  exit
exit

security ike proposal ike_prop1
  authentication algorithm md5
  encryption algorithm aes128
  dh-group 2
exit

security ike policy ike_pol1
  pre-shared-key ascii-text P@ssw0rd
  proposal ike_prop1
exit

security ike gateway ike_gw1
  ike-policy ike_pol1
  local address 11.11.11.11
  local network 11.11.11.11/32 protocol gre 
  remote address 22.22.22.22
  remote network 22.22.22.22/32 protocol gre 
  mode policy-based
exit

security ike gateway ike_gw1
  ike-policy ike_pol1
  local address 22.22.22.22
  local network 22.22.22.22/32 protocol gre 
  remote address 11.11.11.11
  remote network 11.11.11.11/32 protocol gre 
  mode policy-based
exit

security ipsec proposal ipsec_prop1
  authentication algorithm md5
  encryption algorithm aes128
  pfs dh-group 2
exit

security ipsec policy ipsec_pol1
  proposal ipsec_prop1
exit

security ipsec vpn ipsec1
  ike establish-tunnel route
  ike gateway ike_gw1
  ike ipsec-policy ipsec_pol1
  enable
exit

security zone-pair public self
  rule 3
    description "ESP"
    action permit
    match protocol esp
    enable
  exit
  rule 4
    description "AH"
    action permit
    match protocol ah
    enable
  exit
exit

router ospf 1
  area 0.0.0.0
    enable
  exit
  enable
exit

tunnel gre 1
  ip ospf instance 1
  ip ospf
exit

interface gigabitethernet 1/0/2.100
  ip ospf instance 1
  ip ospf
exit

interface gigabitethernet 1/0/2.200
  ip ospf instance 1
  ip ospf
exit

interface gigabitethernet 1/0/2.300
  ip ospf instance 1
  ip ospf
exit

security zone-pair public self
  rule 5
    description "OSPF"
    action permit
    match protocol ospf
    enable
  exit
exit

Проверка

sh ip ospf neighbors

sh ip route

image

image
10. -> На сервере SRV-HQ сконфигурируйте основной доменный контроллер на базе FreeIPA

a) Создайте 30 пользователей user1-user30.
b) Пользователи user1-user10 должны входить в состав группы group1.
c) Пользователи user11-user20 должны входить в состав группы group2.
d) Пользователи user21-user30 должны входить в состав группы group3.
e) Разрешите аутентификацию с использованием доменных учетных данных на ВМ CLI-HQ.
f) Установите сертификат центра сертификации FreeIPA в качестве доверенного на обоих клиентских ПК.
ТЫКНИ

->
11. На SRV-BR сконфигурируйте proxy-сервер со следующими параметрами

a) Пользователям group1 разрешен доступ на любые сервисы предприятия
b) Пользователям group2 разрешен доступ только к системе мониторинга
c) Пользователям group3 не разрешен доступ никуда, также, как и пользователям, не прошедшим аутентификацию
d) Любым пользователям компьютера CLI-HQ разрешен доступ в сеть Интернет и на все сервисы предприятия, кроме доменов vk.com, mail.yandex.ru и worldskills.org
e) Настройте клиент правого офиса на использование прокси сервера предприятия
f) Авторизация для proxy спрашивается браузером, SSO не ожидается
Запись 	Тип записи
rtr-hq.company.prof 	A
rtr-br.company.prof 	A
sw-hq.company.prof 	A
sw-br.company.prof 	A
srv-hq.company.prof 	A
srv-br.company.prof 	A
cli-hq.company.prof 	A
cli-br.company.prof 	A
dbms.company.prof 	CNAME
ТЫКНИ
SRV-BR
SRV-HQ

Установка:

apt-get install alterator-squid

Доступ осуществляется по https://ip-address:8080/

image
