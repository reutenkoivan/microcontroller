# Raspberry Pi Pico та Pico 2

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для протоколу через PIO задай допустимі тривалості імпульсів, обсяг буфера та умови навантаження DMA. Визнач, які зовнішні компоненти мають бути підключені для перевірки та як плата має запускатися після вимкнення живлення. [Приклади PIO](https://github.com/raspberrypi/pico-examples/tree/master/pio)

## Цільова платформа

Почни з Pico на RP2040 або Pico 2 на RP2350 без суфікса W. Для Pico 2 зафіксуй архітектуру, під яку будуєш образ. За потреби додай окрему ціль для Hazard3 RISC-V. Для покрокового налагодження під’єднай Raspberry Pi Debug Probe через SWD. [Плати Pico](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html), [Апаратна довідка](../wiki/1-hardware-platforms.md#14-raspberry-pi-rp2040-та-rp2350)

## Вибір стека прошивки

**Pico C/C++ SDK** спрощує складання PIO-програм, ініціалізацію периферії та формування UF2. **Bare metal** доречний для обмеженого набору периферії, якщо команда бере на себе startup, мапінг пам’яті та формат образу. [Pico C/C++ SDK](https://www.raspberrypi.com/documentation/microcontrollers/c_sdk.html), [RP2350 Datasheet](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf)

## Межі для розробника

Власник цілі плати задає частоту, конфігурацію Flash і призначення машин станів PIO та каналів DMA. Розробник змінює логіку протоколу через наявний інтерфейс. Зміни цих ресурсів чи архітектури видно у параметрах збірки, а зміни часових гарантій виявляє осцилограма зі стенда.

## Відтворюване середовище розробки

### Сетап через Pico C/C++ SDK

1. Для Ubuntu 24.04 x86-64 і Arm-цілей бери `pico-sdk` `2.1.1` з підмодулями, `pico-examples` `sdk-2.1.1`, Arm GNU Toolchain `13.3.Rel1` (`arm-none-eabi`), CMake 3.28 і `picotool` `2.1.1`. Передай в `PICO_SDK_PATH` абсолютний шлях до SDK `2.1.1`. [SDK 2.1.1](https://github.com/raspberrypi/pico-sdk/releases/tag/2.1.1), [Приклади `sdk-2.1.1`](https://github.com/raspberrypi/pico-examples/tree/sdk-2.1.1), [Arm GNU Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), [`picotool` 2.1.1](https://github.com/raspberrypi/picotool/releases/tag/2.1.1)
2. З кореня `pico-examples` збудуй `blink`. Для Pico 2 Arm зміни `PICO_BOARD` на `pico2` і використай окремий каталог збірки. Для Pico 2 Hazard3 додай `-DPICO_PLATFORM=rp2350-riscv` і налаштуй сумісний RISC-V toolchain за документацією релізу. [Вибір платформи в SDK](https://github.com/raspberrypi/pico-sdk/blob/master/cmake/pico_pre_load_platform.cmake)

   ```sh
   cmake -S . -B build/pico -DPICO_BOARD=pico -DCMAKE_BUILD_TYPE=Debug \
     -DPICO_SDK_PATH="${PICO_SDK_PATH:?Вкажи шлях до pico-sdk 2.1.1}" &&
   cmake --build build/pico --target blink
   ```

3. Під’єднай плату в режимі BOOTSEL і скопіюй `blink.uf2` на змонтований диск або запиши файл через `picotool`. Після reset очікуй блимання LED. Для налагодження відкрий `blink.elf` у GDB потрібної архітектури й під’єднайся до OpenOCD на `localhost:3333`. Вибери скрипт `target/rp2040.cfg` або `target/rp2350.cfg`. Для Hazard3 у форку OpenOCD Raspberry Pi, що підтримує `USE_CORE`, передай `-c 'set USE_CORE {rv0 rv1}'` перед скриптом RP2350. [Debug Probe](https://www.raspberrypi.com/documentation/microcontrollers/debug-probe.html), [RP2350 OpenOCD](https://github.com/raspberrypi/openocd/blob/master/tcl/target/rp2350.cfg)

### Сетап без фреймворку

**Pico (RP2040).** Використовуй `c-pico-minimal` з власними startup, linker script і генератором UF2. На Ubuntu 24.04 x86-64 потрібні GNU Make, хостовий `gcc` та Arm GNU Toolchain `13.3.Rel1`. Збірка створює `build/blink.elf`, `build/blink.map` і `build/blink.uf2`, що записується у Flash від `0x10000000`. При холодному старті Boot ROM перевіряє CRC32 перших 252 байтів `boot2`, записаний у його останніх чотирьох байтах. `boot2` копіює застосунок у SRAM і запускає його. Скопіюй UF2 на `RPI-RP2` у BOOTSEL, від’єднай і знову під’єднай живлення та перевір блимання GPIO25. [Закріплений приклад](https://github.com/jameesbrown/c-pico-minimal/tree/edc2e9ac816fea0b4cca3fd792c10015ae065362), [Компонування образу](https://github.com/jameesbrown/c-pico-minimal/blob/edc2e9ac816fea0b4cca3fd792c10015ae065362/include/blink.ld), [RP2040 Datasheet, §2.8.1.3.1](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf#page=133)

```sh
git clone https://github.com/jameesbrown/c-pico-minimal.git rp2040-register-blink &&
git -C rp2040-register-blink checkout --detach edc2e9ac816fea0b4cca3fd792c10015ae065362 &&
make -C rp2040-register-blink
```

**Pico 2 (RP2350, Cortex-M33).** Для окремого bare-metal прикладу на Ubuntu 24.04 бери Arm GNU Toolchain `15.2.Rel1` (`arm-none-eabi`). Makefile отримує Pico SDK `2.2.0` для CMSIS, а власні linker script та `IMAGE_DEF` належать прикладу. Під’єднай Raspberry Pi Debug Probe через SWD, встанови OpenOCD з `target/rp2350.cfg`, і після збірки запиши `program.elf`. Перевір блимання GPIO25 після холодного запуску з Flash. Для Hazard3 потрібні RISC-V startup і відповідний `IMAGE_DEF`. [Зафіксований RP2350-приклад](https://github.com/metebalci/rp2350-bare-metal-build/tree/b1885574f90eec670684644692ac939af1fdc4b7), [RP2350 Datasheet](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf)

```sh
git clone https://github.com/metebalci/rp2350-bare-metal-build.git rp2350-register-blink &&
git -C rp2350-register-blink checkout --detach b1885574f90eec670684644692ac939af1fdc4b7 &&
make -C rp2350-register-blink &&
make -C rp2350-register-blink flash
```

## Перевірка й фіксація рішення

Зберігай ELF, карту лінкера і UF2 під конкретну плату та архітектуру. Для PIO перевір осцилограму, частоту й граничні таймінги під навантаженням із реальними налаштуваннями DMA та тактування. Для власної плати додай повторюваний холодний старт зі штатної Flash і перевір доступний режим відновлення. [Приклади PIO](https://github.com/raspberrypi/pico-examples/tree/master/pio), [Спільний процес випуску](firmware-development.md#випуск-і-відновлення)

Зафіксуй, який стек і архітектура пройшли перевірки, та за якої зміни таймінгів чи плати потрібно повторити вибір.
