# Налаштування командної розробки прошивок

Почни зі [спільного процесу](firmware-development.md), потім вибери гайд для своєї плати. Кожен маршрут задає тестову плату й першу перевірку. Для власного виробу зафіксуй цільовий чип, ревізію плати, версії інструментів і характеристики, які має підтвердити стенд.

Кожен гайд містить два маршрути. Перший використовує SDK, бібліотеки або фреймворк виробника. Другий показує bare-metal налаштування без фреймворку, включно з обов’язковими для чипа компонентами запуску й обмеженнями щодо радіо, завантаження або підтримки інструментів.

| Платформа | Тестова ціль | Базовий стек |
| --- | --- | --- |
| [ESP32](esp32.md) | ESP32-C3-DevKitM-1 і Waveshare ESP32-S3-Zero | ESP-IDF |
| [STM32](stm32.md) | NUCLEO-G474RE | STM32CubeMX, STM32CubeG4, Arm GNU Toolchain |
| [Raspberry Pi RP2040 та RP2350](raspberry-pi-rp2040-rp2350.md) | Pico і Pico 2 | Pico C/C++ SDK, CMake |
| [Nordic nRF](nordic-nrf.md) | nRF52840 DK | nRF Connect SDK, Zephyr, `west` |
| [AVR / ATmega](avr-atmega.md) | Arduino Uno R3 з ATmega328P | Arduino CLI, Arduino AVR Core |
| [PIC / dsPIC](pic-dspic.md) | MPLAB Xpress DM164140 і Explorer 16/32 з MA330042 | MPLAB X, MCC, XC8 / XC-DSC |
| [NXP MCX / i.MX RT](nxp-mcx-imx-rt.md) | FRDM-MCXN947 і MIMXRT1060-EVKB | MCUXpresso SDK, Arm GNU Toolchain, LinkServer |
| [TI MSP430 / SimpleLink](ti-msp430-simplelink.md) | MSP-EXP430FR2433 і LAUNCHXL-CC26X2R1 | Code Composer Studio, SDK відповідної родини |
| [WCH CH32](wch-ch32.md) | CH32V003F4 EVT, CH32V203C8 EVT і CH32F203C8T6 EVT | WCH Arduino Core / EVT, WCH GCC / Arm GNU Toolchain, WCH-LinkE |

Якщо платформа ще не визначена, порівняй [апаратні вимоги](../wiki/1-hardware-platforms.md#11-порівняння-платформ) і [моделі виконання](../wiki/2-rtos.md#21-вибір-між-основним-циклом-embassy-та-rtos). Окремий маршрут для Rust описано в [огляді підтримуваних платформ](../wiki/development/rust/3.5-platform-support.md).

Гайди проходять від вимог до перевірки рішення за структурою розділу 3.1. Під час підготовки використано робочу версію Engineering Compass із `tan-i18n` станом на 24 листопада 2026 року. Чинну версію Engineering Compass для репозиторію задає `AGENTS.md`. [Розділ 3.1](../wiki/development/rust/3.1-development-environment.md), [Чинний запис Engineering Compass](../AGENTS.md#engineering-compass)
