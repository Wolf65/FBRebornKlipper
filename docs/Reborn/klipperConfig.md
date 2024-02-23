**== В РАЗРАБОТКЕ ==**

Настройки Klipper хранятся в текстовом файле `printer.cfg` и изменяются с помощью правок этого файла. Ничего компилировать не нужно. Внёс правки → сохранил → перезапустил → готово.

Есть обязательные параметры, которые необходимо явно указывать и определять и необязательные, которые имеют встроенное значение по умолчанию. Необязательные в свою очередь делятся на те, что всегда активны, даже если не указаны или закомментированы (с помощью #) в файле и на те, что отключены, если не указаны или закомментированы.

Например, вы не можете не указать кинематику, это вызовет ошибку запуска клиппера. Но если вы не укажете минимальную температуру экструзии, будет использовано значение 170℃ по умолчанию. Если вы не укажете Input Shaping в конфиге, он будет отключён, а команды, связанные с ним, будут вызывать ошибку. Иногда в других руководствах можно встретить какие-то параметры, которые и так соответствуют значениям по умолчанию. Я такие параметры просто не указываю в большинстве случаев.

`printer.cfg` поддерживает модульность и вложенность. Блоки кода могут быть вынесены в отдельные конфигурационные файлы, для подключения которых в `printer.cfg` нужно добавить строчку

```
[include имяфайла.cfg]
```

Значения могут дублироваться в основном файле `printer.cfg` и дочерних конфигах, и при этом иметь разные значения по следующим правилам:

- значение параметра из каждого следующего включённого конфига перезаписывает предыдущий, т.е. чем ниже в списке включённых файл, тем выше его приоритет
- файл `printer.cfg` имеет наивысший приоритет и значения параметров, указанных в этом файле переопределяют значения этих параметров в других включённых

Файл `printer.cfg` расположен в папке `/home/pi/klipper_config/`

---
## MCU - микроконтроллер

``` title="printer.cfg"
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
restart_method: command
```

===+ "USB"
    ``` title="printer.cfg"
    serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
    ```

    но лучше проверить свой порт выполнив команду

    ```shell title="bash"
    ls /dev/serial/by-id/*
    ```

=== "UART RPi Zero W и RPi 3B+"
    ``` title="printer.cfg"
    serial: /dev/ttyAMA0
    ```

=== "UART RPi 4"
    ``` title="printer.cfg"
    serial: /dev/ttyAMA1
    ```

---
## PRINTER - кинематика

``` title="printer.cfg"
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
## EXTRUDER - фидер и хотэнд

---
## HEATER BED - стол

``` title="printer.cfg" hl_lines="6 7 8"
[heater_bed]
heater_pin: PA0
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PC0
control: pid
pid_Kp: 0
pid_Ki: 0
pid_Kd: 0
min_temp: 0
max_temp: 130
```

---
## FANS - обдув

### Модели

``` title="printer.cfg"
[fan]
pin: PB1
```
### Хотэнда

``` title="printer.cfg"
[heater_fan heater_fan]
pin: PB0
```

---
## BEEPER - пищалка

``` title="printer.cfg"
[output_pin BEEPER_pin]
pin: PC5
pwm: True
value: 0
shutdown_value: 0
cycle_time: 0.001
scale: 1000
```

---
## FILAMENT SENSOR - датчик окончания филамента

``` title="printer.cfg"
[filament_switch_sensor filament_sensor]
switch_pin: PA4
runout_gcode:
    BEEP P=1500
```

Можно включать и отключать командой G-code `SET_FILAMENT_SENSOR SENSOR=filament_sensor ENABLE=[0|1]`. В данной конфигурации помимо паузы запускает макрос `BEEP` - подачу звукового сигнала.

---
## BED LEVELING - регулировка уровня стола

``` title="printer.cfg"
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