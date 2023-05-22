# C0mm@nd

# WEB-L, WEB-R, ISP, RTR-L, RTR

hostnamectl set-hostname NAME - Команда для настройки имени

hostnamectl - Проверка

# SRV-Win

Заходим в проводник – This PC – в пустом месте правой кнопкой – Properties – Change Settings – Change – Computer Name – устанавливаем SRV-Win

или 'PowerShell командой Rename-Computer -NewName SRV'

CLI аналогично 

или через 'PowerShell командой Rename-Computer -NewName CLI'

# SRV - Win 

Settings – Network and Internet – Change adapter options – Ethernet 0 – Properties – Internet Protocol version 4 – Use the following IP address – OK. Заполняется по таблице.

Далее: Стрелка вверх – Network and Sharing Center – Change advanced sharing settings – Guest or Public – File and printer sharing – Turn on file and printer sharing – Save changes

CLI аналогично заполнять исходя из данных в таблице.

# WEB-R, WEB-L

устанавливаем диски

apt-cdrom add - Подключение диска в файловую систему

apt install -y network-manager - Установка приложений диспетчера сети

nmcli connection show - показ активных подключений

nmcli connection change Wired\ connection\ 1 .conn.autoconnect yes conn.interface-name ens192 ipv4.method manual ipv4.addresses 'IP-адрес WEB-R-L' ipv4.dns IP-адрес SRV ipv4.gateway IP-адрес WEB-R-L .254 

ifconfig 

Web-L аналогично

# ISP 

apt-cdrom add

apt install -y network-manager 

nmcli connection show

Данные для заполнения все написаны в таблице. Пример ниже

nmcli connection modify Wired\ connection\ 1 conn.autoconnect yes conn.interface-name ens192 ipv4.method manual ipv4.addresses '3.3.3.1/24' – настройка 1 интерфейса

nmcli connection modify Wired\ connection\ 2 conn.autoconnect yes conn.interface-name ens224 ipv4.method manual ipv4.addresses '4.4.4.1/24' - настройка 2 интерфейса

nmcli connection modify Wired\ connection\ 3 conn.autoconnect yes conn.interface-name ens256 ipv4.method manual ipv4.addresses '5.5.5.1/24' - настройка 3 интерфейса

# ISP forward

nano /etc/sysctl.conf

net.ipv4.ip_forward=1 - НУЖНО РАСКОММЕНТИРОВАТЬ строчку net.ipv4…, ctrl+O (сохранить), ctrl+x, enter

sysctl -p

# SSH WEB-L, SSH WEB-R

apt-cdrom add – подключение дисковода

apt install -y openssh-server ssh – установка компонента openssh-server

systemctl start sshd – включение данного компонента

systemctl enable ssh – включение протокола ssh

# ISP

Подключить диск

apt-cdrom add – подключаем cdrom

apt install -y bind9 – установка компонента bind9

mkdir /opt/dns – создание новой директории

cp /etc/bind/db.local /opt/dns/demo.db – копирование файла в указанную директорию (dns)

chown -R bind:bind /opt/dns – перенастраивает права расположения и доступа на указанную директорию

nano /etc/apparmor.d/usr.sbin.named – открываем файл в текст.редакторе

затем после строчки /var/cache/bind/ rw надо добавить строчку

