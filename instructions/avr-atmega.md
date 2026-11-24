# AVR ATmega328P на Arduino Uno R3

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для контролера датчика або панелі визнач розмір буферів, допустиму затримку реакції на вхід і дані, що зберігаються в EEPROM. Зафіксуй очікувану частоту плати й потрібний спосіб перепрошивання після монтажу. [ATmega328P Datasheet](https://www.microchip.com/en-us/product/ATmega328P)

## Цільова платформа

Uno R3 із заводським завантажувачем дає змогу записувати образ через USB-послідовний порт. Для власної плати вибери спосіб першого програмування та перевір точну модель AVR, частоту й доступність ISP. Microchip позначає ATmega328P як NRND, тому для нового серійного виробу перевір життєвий цикл вибраного чипа. [Arduino Uno R3](https://docs.arduino.cc/hardware/uno-rev3/), [Статус ATmega328P](https://www.microchip.com/en-us/product/ATmega328P), [Апаратна довідка](../wiki/1-hardware-platforms.md#16-avr--atmega)

## Вибір стека прошивки

**Arduino AVR Core** зручний для Uno R3 і пристроїв, де вистачає наявних бібліотек та USB-завантажувача. **Bare metal** доречний, коли команда хоче явно керувати таймерами, векторами й використанням SRAM. [Arduino AVR Core](https://github.com/arduino/ArduinoCore-avr), [AVR-GCC](https://gcc.gnu.org/onlinedocs/gcc/AVR-Options.html)

## Межі для розробника

Тримай конфігурацію таймерів і виводів у коді плати. Розробник додає функцію через наявний інтерфейс вводу або часу. Зміни частоти, параметрів завантажувача або fuse-бітів звіряй у рев’ю. Розмір секцій видно у звіті збірки, а запас стека перевіряється за робочого навантаження.

## Відтворюване середовище розробки

### Сетап через Arduino AVR Core

1. Для Ubuntu 24.04 x86-64 встанови Arduino CLI `1.3.1` і Arduino AVR Core `1.8.6`. Core встановлює AVR-GCC `7.3.0-atmel3.6.1-arduino7` з індексу Arduino. `$PORT` — порт підключеної Uno R3, `Blink` — каталог із `Blink.ino`. Для проєкту зафіксуй core і бібліотеки в `sketch.yaml` та збирай локально й у CI з цим профілем. [Arduino CLI 1.3.1](https://github.com/arduino/arduino-cli/releases/tag/v1.3.1), [Індекс Arduino](https://downloads.arduino.cc/packages/package_index.json), [Профіль `sketch.yaml`](https://docs.arduino.cc/arduino-cli/sketch-project-file/), [Blink](https://docs.arduino.cc/built-in-examples/basics/Blink/)
2. Збери приклад і запиши вже зібраний HEX з перевіркою. LED на платі має блимати після перезапуску. Для діагностики в `arduino-cli monitor` задай baud rate, який встановлено в застосунку. [Команда `compile`](https://docs.arduino.cc/arduino-cli/commands-reference/arduino-cli_compile), [Команда `upload`](https://docs.arduino.cc/arduino-cli/commands-reference/arduino-cli_upload)

   ```sh
   arduino-cli core update-index &&
   arduino-cli core install arduino:avr@1.8.6 &&
   arduino-cli compile --fqbn arduino:avr:uno --output-dir build/uno Blink &&
   arduino-cli upload --fqbn arduino:avr:uno --port "${PORT:?Вкажи порт Uno R3}" --input-dir build/uno --verify Blink
   ```

### Сетап без фреймворку

1. Встанови AVR-GCC, avr-libc, AVR Binutils та AVRDUDE. AVR-GCC вибирає стартовий код для MCU за `-mmcu=atmega328p`. Для Uno R3 задай частоту 16 МГц. У `main.c` підключи `<avr/io.h>`, задай PB5 як вихід через `DDRB` і встанови потрібний рівень через `PORTB`. Для першого тесту після цього достатньо нескінченного циклу і перевірки LED `L`. [AVR Core і параметри Uno](https://github.com/arduino/ArduinoCore-avr/blob/master/boards.txt), [Схема Uno R3](https://docs.arduino.cc/hardware/uno-rev3/), [AVR-GCC Options](https://gcc.gnu.org/onlinedocs/gcc/AVR-Options.html)
2. Збери ELF і Intel HEX, потім запиши HEX через наявний завантажувач Uno R3 на виділеній платі. Команда записує Flash через протокол Arduino. Для окремого чипа з ISP використовуй конфігурацію свого програматора. [Приклад AVRDUDE для Arduino](https://github.com/avrdudes/avrdude)

   ```sh
   mkdir -p build &&
   avr-gcc -mmcu=atmega328p -DF_CPU=16000000UL -Os -g -Wl,-Map=build/firmware.map -o build/firmware.elf main.c &&
   avr-objcopy -O ihex -R .eeprom build/firmware.elf build/firmware.hex &&
   avrdude -c arduino -P "${PORT:?Вкажи порт Uno R3}" -b 115200 -p m328p -D -U flash:w:build/firmware.hex:i
   ```

3. Для покрокового налагодження застосуй MPLAB X і debugWIRE на платі з відповідною схемою RESET та сумісним адаптером. Якщо IDE керує перемиканням debugWIRE, заверши сеанс через її штатну команду вимкнення debugWIRE, щоб відновити ISP. Через USB-завантажувач Uno використовуй UART або GPIO для спостереження роботи. [Microchip debugWIRE](https://developerhelp.microchip.com/xwiki/bin/view/software-tools/ides/x/debugging/avr-debugwire/)

## Перевірка й фіксація рішення

Фіксуй бюджет прикладної Flash з урахуванням завантажувача, статичну RAM і запас стека. На стенді перевір старт за потрібної частоти та сумісність EEPROM між випусками. Якщо серійний запис використовує ISP, опиши fuse-біти, політику стирання EEPROM і спосіб повернути штатний завантажувач. [ATmega328P Datasheet](https://www.microchip.com/en-us/product/ATmega328P)

Зафіксуй обраний стек, цільову частоту, спосіб запису і запас ресурсів, за якого цей варіант підтримує сценарій.
