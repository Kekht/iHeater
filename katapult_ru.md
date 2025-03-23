# Katapult Bootloader for STM32F042F6P6

Этот документ содержит инструкции по сборке и прошивке загрузчика **Katapult** от Klipper для микроконтроллера **STM32F042F6P6**. Загрузчик Katapult позволяет прошивать прошивку Klipper по USB без ST-Link, через DFU.

---

## 🧰 Что потребуется

- STM32F042F6P6
- Плата iHeater
- ST-Link V2 программатор (для первой прошивки) или USB кабель
- Linux-система (например, Raspberry Pi или принтер)

---

## ⚙️ Сборка Katapult

1. Клонируем репозиторий Katapult:

```bash
git clone https://github.com/Klipper3d/katapult
cd katapult
make menuconfig
```

2. В `menuconfig` выбери:

- MCU Architecture: STM32
- Processor model: STM32F042
- Clock Reference: Internal
- Communication interface: USB (on PA9/PA10)
- Application start offset: **8KiB offset**
- [x] Support bootloader entry on rapid double click of reset button
- [x] Enable bootloader entry on button (or gpio) state
- (!PA4)  Button GPIO Pin
- [X] Enable Status LED
- (PA5)   Status LED GPIO Pin


![menuconfig](img/katapult_menuconfig.jpg)

3. Сборка:

```bash
make
```

Прошивка будет создана в `out/katapult.bin`.

---

## 🔌 Прошивка Katapult через ST-Link

> Этот шаг нужен только один раз, для загрузки самого Katapult.

### Установка утилиты `st-flash`:

```bash
sudo apt install stlink-tools
```

### Подключение ST-Link:

| ST-Link | STM32     |
|---------|-----------|
| SWDIO   | PA13      |
| SWCLK   | PA14      |
| GND     | GND       |
| 3.3V    | VDD       |

### Прошивка:

```bash
sudo st-flash write out/katapult.bin 0x08000000
```

Если всё в порядке, появится сообщение:

```
Flash written and verified! jolly good!
```

---

## Прошивка Katapult через DFU

> Этот шаг нужен только один раз, для загрузки самого Katapult.

### Подготовка:
Установи утилиту dfu-util, если она ещё не установлена:
    
    sudo apt install dfu-util

Установи джампер на BOOT0 и перезапусти питание платы (или нажми кнопку RESET).
Микроконтроллер загрузится в режим DFU.

Проверь подключение:

    lsusb

Ты должен увидеть:

    ID 0483:df11 STMicroelectronics STM Device in DFU Mode

### Прошивка Katapult:
Выполни команду:

    dfu-util -a 0 -D out/katapult.bin -s 0x08000000:leave

Пример успешной прошивки:

    Downloading to address = 0x08000000, size = 4968
    Download        [=========================] 100%         4968 bytes
    Download done.
    File downloaded successfully
    Transitioning to dfuMANIFEST state

После перезапуска плата появится как:

    /dev/serial/by-id/usb-katapult_stm32f042x6_XXXXXXXXXXXXXX-if00


## 📋 Примечания

- Katapult занимает первые 8 КБ Flash, поэтому **в Klipper обязательно указывать смещение 8 KiB**.
- Можно использовать либо двойной Reset, либо кнопку на GPIO (PA4) для входа в DFU.
- Если PA13/PA14 используются для SWD
- После прошивки Katapult можно больше не использовать ST-Link — вся дальнейшая работа по USB.

