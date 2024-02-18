# Reborn

## Прошивка плата принтера
### Настройка прошивки
Для того, чтобы Raspberry Pi мог управлять микроконтроллером принтера, требуется скомпилировать и установить прошивку МК. Настройки прошивки зависят от способа соединения Raspberry Pi с управляющей платой - по USB или по UART.

В терминале предварительно очищаем рабочий каталог, чтобы гарантированно полностью пересобрать прошивку:
```batch
cd ~/klipper/
make clean
```
Запускаем интерфейс настройки прошивки:
```batch
make menuconfig
```
#### Конфигурации для плат принтера
=== "MKS Robin Nano v1.1"
    - Включаем `Extra low-level configuration options`
    - Micro-controller Architecture → `STMicroelectronics STM32`
    - Processor model → `STM32F103`
    - Bootloader offset → `28KiB bootloader`
    ===+ "USB"
        - Communication interface → `Serial (on USART3 PB11/PB10)`
        потому, что USB подключение этой платы использует пины UART3: PB10-TX и PB11-RX
    === "UART" 
        - Communication interface → `Serial (on USART1 PA10/PA9)`
        потому, что WiFi модуль платы использует пины UART1: PA9-TX и PA10-RX

    ---

===+ "MKS Robin Nano v1.3/MKS Robin Nano-S v1.3"
    - Включаем `Extra low-level configuration options`
    - `Micro-controller Architecture` → `STMicroelectronics STM32`
    - `Processor model` → `STM32F407`
    - `Bootloader offset` → `32KiB bootloader`
    ===+ "USB"
        - `Communication interface` → `Serial (on USART3 PB11/PB10)`

        т.к. USB подключение этой платы использует пины UART3: PB10-TX и PB11-RX

        ---

    === "UART"
        - `Communication interface` → `Serial (on USART1 PA10/PA9)` 
        
        т.к. WiFi модуль платы использует пины UART1: PA9-TX и PA10-RX

        ---

    - Если нужно отключить стоковый экран (т.к. klipper его не поддерживает) можно в `GPIO pins to set at micro-controller startup` → `!PD13 !PC6`

    ---

### Компиляция прошивки

Сохраняем конфигурацию нажатием последовательно «Q» и «Y». После конфигурации запускаем компиляцию:
```batch
make
```
Результат - файл `klipper.bin` в папке `~/klipper/out/`
=== "MKS Robin Nano v1.1"
    - Бутлоадер этой платы требует шифрования и определённого имени файла. Выполните следующую команду для шифрования и переименования:
    ```batch
    ~/klipper/scripts/update_mks_robin.py ~/klipper/out/klipper.bin ~/klipper/out/Robin_nano35.bin
    ```
    ---
===+ "MKS Robin Nano v1.3/MKS Robin Nano-S v1.3"
    - Бутлоадер этой платы уже не требует шифрования, а только определённого имени файла. Выполните следующую команду для переименования:
    ```batch
    mv ~/klipper/out/klipper.bin ~/klipper/out/Robin_nano35.bin
    ```
    ---
Так же для удобства можно сразу переместить готовый `Robin_nano35.bin` в корень `printer_data/config/`
```batch
mv ~/klipper/out/Robin_nano35.bin ~/printer_data/config/Robin_nano35.bin
```
После файл можно `Robin_nano35.bin` скачать через веб-интерфейсе Fluidd/Mainsail

### Прошивка платы

Вставляем MicroSD в принтер и обесточиваем его. Если к плате принтера подключен Raspberry Pi, то отключите и его.

Через пару минут МК принтера готов к дальнейшей работе.

Стоковый экран принетра не даст вам определить, прошился ли он. На экране зависнет ход прогресса на 100%. А при перезагрузке будет либо чёрный экран, либо вечная надпись `Booting...`. Это нормально.

## Конфигурация klipper

