## LCD ST7796S 

LCD экран 320x480 с контроллером fb st7796s, без тача\
В репозитории мои наработки под печать крепления к стандартному корпусу, исправленный dts-оверлей под ubuntu и внешний модуль ядра, провереные на Orange PI 3 LTS (Allwinner H6 Cortex-A53), Ubuntu 5.16

Для других устройств\операционок dts оверлей будет отличатся. 
В дальнейшем если будут какие-либо еще оверлеи под конкретные операционки, будут оставлены соответствующие пометки. Пока кроме как на ubuntu на других система проверить не было возможности.

Доработки драйвера FBTFT для поддержки контроллера st7796s взяты <a href="https://github.com/Sergey1560/fb_st7796s">здесь</a>

## Установка и настройка на ubuntu

1. Для сборки FBTFT модуля требуются linux-headers - оф. инструкция по сборке пакетов ядра (потребуется машина с Ubuntu 20) - <a href="http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_Zero_3#Linux_SDK.E2.80.94.E2.80.94orangepi-build_instruction">здесь</a>

Мой альтернативный нерациональный опыт получения заголовков описан в <i>long.txt</i>

sudo apt update\ 
sudo apt install build-essential


2. компилируем драйвер 

cd [путь до FBTFT]

make\
sudo make install\
make clean\
sudo depmod -A

3. добавляем драйвер в образ загрузки при запуске системы

sudo bash -c 'echo "fb_st7796s" >> /etc/initramfs-tools/modules'\
sudo update-initramfs -u

проверить есть ли драйвер в ядре\
lsinitramfs  /boot/uInitrd-[версия сборки образа см в папке]  |grep fb

корректно ли загружен (ошибки SPI если они есть будут там)\
lsmod | grep 7796

вывод от драйвера дисплея\
dmesg|grep 7796


4. сборка и добавление оверлея драйвера из папки DTS \

orangepi-add-overlay /boot/overlay-user/sun50i-h6-st7796s-dummy.dts

или 

dtc -O dtb -o /boot/overlay-user/sun50i-h6-st7796s-dummy.dtbo /boot/overlay-user/sun50i-h6-st7796s-dummy.dts\
и добавить строку user_overlays=sun50i-h6-st7796s-dummy в файл /boot/oragepiEnv.txt

после перезагрузки драйвер должен инициализировать дисплей
