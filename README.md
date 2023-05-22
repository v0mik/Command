# Command
Команды для настройки 
hostnamectl set-hostname NAME - Команда для настройки имени
hostnamectl - Проверка
apt-cdrom add - Подключение диска в файловую систему
apt install -y network-manager - Установка приложений диспетчера сети
nmcli connection show - показ активных подключений
nmcli connection change Wired\ connection\ 1 .conn.autoconnect yes conn.interface-name ens192 ipv4.method manual ipv4.addresses 'IP-адрес RTR-R-L' ipv4.dns IP-адрес SRV ipv4.gateway IP-адрес RTR-R-L .254 