Настройки Klipper хранятся в текстовом файле `printer.cfg` и изменяются с помощью правок этого файла. Ничего компилировать не нужно. Внёс правки → сохранил → перезапустил → готово.

Есть обязательные параметры, которые необходимо явно указывать и определять и необязательные, которые имеют встроенное значение по умолчанию. Необязательные в свою очередь делятся на те, что всегда активны, даже если не указаны или закомментированы (с помощью #) в файле и на те, что отключены, если не указаны или закомментированы.

Например, вы не можете не указать кинематику, это вызовет ошибку запуска клиппера. Но если вы не укажете минимальную температуру экструзии, будет использовано значение 170℃ по умолчанию. Если вы не укажете Input Shaping в конфиге, он будет отключён, а команды, связанные с ним, будут вызывать ошибку. Иногда в других руководствах можно встретить какие-то параметры, которые и так соответствуют значениям по умолчанию. Я такие параметры просто не указываю в большинстве случаев.

`printer.cfg` поддерживает модульность и вложенность. Блоки кода могут быть вынесены в отдельные конфигурационные файлы, для подключения которых в `printer.cfg` нужно добавить строчку
```batch
[include имяфайла.cfg]
```
Значения могут дублироваться в основном файле `printer.cfg` и дочерних конфигах, и при этом иметь разные значения по следующим правилам:

- значение параметра из каждого следующего включённого конфига перезаписывает предыдущий, т.е. чем ниже в списке включённых файл, тем выше его приоритет
- файл `printer.cfg` имеет наивысший приоритет и значения параметров, указанных в этом файле переопределяют значения этих параметров в других включённых

Файл `printer.cfg` расположен в папке `/home/pi/klipper_config/`

---
### MCU - микроконтроллер

```
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
restart_method: command
```
===+ "USB"
    ```
    serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
    ```
    но лучше проверить свой порт выполнив команду
    ```batch
    ls /dev/serial/by-id/*
    ```

=== "UART RPi Zero W и RPi 3B+"
    ```
    serial: /dev/ttyAMA0
    ```

=== "UART RPi 4"
    ```
    serial: /dev/ttyAMA1
    ```

---
### PRINTER - кинематика

```batch
[printer]
kinematics: corexy
max_velocity: 250
max_accel: 3000
max_accel_to_decel: 2000
max_z_velocity: 20
max_z_accel: 100
```

Значения `max_velocity` и `max_accel` определяются в ходе калибровок, а `square_corner_velocity` лучше оставить по умолчанию (не указывать) для корректной работы Input Shaping.

---
### EXTRUDER - фидер и хотэнд

---
### HEATER BED - стол

```batch
[heater_bed]
heater_pin: PA0
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PC0
control: pid
pid_Kp: 325.10
pid_Ki: 63.35
pid_Kd: 417.10
min_temp: 0
max_temp: 130
```

---
### FANS - обдув

#### Модели

```batch
[fan]
pin: PB1
```
#### Хотэнда

```batch
[heater_fan heater_fan]
pin: PB0
```

---
### BEEPER - пищалка

```batch
[output_pin BEEPER_pin]
pin: PC5
pwm: True
value: 0
shutdown_value: 0
cycle_time: 0.001
scale: 1000
```

---
### FILAMENT SENSOR - датчик окончания филамента

```batch
[filament_switch_sensor filament_sensor]
switch_pin: PA4
runout_gcode:
    BEEP P=1500
```

Можно включать и отключать командой G-code `SET_FILAMENT_SENSOR SENSOR=filament_sensor ENABLE=[0|1]`. В данной конфигурации помимо паузы запускает макрос `BEEP` - подачу звукового сигнала.

---
### BED LEVELING - регулировка уровня стола

```batch
[bed_screws]
screw1: 25,30
screw1_name: front left screw
screw2: 230,30
screw2_name: front right screw
screw3: 230,180
screw3_name: back right screw
screw4: 25,180
screw4_name: back left screw
speed: 150
```