/opt/dns/** rw, - для того, чтобы система имела доступ к записи в данной директории 

Ctrl+o Enter сохранить ctrl+x выйти их текст.файла

systemctl restart apparmor.service

nano /etc/bind/named.conf.options – открываем в текст.редакторе файл, который отвечает за настройку dns-зон

меняем строчку forwarders 0.0.0.0 на 4.4.4.100 (адрес берем из таблицы – ip-адрес из сети ISP) затем в строке dnssec-validation меняем auto на no – проверка при подключении к dns отключается и добавляем строку allow-query {any;}; - все подключения в очереди разрешены

Ctrl+o Enter сохранить ctrl+x выйти их текст.файла

nano /etc/bind/named.conf.default-zones – открываем файл, который отвечает за нахождение базы данных dns-зон

в нем изменяем zone “localhost” на zone "demo.wsr", добавляем строчку allow-transfer { any; }  и меняем расположение и название файла (см.ниже):

zone "demo.wsr" {

type master;
   
allow-transfer { any; };
   
file "/opt/dns/demo.db";
   
};

Ctrl+o Enter сохранить ctrl+x выйти их текст.файла

nano /opt/dns/demo.db – открываем сам файл с БД, меняем localhost на данные из таблицы (название dns-зоны, где она расположена, новые ip-адреса из таблицы, а также прописыаем настройки интерфейсов):

@ IN SOA demo.wsr. root.demo.wsr.(

@ IN NS isp.demo.wsr.

isp IN A 3.3.3.1 (менять исходя по данным из таблицы)

www IN A 4.4.4.100 (аналогично)

www IN A 5.5.5.100 (аналогично)

internet CNAME isp.demo.wsr.

int IN NS rtr-l.demo.wsr.

rtr-l IN  A 4.4.4.100 (менять исходя из задания)

systemctl restart bind9 – перезапуск компонента bind9

# SRV

Server Manager - Add roles and features - Next до окна Server Roles - Выбираем DNS-Server - Add Features - Next до конца - Install - Перезагрузка 

Server Manager - слева выбираем DNS - в разделе Servers на имени сервера правой кнопкой -  DNS Manager - Action - New Zone - Primary Zone - Next - вводим int.demo.wsr - Next - вводим int.demo.wsr.dns - Next - Finish 

Заходим в int.demo.wsr - в левой части по нему нажимаем правой кнопкой

Выбираем New Host, вносим имя и ip (которые давали в таблице). Пример NAME=>web-l; IP=>192.168.100.100

Затем в Revers Lookup Zones – правой кнопкой – New Zone - Next – Next – затем вводи ip сети: их 2 WEB-R и WEB-L => Use... int.demo.wsr.dns

Возвращаемся в Forward Lookup Zones – int.demo.wsr.dns – правой кнопкой мыши – New Alias. Alias name: webapp1 - Browse - выбрать web-l, webapp2 - web-r, ntp-srv, dns-srv.

# ISP NTP

apt install -y chrony – установка компонента chrony (синхронизация времени)

nano /etc/chrony/chrony.conf – открываем в текст.редакторе, добавляем после строчки pool 2.debian…………… iburst:

local stratum 4

allow 4.4.4.0/24

allow 3.3.3.0/24

Ctrl+o  enter  ctrl+x

systemctl restart chronyd 

# SRV NTP 

Заходим на SRV-Win – Control Panel – Windows Defender Firewall – Advanced Settings – Inbound Rules – New Rule (справа) – Port – UDP – Specific 123 - Next - Next - ВСЕ ГАЛЛОЧКИ - Next - Name: NTP

PowerShell

w32tm /query /status – проверка очереди и статуса сервиса W32Time(сервис синхронизации)

Start-Service W32Time – запуск сервиса W32Time

w32tm /config /manualpeerlist:4.4.4.1 /syncfromflags:manual /reliable:yes /update – настройка сервиса W32Time

Restart-Service W32Time – перезапуск сервиса W32Time

# CLI 

New-NetFirewallRule -DisplayName "NTP" -Direction Inbound -LocalPort 123 -Protocol UDP -Action Allow

Start-Service W32Time

w32tm /config /manualpeerlist:4.4.4.1 /syncfromflags:manual /reliable:yes /update

Restart-Service W32Time

Set-Service -Name W32Time -StartupType Automatic

# WEB-L

apt-cdrom add – прикрепление CD-ROM

apt install -y chrony – установка компонента chrony(компонент для настройки синхронизации)

nano /etc/chrony/chrony.conf – открытие в текст.редакторе файла с настройками chrony

закомментировать строчку pool 2.debian…………….. iburst

далее прописываем после строчки #pool 2.debian…………….. iburst:

pool ntp.int.demo.wsr iburst

allow 192.168.100.0/24

Ctrl+o Enter, Ctrl+x

systemctl restart chrony – перезапуск компонента chrony

# WEB-R

apt-cdrom add

apt install -y chrony 

nano /etc/chrony/chrony.conf

pool ntp.int.demo.wsr iburst

allow 192.168.100.0/24

systemctl restart chrony

# SRV RAID1

В поиске найти Computer Management – Disk Management -правой кнопкой по Disk 1/2 – выбрать пункт Online

Далее закрыть данное окно и заново открыть. Выбираем наши диски и разметку GPT и нажимаем ОК.

Далее нажимаем на Disk 1 и выбираем пункт New Mirrored Volume.

Далее в диалоговом окне нажимаем на Disk 2 и Add>, далее Next.

Выбираем букву для нашего раздела. Next.

Назначаем имя и next. Далее проверяем наши настройки и next.

На предупреждение конвертации диска нажимаем Yes.

Server Manager -> File and Storage Services -> Shares -> Start the Add Roles and Features Wizard

Next

Next

File and Storage Service

Next

Ставим галочку Restart the destination server automatically if required. Соглашаемся с предупреждением. Нажимаем Install.

Install

Ждём Feature installation. Close. Перезапускаем сервер.

Server Manager -> File and Storage Services -> Shares -> TASKS ->NewShare или клик правой кнопкой-> New Share

Выбираем SMB Share – Quick и нажимаем Next.

Выбираем путь до нашей папки на новом разделе Type a custom path -> Browse…

Select Folder и нажимаем Next.

Далее задаем имя, лучше оставить такое же как и папка, и нажимаем Next.

Оставляем галочки на своих местах и нажимаем Next.

Нажимаем Next.

Проверяем настройки и нажимаем Create.

После завершения создания нажимаем Close.

Далее открываем проводник и заходим на новый раздел. Кликаем по нашей папке правой кнопкой и выбираем Properties, далее заходим во вкладку Sharing. Нажимаем на кнопку Share.

Выбираем из выпадающего списка Everyone и нажать Add.

Присваиваем Everyone права Read/Write и нажимаем Share.

Нажимаем Yes, turn on network discovery and file sharing for all public networks.

Нажимаем Done.

powerShell тоже самое что и выше 

get-disk

set-disk -Number 1 -IsOffline $false

set-disk -Number 2 -IsOffline $false

New-StoragePool -FriendlyName "POOLRAID1" -StorageSubsystemFriendlyName "Windows Storage*" -PhysicalDisks (Get-PhysicalDisk -CanPool $true)

New-VirtualDisk -StoragePoolFriendlyName "POOLRAID1" -FriendlyName "RAID1" -ResiliencySettingName Mirror -UseMaximumSize

Initialize-Disk -FriendlyName "RAID1"

New-Partition -DiskNumber 3 -UseMaximumSize -DriveLetter R

Format-Volume -DriveLetter R

Install-WindowsFeature -Name FS-FileServer -IncludeManagementTools

New-Item -Path R:\storage -ItemType Directory

New-SmbShare -Name "SMB" -Path "R:\storage" -FullAccess "Everyone"

# WEB-L 

apt-cdrom add – добавление диска

apt install -y cifs-utils – установка компонента cifs-utils

nano /root/.smbclient – файл с настройками smb

username=Administrator

password=Pa$$w0rd

nano /etc/fstab – файл с настройками местоположения папки

//srv.int.demo.wsr/share_folder /opt/share cifs user,rw,_netdev,credentials=/root/.smbclient 0 0

mkdir /opt/share – создание раздела с сетевыми папками

mount -a – присоединение

# WEB-R 

apt-cdrom add

apt install -y cifs-utils

nano /root/.smbclient

username=Administrator

password=Pa$$w0rd

nano /etc/fstab

//srv.int.demo.wsr/share_folder /opt/share cifs user,rw,_netdev,credentials=/root/.smbclient 0 0

mkdir /opt/share

mount -a

В Server Manager – справа вверху Manage – Add roles and Features – Next – Next – Next – в списках ролей выбираем Active Directory Certification Services – Add features – Next – Next – Next –Install

Если все ОК, слева в Server Manager должна появиться вкладка AD CS. Зайти туда.

Выбираем справа вверху Tasks – Add Roles and Features - Next – Next – Next – раскрыть первую роль – выбрать Certification Authority Web Enpollment – кнопка Add Features - Next – Next – Next – Next – Install – Close

В Server Manage нажимаем на флажок справа сверху и выбираем Configure Active Directory Certification Services

Next

Выбираем две роли Certification Authority и Certification Authority Web Enrollment

Выбираем Standalone CA - Next

Выбираем Root CA - Next

Create a new private key - next

Next

Называем в поле Common name for this CA: Demo.wsr и в Preview of distinguished name: CN=Demo.wsr

Next

Next

Configure
