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


