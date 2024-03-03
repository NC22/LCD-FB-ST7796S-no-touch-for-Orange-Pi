## LCD ST7796S 

LCD экран 320x480 с контроллером fb st7796s, без тача

В репозитории мои наработки под печать крепления к стандартному корпусу, исправленный dts-оверлей под ubuntu и внешний модуль ядра, провереные на Orange PI 3 LTS (Allwinner H6 Cortex-A53), Ubuntu 5.16

Для других устройств\операционок dts оверлей будет отличатся.\
В дальнейшем если будут какие-либо еще оверлеи под конкретные операционки, будут оставлены соответствующие пометки. Пока кроме как на ubuntu на других система проверить не было возможности.

Доработки драйвера FBTFT для поддержки контроллера st7796s взяты <a href="https://github.com/Sergey1560/fb_st7796s">здесь</a> так же там более простая инструкция на случай если у вас armbian

## Распиновка

<img src="https://github.com/NC22/LCD-FB-ST7796S-no-touch-for-Orange-Pi/blob/main/opi3lts_pinout.jpg?raw=true">

## Установка и настройка на ubuntu

1. Установить необходимые пакеты

Для сборки FBTFT модуля требуются linux-headers которые через apt на ubuntu на орандже не установить - официально предлагается использовать софт для сборки ядра под конкретную модель устройства \ процессора (потребуется машина с Ubuntu 20) инструкция - <a href="http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_Zero_3#Linux_SDK.E2.80.94.E2.80.94orangepi-build_instruction">здесь</a>. Так же есть мой альтернативный нерациональный опыт компиляции ядра без использования внешней мощной машины для сборки - он описан в <i>long.txt</i>, но лучше разобратся с официальным если есть возможность.

```
sudo apt update 
sudo apt install build-essential
```

2. компилируем драйвер 

```
cd [путь до папки с FBTFT]

make\
sudo make install\
make clean\
sudo depmod -A
```

3. добавляем драйвер в образ загрузки при запуске системы

```
sudo bash -c 'echo "fb_st7796s" >> /etc/initramfs-tools/modules'\
sudo update-initramfs -u
```

проверить есть ли драйвер в ядре
```
lsinitramfs  /boot/uInitrd-[версия сборки образа см в папке]  |grep fb
```

корректно ли загружен
```
lsmod | grep 7796
```

вывод от драйвера дисплея (если драйвер загрузился при запуске системы, то хоть какие-то данные будет выводить - например ошибки инициализации SPI) 
```
dmesg|grep 7796
```

4. сборка и добавление оверлея драйвера из папки DTS
   
```
orangepi-add-overlay /boot/overlay-user/sun50i-h6-st7796s-dummy.dts
```

или 

```
dtc -O dtb -o /boot/overlay-user/sun50i-h6-st7796s-dummy.dtbo /boot/overlay-user/sun50i-h6-st7796s-dummy.dts
```
и добавить строку user_overlays=sun50i-h6-st7796s-dummy в файл /boot/oragepiEnv.txt

после перезагрузки драйвер должен инициализировать дисплей

5. Настройка графического интерфейса Xorg

Установить если необходимо среду рабочего стола \ драйвер под Allwiner

```
   sudo apt-get install xorg
   sudo apt-get install xfce4
   sudo apt-get install xserver-xorg-video-fbdev
```

Файл конфига 50-fbdev.conf скопировать в /etc/X11/xorg.conf.d

Запуск графического интерфейса командой: startx или startxfсe4
