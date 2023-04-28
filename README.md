# mini-smart-switch2
mini smart switch - Прошивка родного модуля для работы без облака, через mqtt сервис. или прошивка в аналог [esphome - libretuya](https://docs.libretiny.eu/docs/flashing/esphome/). Конфиг в конце сообщения

![photo_5406796972041551310_y](https://user-images.githubusercontent.com/64173457/199171022-25f41710-afef-4c5f-81bd-28cb0680579a.jpg)

![photo_5406796972041551315_y](https://user-images.githubusercontent.com/64173457/199172046-9762c24b-a1b2-4c9b-8c7c-6cb610e02025.jpg)

В приложении от Tuya это выглядит так:

![photo_5406796972041551349_y](https://user-images.githubusercontent.com/64173457/199174821-9ad6d85f-7890-4259-a620-97959f40062e.jpg)
![photo_5406796972041551350_y](https://user-images.githubusercontent.com/64173457/199174817-6305af05-69a9-4ad3-bfe3-1020695ba97c.jpg)

Корпус собран на 4-х защелках, вскрывается медиатором или пластиковой карточкой или ногтями :-) начинаем от щели из угла с контактами.
Внутри вполне приличное реле на 16 ампер и неизвестный модуль а-ля Tuya

![photo_5406796972041551314_y](https://user-images.githubusercontent.com/64173457/199171339-020481b3-7612-4394-993b-19a15c2b45cd.jpg)

На модуле установлен чип от Belon BL2028N

![photo_5406796972041551313_y](https://user-images.githubusercontent.com/64173457/199172440-e0386923-1bbe-4fb0-b44b-0186ec564d2d.jpg)

Информации маловато, например https://market.cloud.tencent.com/products/28166

Прошивать его будем как чип BK7231N

Для считывания данных и прошивки достаточно подключить внешнее питание 3,3v и uart

Все пины доступны без выпаивания модуля.

![photo_5404545172227867270_y](https://user-images.githubusercontent.com/64173457/199173076-2e3727ef-7ea0-4d41-9ba8-16dbef886329.jpg)

Спасибо разрабочикам - можно ничего не искать, все подписано на плате:

![photo_5406796972041551312_y](https://user-images.githubusercontent.com/64173457/199173336-b5761ee7-c6f3-4570-acf7-52c3c23e78d2.jpg)

нам нужны 3.3v, GND, RX1, TX1

TP CS это, очевидно не чип селект, а CEN ( сброс ) см фото с фронта, но она нам и не нужна.

![photo_5406796972041551311_y](https://user-images.githubusercontent.com/64173457/199173842-9c0e2e0f-f809-4154-9241-5489f19be951.jpg)

Прошивать лучше по инструкции из https://github.com/openshwprojects/OpenBK7231T_App#flashing-for-bk7231n
Нужен установленный Питон и архивчик прошивальщика https://github.com/OpenBekenIOT/hid_download_py/archive/refs/heads/master.zip

Но для начала полезно сохранить текущую прошивку. ключики -r --unprotect  --startaddr 0x0 --length 200000 (--baudrate 115200 ,  если ваш порт не умеет 921600  все описаны тут: https://github.com/OpenBekenIOT/hid_download_py )

Метод чтения ( прошивки ) подключаем порт запускаем прошивальщик на чтение ( запись) и подаем питание на чип. Если все запустилось прошивальщик начнет бодро рисовать на экране разные цифры. Если не запустилось повторяем. Если получили на выходе файл размером 2mb - у вас есть бэкап маков, калибровки и прошивка от производителя - всегда можно откатится.

На страничке проэкта https://github.com/openshwprojects/OpenBK7231T_App
берем крайний релиз https://github.com/openshwprojects/OpenBK7231T_App/releases
Нам нужен BK7231N	UART Flash ( Например https://github.com/openshwprojects/OpenBK7231T_App/releases/download/1.14.104/OpenBK7231N_QIO_1.14.104.bin )
Прошиваем:
```
python uartprogram c:\temp\OpenBK7231N_QIO_1.14.104.bin --unprotect -d com10 -w --startaddr 0x0
```
По окончании прошивки чип перезагрузится и поднимется открытая точка доступа. Подключаемся, открываем в обозревателе адрес 192.168.4.1
Настраиваем подключение к wifi, mqtt, и задаем настройки пинов чипа:

![photo_5404545172227867281_y](https://user-images.githubusercontent.com/64173457/199177429-9139ed3c-0f29-484b-86cd-dd6f5a1423ca.jpg)

Отправляем discovery

![photo_5404545172227867296_x](https://user-images.githubusercontent.com/64173457/199177697-a3c29a88-fecf-4e6b-b17c-49d9d425cca6.jpg)


и получаем в HA:

![photo_5404545172227867299_y](https://user-images.githubusercontent.com/64173457/199177796-6453d2f0-f502-4126-bfef-fbc1a511b19c.jpg)

yaml esphome [libretuya](https://docs.libretiny.eu/docs/flashing/esphome/)
```
substitutions:
  board_name: "mini2"

esphome:
  name: $board_name
  project:
    name: "MiniSmartSwitch/Aubess.LibreTuya"
    version: "BL2028N"
  comment: "коридор"

libretuya:
  board: generic-bk7231n-qfn32-tuya
  framework:
    version: dev

api:
  encryption:
    key: !secret keyapi 
  #reboot_timeout: 0s

ota:
  password: !secret passwordota

logger:
  baud_rate: 0

captive_portal:
wifi:
  ssid: !secret wifi1
  password: !secret password1
  ap:
    ssid: "$board_name Hotspot"
    password: !secret password1

web_server:
  port: 80  

binary_sensor:
  - platform: gpio
    pin:
      number: P26 # контакты выключателя
      mode: INPUT_PULLUP
      inverted: true
    name: switch_$board_name
    on_state:
    - light.toggle: light_1
    
  - platform: gpio
    pin:
      number: P6 # кнопка
      mode: INPUT_PULLUP
      inverted: true
    name: key_$board_name
    on_press:
    - light.toggle: light_1

output:
  - platform: gpio
    pin: P24
    id: output1

light:
  - platform: status_led
    name: Status_$board_name
    id: light_s
    internal: true
    pin:
      number: P7 # индикатор
#      inverted: true

  - platform: binary
    name: $board_name
    id: light_1
    output: output1
    restore_mode: RESTORE_DEFAULT_OFF
    on_turn_on:
      - light.turn_off: light_s
    on_turn_off:
      - light.turn_on: light_s


sensor:
  - platform: wifi_signal
    name: WiFi_Signal.$board_name
  - platform: uptime
    name: uptime_sensor.$board_name
button:
  - platform: restart
    name: Reset.$board_name
text_sensor:
  - platform: version
    name: Version.$board_name
```





