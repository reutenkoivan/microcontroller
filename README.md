# База знань про розробку для мікроконтролерів

Україномовний тематичний довідник про мікроконтролери та розробку вбудованих систем.

## Зміст

- [1. Апаратні платформи](wiki/1-hardware-platforms.md)
  - [1.1. Порівняння платформ](wiki/1-hardware-platforms.md#11-порівняння-платформ)
  - [1.2. ESP32](wiki/1-hardware-platforms.md#12-esp32)
  - [1.3. STM32](wiki/1-hardware-platforms.md#13-stm32)
  - [1.4. Raspberry Pi RP2040 та RP2350](wiki/1-hardware-platforms.md#14-raspberry-pi-rp2040-та-rp2350)
  - [1.5. Nordic nRF](wiki/1-hardware-platforms.md#15-nordic-nrf)
  - [1.6. AVR / ATmega](wiki/1-hardware-platforms.md#16-avr--atmega)
  - [1.7. PIC / dsPIC](wiki/1-hardware-platforms.md#17-pic--dspic)
  - [1.8. NXP MCX / i.MX RT](wiki/1-hardware-platforms.md#18-nxp-mcx--imx-rt)
  - [1.9. TI MSP430 / SimpleLink](wiki/1-hardware-platforms.md#19-ti-msp430--simplelink)
  - [1.10. WCH CH32](wiki/1-hardware-platforms.md#110-wch-ch32)
  - [1.11. Спільне оснащення для роботи з платою](wiki/1-hardware-platforms.md#111-спільне-оснащення-для-роботи-з-платою)
- [2. RTOS для мікроконтролерів](wiki/2-rtos.md)
  - [2.1. Що таке RTOS](wiki/2-rtos.md#21-що-таке-rtos)
  - [2.2. Види вимог реального часу](wiki/2-rtos.md#22-види-вимог-реального-часу)
  - [2.3. Приклади RTOS](wiki/2-rtos.md#23-приклади-rtos)
- [3. Rust для мікроконтролерів](wiki/development/rust/3-rust.md)
  - [3.1. Системний дизайн прошивки та середовище розробки](wiki/development/rust/3.1-development-environment.md#31-системний-дизайн-прошивки-та-середовище-розробки)
    - [3.1.1. Вимоги й обмеження пристрою](wiki/development/rust/3.1-development-environment.md#311-вимоги-й-обмеження-пристрою)
    - [3.1.2. Цільова платформа](wiki/development/rust/3.1-development-environment.md#312-цільова-платформа)
    - [3.1.3. Вибір стека прошивки](wiki/development/rust/3.1-development-environment.md#313-вибір-стека-прошивки)
    - [3.1.4. Межі для розробника](wiki/development/rust/3.1-development-environment.md#314-межі-для-розробника)
    - [3.1.5. Відтворюване середовище розробки](wiki/development/rust/3.1-development-environment.md#315-відтворюване-середовище-розробки)
    - [3.1.6. Перевірка й фіксація рішення](wiki/development/rust/3.1-development-environment.md#316-перевірка-й-фіксація-рішення)
  - [3.2. Середовище виконання та `no_std`](wiki/development/rust/3.2-runtime-no-std.md#32-середовище-виконання-та-no_std)
  - [3.3. Доступ до периферії та `embedded-hal`](wiki/development/rust/3.3-peripherals-embedded-hal.md#33-доступ-до-периферії-та-embedded-hal)
  - [3.4. Переривання, асинхронність та Embassy](wiki/development/rust/3.4-interrupts-async-embassy.md#34-переривання-асинхронність-та-embassy)
  - [3.5. Підтримка ESP32, STM32, Raspberry Pi Pico та Nordic nRF](wiki/development/rust/3.5-platform-support.md#35-підтримка-esp32-stm32-raspberry-pi-pico-та-nordic-nrf)
  - [3.6. Збірка, прошивання й налагодження](wiki/development/rust/3.6-build-flash-debug.md#36-збірка-прошивання-й-налагодження)

## Структура репозиторію

- [`wiki/`](wiki/) — статті й довідкові матеріали про мікроконтролери та вбудовану розробку.
- [`docs/`](docs/) — документація про структуру репозиторію, його правила та робочі процеси.
