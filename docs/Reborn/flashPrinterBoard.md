## Настройка прошивки

Для того, чтобы Raspberry Pi мог управлять микроконтроллером принтера, требуется скомпилировать и установить прошивку на плату принтера. Настройки прошивки зависят от способа соединения Raspberry Pi с управляющей платой - по USB или по UART.

В терминале предварительно очищаем рабочий каталог, чтобы гарантированно полностью пересобрать прошивку:

```shell
cd ~/klipper/
make clean
```

Запускаем интерфейс настройки прошивки:

```shell
make menuconfig
```

### Конфигурации для платы принтера

=== "MKS Robin Nano v1.1"
    - Включаем `Extra low-level configuration options`
    - `Micro-controller Architecture` → `STMicroelectronics STM32`
    - `Processor model` → `STM32F103`
    - `Bootloader offset` → `28KiB bootloader`

    ===+ "USB"
        - `Communication interface` → `Serial (on USART3 PB11/PB10)` т.к. USB подключение этой платы использует пины `UART3`: `PB10-TX` и `PB11-RX`

        ---

    === "UART" 
        - `Communication interface` → `Serial (on USART1 PA10/PA9)` т.к. WiFi модуль платы использует пины `UART1`: `PA9-TX` и `PA10-RX`

        ---
    
    (Опционально)

    - Для отключения стокового экрана (т.к. klipper его не поддерживает) можно указать в `GPIO pins to set at micro-controller startup` → `!PD13,!PC6`

===+ "MKS Robin Nano v1.3/MKS Robin Nano-S v1.3"
    - Включаем `Extra low-level configuration options`
    - `Micro-controller Architecture` → `STMicroelectronics STM32`
    - `Processor model` → `STM32F407`
    - `Bootloader offset` → `32KiB bootloader`

    ===+ "USB"
        - `Communication interface` → `Serial (on USART3 PB11/PB10)` т.к. USB подключение этой платы использует пины `UART3`: `PB10-TX` и `PB11-RX`

        ---

    === "UART"
        - `Communication interface` → `Serial (on USART1 PA10/PA9)` т.к. WiFi модуль платы использует пины `UART1`: `PA9-TX` и `PA10-RX`

        ---

    (Опционально)

    - Для отключения стокового экрана (т.к. klipper его не поддерживает) можно указать в `GPIO pins to set at micro-controller startup` → `!PD13,!PC6`

---
## Компиляция прошивки

Сохраняем конфигурацию нажатием последовательно «Q» и «Y». После конфигурации запускаем компиляцию:

```shell
make
```

Результат - файл `klipper.bin` в папке `~/klipper/out/`

=== "MKS Robin Nano v1.1"
    - Бутлоадер этой платы требует шифрования и определённого имени файла. Выполните следующую команду для шифрования и переименования:
    ```shell
    ~/klipper/scripts/update_mks_robin.py ~/klipper/out/klipper.bin ~/klipper/out/Robin_nano35.bin
    ```

    ---

===+ "MKS Robin Nano v1.3/MKS Robin Nano-S v1.3"
    - Бутлоадер этой платы уже не требует шифрования, а только определённого имени файла. Выполните следующую команду для переименования:
    ```shell
    mv ~/klipper/out/klipper.bin ~/klipper/out/Robin_nano35.bin
    ```

    ---

Так же для удобства можно сразу переместить готовый `Robin_nano35.bin` в корень `printer_data/config/`

```shell
mv ~/klipper/out/Robin_nano35.bin ~/printer_data/config/Robin_nano35.bin
```

После файл можно `Robin_nano35.bin` скачать через веб-интерфейсе `Fluidd/Mainsail`

## Прошивка платы

Вставляем MicroSD в принтер и обесточиваем его. Если к плате принтера подключен Raspberry Pi, то отключите и его.

Так как стоковый экран принетра не даст вам определить, прошился ли он. На экране зависнет ход прогресса на 100%. Ждем пару минут. После перезагрузке будет либо чёрный экран (если при конфигурации прошивки он был отключен [в конфигурации платы принтера](#конфигурации-для-платы-принтера)), либо вечная надпись `Booting...`. Это нормально.