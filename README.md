Control the car engine start using the sms request, and send the data to narodmon.ru 

# Проект для удаленного автозапуска двигателя автомобиля на дешевом GSM модеме M590 и Arduino Pro Mini

![](https://github.com/martinhol221/M590_autostart_car_engine/blob/master/other/188cce5s-960%20(2).jpg)


## Отправка данных на сервис народного мониторинга каждых 5 минут

TCP пакет уходящий на сервер `94.142.140.101` в порт `8283` каждых 5 минут:

`#59-01-AA-00-00-00#GSM-Sensor`  - адресс и имя устройства      

`#Temp0#17.30`                   - температура с датчика DS18B20 (крепление на двигатель)

`#Temp1#19.85`                   - температура с датчика DS18B20 (нахождение в салоне)

`#Temp3#9.11`                    - с других датчиков , до 10 шт. на одной шине.

`#Vbat#7.60`                     - напряжение АКБ

`#Uptime#102`                    - время непрерывной работы устройства в секундах

`##`                             

## Получение данных на сервис народного мониторинга каждых 5 минут

`+TCPSETUP:0,OK`                 - обычный ответ без команды

`+TCPRECV:0,8,#estart`           - с командой на запуск двигятеля

`+TCPRECV:0,7,#estop`            - с командой на останов прогрева

Получение данных и отправка команд возможна как с веб сайта [narodmon.ru](https://narodmon.ru), так и из приложения [Народмон 2017](https://play.google.com/store/apps/details?id=com.axbxcx.narodmon&hl=ru). Необходима регистрация на сайте и добавление своего датчика. "Датчики" далее  "добавить мое устройство мониторинга" вставить адрес (пример 59-01-AA-00-00-00), "ок". После чего в приложении будут отображаться ваши датчики. В приложении "Управление", "+", "Произвольная команда", "Команда", напечатать `estart` или `estop`, можно вывести виджетом на главный экран телефона. 

## Запуск по входящему звонку

Тут все просто, звонок с номера `call_phone = "375290000000";` - вызывает алгаритм запуска, повторный звонок на этот номер в течении прогрева останавливает прогрев.

## СМС отчет в случае запуска не с первой попытки

Исходящая СМС будет отправлена за 2 минуты до остановки прогрева по таймеру или в случае если предпринято более 1 попытки запуска.

`Privet GSM-Sensor!`           - приветствие с именем устройства

`Voltage BAT Now: 14.24`       - напряжение АКБ сию минуту

`Voltage BAT Min: 8.29`        - нимимальное напряжение зафиксированное в момент последнего старта 

`Temp0: 17.30`                  - температура с датчика DS18B20 (крепление на двигатель)

`Temp1: 19.85`                  - температура с датчика DS18B20 (нахождение в салоне)

`Temp3: 9.11`                   - с других датчиков , до 10 шт. на одной шине.

`Attempts: 2`                   - количество попыток запуска при последнем старте

`Uprime: 10H `                  - время непрерывной работы устройства без перезагрузок в часах


#  Алгаритм запуска

 ## Предпусковая настройка
 
 В зависимости от температуры двигателя на датчике `Temp0` автоматически подбирается:
 
 Время работы стартера `StTime` от 1 до 6 сек 
 
 Таймер обратного отсчета `Timer` от 5 до 30 минут

 Число повторов прогрева свечей накала (для дизелистов) о 0 до 5
 
 в соответствии с [таблицей](https://raw.githubusercontent.com/martinhol221/M590_autostart_car_engine/master/other/calibr.log)
 
 ## Попытки запуска № 1,2,3 

 Наступят в случае если напряжение АКБ выше 10 вольт, на ножке `A3` ардуино низкий логический уровень, температура `Temp0` выше минус 30 градусов, и кличество оставшихся попыток меньше `Attempts`
 
1.Бует включены реле первого положения ключа (и реле обходчика иммобилайзера)

2.Включено зажигание на 4 сек

3.Многократно отключено и сново включено зажигание для прогрева свечей накала (можно отклбчить функцию для бензиновых двигателей)

4.Будет произведена проверка выключенной передачи по пину `A2` (низкий уровень) 

5.Включится реле стартера на время `StTime` в границах которого будет замерятся минимальное напряжение при работе стартера

6.Выключится реле стартера и программа приостановится на 6 сек ожидая пявление зарядки от генератора

7. В случае если напряжение АКБ окажется больше `Vstart` считаем старт успешным, в противном случае возвращаемся к пункту 1 
 
 ## Остановка по таймеру, при низком напряжении

![](https://github.com/martinhol221/M590_autostart_car_engine/blob/master/other/IMG_3714.JPG)
![](https://github.com/martinhol221/M590_autostart_car_engine/blob/master/other/IMG_3712-001.JPG)
![](https://github.com/martinhol221/M590_autostart_car_engine/blob/master/other/IMG_3711-002.JPG)
![Схема, возможны не точности](https://github.com/martinhol221/M590_autostart_car_engine/blob/master/other/Shema.jpg)

[Лог работы](https://raw.githubusercontent.com/martinhol221/M590_autostart_car_engine/master/other/send_data_narodmon.log)



## Важно понимать, что модемы М590 приходят из Китая уже выпаянные их других устройст с большим процентам брака, от этого могут не регистрироваться в сети и быть в дальнейшем не пригодными для эксплуатации.

[Мой проект на модеме SIM800L](https://github.com/martinhol221/SIM800L_DTMF_control)
