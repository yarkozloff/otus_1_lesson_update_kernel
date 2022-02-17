# Установка ПО
Среда выполнения:
Рабочая машина с VMware - windows 10 версия 1809 (Сборка ОС 17763.1577)
Виртуальная машина -  Ubuntu 20.04.3 LTS
## Установка Virtualbox
Версия VirtualBox - 6.1.26_Ubuntur145957
```
sudo apt-get update
sudo apt-get install virtualbox
```
## Установка Vagrant
Версия Vagrant - 2.2.6
```
sudo apt install vagrant
```
## Установка Packer
```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install packer
```
# Kernel update
Создал в корне каталог /otus, чтобы не работать в директории /home
Выполнил fork данного репозитория: https://github.com/dmitry-lyutenko/manual_kernel_update
Далее клонировал на свою машину
```
git clone git@github.com:yarkozloff/manual_kernel_update.git
```
Изменил количество ядер, оперативной памяти, использовал также centos/7
запустил машину:
```
sudo vagrant up
```
Подключился по ssh. Проверил версию ядра
```
uname -r
```
3.10.0-1127.el7.x86_64
Приступил к обновлению с последующим grub update и перезагрузкой
```
sudo yum install -y http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
sudo yum --enablerepo elrepo-kernel install kernel-ml -y
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo grub2-set-default 0
sudo reboot
uname -r
```
Новая версия ядра:
5.16.9-1.el7.elrepo.x86_64

## Packer
Теперь необходимо создать свой образ системы, с уже установленым ядром 5й версии. Для это воспользуемся ранее установленной утилитой packer
### Изменения в файле centos.js:
"disk_size": "6000",  
**Экономлю место на диске, этого достаточно**

 "iso_url": "http://mirrors.powernet.com.ru/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso", 
 **Нашел iso образ версии 7.6.1810**
 
"shutdown_timeout" : "10m", 
**С меньшим таймаутом возникала ошибка Timeout waiting for SSH, очень долго разбирался**

"headless": "true",
**Отключает запуск ui vb, иначе ошибка NS_ERROR_FAILURE (0x80004005)**

Скрпиты изучил, применимы и для этой версии. Также были первый попытки собрать образ для centos 8, разбирался не в рамках выполнения дз
### Создание образа Packer
В каталоге packer выполнил
```
packer build centos.json
```
Builds finished. Создался box в текущем каталоге

## Vagrant cloud
Зарегистрировался в Vagrant cloud, создал box. Физически выгрузил box из виртуальной машины и воспользовался возможностью загрузить его через web версию. Посчитал и вписал сумму бокса, создал релизную версию yarkozloff/centos-7-6

## Проверка
Так как уже был запущен образ в vagrant, выполнил:
~~~
vagrant halt
~~~
Изменил в Vagrantfile box_name на yarkozloff/centos-7-6. Выполнил vagrant up для проверки. Поднялась машина с нужной версией ядра 5.16.9-1.el7.elrepo.x86_64
