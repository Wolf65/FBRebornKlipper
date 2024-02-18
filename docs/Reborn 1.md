# Reborn
## Плата принтера
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
    - Для подключения по "USB"
        * Communication interface → `Serial (on USART3 PB11/PB10)` потому, что USB подключение этой платы использует пины UART3: PB10-TX и PB11-RX
    - Для подключения по "UART" 
        * Communication interface → `Serial (on USART1 PA10/PA9)` потому, что WiFi модуль платы использует пины UART1: PA9-TX и PA10-RX
    - 
===+ "MKS Robin Nano v1.3 и MKS Robin Nano-S v1.3"
    - Включаем `Extra low-level configuration options`
    - `Micro-controller Architecture` → `STMicroelectronics STM32`
    - `Processor model` → `STM32F407`
    - `Bootloader offset` → `32KiB bootloader`
    - Для подключения по "USB"
        * `Communication interface` → `Serial (on USART3 PB11/PB10)` потому, что USB подключение этой платы использует пины UART3: PB10-TX и PB11-RX
    - Для подключения по "UART" 
        * `Communication interface` → `Serial (on USART1 PA10/PA9)` потому, что WiFi модуль платы использует пины UART1: PA9-TX и PA10-RX
    - Если нужно отключить стоковый экран (т.к. klipper его не поддерживает) можно в `GPIO pins to set at micro-controller startup` → `!PD13 !PC6`
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
===+ "MKS Robin Nano v1.3 и MKS Robin Nano-S v1.3"
    - Бутлоадер этой платы уже не требует шифрования, а только определённого имени файла. Выполните следующую команду для переименования:
    ```batch
    mv ~/klipper/out/klipper.bin ~/klipper/out/Robin_nano35.bin
    ```
Так же для удобства можно сразу переместить готовый `Robin_nano35.bin` в корень `printer_data/config/`
```batch
mv ~/klipper/out/Robin_nano35.bin ~/printer_data/config/Robin_nano35.bin
```
После файл можно `Robin_nano35.bin` скачать через веб-интерфейсе Fluidd/Mainsail
### Прошивка платы
Вставляем MicroSD в принтер и обесточиваем его. Если к плате принтера подключен Raspberry Pi, то отключите и его.

Через пару минут МК принтера готов к дальнейшей работе.

Стоковый экран принетра не даст вам определить, прошился ли он. На экране зависнет ход прогресса на 100%. А при перезагрузке будет либо чёрный экран, либо вечная надпись `Booting...`. Это нормально.