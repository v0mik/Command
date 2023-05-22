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

