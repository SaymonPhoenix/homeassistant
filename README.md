# Установка Home Assistant
Переходим под пользователя root:
```bash
sudo -i
```
Обновляем все пакеты ОС:
```bash
apt update && apt -y upgrade
```
Обновляем прошивку:
```bash
rpi-update
```
Устанавливаем необходимые пакеты:
```bash
apt-get install -y jq wget curl udisks2 apparmor-utils libglib2.0-bin network-manager dbus systemd-journal-remote
```
Запускаем Network Manager:
```bash
systemctl enable NetworkManager --now
```
Добавляем русскую локаль и часовой пояс:
```bash
raspi-config
```
```
5 Localisation Options / I1 Change Locale - ищем и выбираем пробелом ru_RU.UTF-8 UTF-8
5 Localisation Options / I2 Change Timezone - выбираем часовой пояс
```
Делаем дополнительные настройки для устранения ошибок в НА:
```bash
sed -i 's/$/ systemd.unified_cgroup_hierarchy=false lsm=apparmor/' /boot/cmdline.txt
```
Устанавливаем скрипт управления вентилятором:
```bash
curl https://download.argon40.com/argon1.sh | bash
```
Если требуется, то меняем настройки по умолчанию:
```bash
argonone-config
```
Перезагружаем хост:
```bash
systemctl reboot
```
Переходим под пользователя root:
```bash
sudo -i
```
Устанавливаем docker:
```bash
curl -fsSL https://get.docker.com | sh
```
Устанавливаем OS-Agent (предварительно проверив, что пакет последней версии):
```bash
wget https://github.com/home-assistant/os-agent/releases/download/1.5.1/os-agent_1.5.1_linux_aarch64.deb && dpkg -i os-agent_1.5.1_linux_aarch64.deb
```
Устанавливаем Home Assisistant Supervised:
```bash
wget https://github.com/home-assistant/supervised-installer/releases/latest/download/homeassistant-supervised.deb && dpkg -i homeassistant-supervised.deb
```
Удаляем мусор:
```bash
rm *.deb
```
Выходим из под пользователя root:
```bash
exit
```
Добавляем в группу docker пользователя:
```bash
sudo gpasswd -a $USER docker && newgrp docker
```
Веб интерфейс Home Assistant доступен по адресу http://host:8123

---

# Обновление прошивки Zigbee стика SONOFF ZBDongle-E (на чипе EFR32MG21)
Переходим под пользователя root:
```bash
sudo -i
```
Обновляем все пакеты ОС:
```bash
apt update && apt -y upgrade
```
Устанавливаем необходимые пакеты:
```bash
apt-get install -y git python3-pip
```
Клонируем репозиторий с прошивками:
```bash
git clone https://github.com/xsp1989/zigbeeFirmware.git
```
Клонируем репозиторий с утилитой для прошивки:
```bash
git clone https://github.com/Elelabs/elelabs-zigbee-ezsp-utility.git
```
Копируем нужную версию файла прошивки (для примера использована версия 7.2.2.0_115200):
```bash
cp zigbeeFirmware/firmware/Zigbee3.0_Dongle-NoSigned/EZSP/ncp-uart-sw_7.2.2.0_115200.gbl elelabs-zigbee-ezsp-utility/ 
```
Переходм в папку с утилитой для прошивки:
```bash
cd elelabs-zigbee-ezsp-utility
```
Устанавливаем зависимости для работы утилиты для прошивки:
```bash
pip3 install -r requirements.txt
```
Проверяем на каком порту стик. В конце будет указано системное имя устройства (в моём случае ttyACM0):
```bash
ls -lah /dev/serial/by-path/ & ls -lah /dev/serial/by-id/
```
Для просмотра текущего состояния и версии прошивки стика нужно выполнить:
```bash
python3 Elelabs_EzspFwUtility.py probe -p /dev/ttyACM0
```
Для прошивки нужно выполнить:
```bash
python3 Elelabs_EzspFwUtility.py flash -f ncp-uart-sw_7.2.2.0_115200.gbl -p /dev/ttyACM0
```
Перезагружаем хост:
```bash
systemctl reboot
```

---

# Масска для распознавания команд Алисы для автоматизации:
"Сделай громче на 0(комната)(устройство)(+/- вкл/выкл)", где:
Комнаты:
  "!" - детская
  "@" - лоджия
  "#" - спальня
  "$" - гостиная
  "%" - кухня
  "^" - ванна
  "&" - коридор
  "*" - душ
Устройства:
  "без указания устройства" - основное освещение
  "!" - дополнительное освещение
  "@" - розетка
