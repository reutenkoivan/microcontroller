# 1. Апаратні платформи

[До змісту](../README.md#зміст)

Платформу зручно оцінювати від задачі пристрою. Для бездротового датчика важливі радіопротокол і режими сну, для двигуна — таймери, АЦП та можливість синхронізувати вимірювання, для дисплея — обсяг пам’яті й інтерфейс виводу. Сімейство визначає загальний набір можливостей, чип — конкретні характеристики, а плата або модуль додають живлення, пам’ять, антени й роз’єми.

## 1.1. Порівняння платформ

У таблиці наведено характеристики конкретних чипів, модулів або плат. **SRAM** — робоча пам’ять для стека, змінних і буферів, **Flash або FRAM** — енергонезалежна пам’ять програми. Для ESP32 і Pico місце встановлення пам’яті зазначено окремо. Для PIC16 наведено частоту генератора, а для dsPIC — продуктивність у мільйонах інструкцій за секунду. Функції виводів і доступність периферії перевіряють за документацією конкретного корпусу та схемою плати.

| Модель або плата | Ядро й максимальна частота | SRAM | Пам’ять програми | Типова задача |
| --- | --- | --- | --- | --- |
| [ESP32-C3, модуль WROOM-02-N4](#12-esp32) | 1× RISC-V, 160 МГц | 400 КБ | 4 МБ Flash у модулі | Датчик або контролер із Wi-Fi та Bluetooth LE |
| [ESP32-S3, модуль WROOM-1-N8R8](#12-esp32) | 2× Xtensa LX7, 240 МГц | 512 КБ + 8 МБ PSRAM | 8 МБ Flash у модулі | Дисплей, аудіо або робота з кадрами |
| [STM32G474RE](#13-stm32) | Cortex-M4F, 170 МГц | 128 КБ | 512 КБ Flash | Керування двигуном і синхронні вимірювання |
| [RP2040 на платі Pico](#14-raspberry-pi-rp2040-та-rp2350) | 2× Cortex-M0+, 133 МГц | 264 КБ | 2 МБ Flash на платі | Нестандартні цифрові інтерфейси |
| [RP2350 на платі Pico 2](#14-raspberry-pi-rp2040-та-rp2350) | 2× Cortex-M33 або 2× Hazard3 RISC-V, 150 МГц | 520 КБ | 4 МБ Flash на платі | Керування периферією з більшим буфером даних |
| [Nordic nRF52840](#15-nordic-nrf) | Cortex-M4F, 64 МГц | 256 КБ | 1 МБ Flash | Бездротові датчики та Bluetooth LE-пристрої |
| [ATmega328P на Arduino Uno R3](#16-avr--atmega) | 8-бітний AVR, 16 МГц на платі | 2 КБ | 32 КБ Flash | Невеликий автономний контролер |
| [PIC16F18855](#17-pic--dspic) | 8-бітний PIC, Fosc до 32 МГц | 1 КБ | 14 КБ Flash | Прості вимірювання та апаратна логіка |
| [dsPIC33CK256MP508](#17-pic--dspic) | 16-бітний dsPIC, до 100 MIPS | 24 КБ | 256 КБ Flash | Керування двигунами й цифрові контури регулювання |
| [NXP MCXN947](#18-nxp-mcx--imx-rt) | 2× Cortex-M33, до 150 МГц | 512 КБ | 2 МБ Flash | Пристрій із кількома шинами та локальною обробкою даних |
| [NXP i.MX RT1062, варіант на 600 МГц](#18-nxp-mcx--imx-rt) | Cortex-M7, 600 МГц | 1 МБ | Зовнішня Flash | Графічна панель, камера або аудіо |
| [TI MSP430FR2433](#19-ti-msp430--simplelink) | 16-бітний MSP430, 16 МГц | 4 КБ | 15 КБ програмної FRAM | Автономний вимірювач із журналом |
| [TI SimpleLink CC2652R](#19-ti-msp430--simplelink) | Cortex-M4F, 48 МГц | 80 КБ + 8 КБ кешу | 352 КБ Flash | Батарейний вузол Zigbee або Thread |
| [WCH CH32V003](#110-wch-ch32) | QingKe V2A RISC-V, 48 МГц | 2 КБ | 16 КБ Flash | Плата кнопок, індикатор або простий датчик |
| [WCH CH32V203](#110-wch-ch32) | QingKe V4B RISC-V, 144 МГц | 20 КБ | 64 КБ Flash у наведеній конфігурації | Плата вводу-виводу з USB або CAN |

## 1.2. ESP32

ESP32-C3 і ESP32-S3 поєднують мікроконтролер, Wi-Fi 2,4 ГГц та Bluetooth LE. C3 використовує RISC-V. S3 має два ядра Xtensa LX7 і векторні інструкції для обробки сигналів. У версії модуля S3-WROOM-1-N8R8 є додаткова PSRAM, яку можна використати для кадрів і великих буферів даних. [ESP32-C3](https://documentation.espressif.com/esp32-c3_datasheet_en.html), [ESP32-S3](https://documentation.espressif.com/esp32_s3_datasheet_en.pdf), [Модулі C3](https://documentation.espressif.com/esp32-c3-wroom-02_datasheet_en.html), [Модулі S3](https://documentation.espressif.com/api/resource/doc/file/3yD6w5Y5/FILE/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf)

**Периферія.** У C3 є GPIO, UART, SPI, I²C, I²S, контролер CAN-сумісної шини TWAI, USB Serial/JTAG, PWM для світлодіодів, RMT для імпульсних сигналів, 12-бітні АЦП і датчик температури. Системні блоки охоплюють DMA, таймери, watchdog, RTC-домен і криптографічні прискорювачі. S3 також має USB OTG, периферію LCD і камери, SD/MMC, ємнісні сенсори, лічильник імпульсів і PWM для двигунів. [ESP32-C3 Datasheet](https://documentation.espressif.com/esp32-c3_datasheet_en.html), [ESP32-S3 Datasheet](https://documentation.espressif.com/esp32_s3_datasheet_en.pdf)

**Інструменти розробки.** Для C/C++ використовують ESP-IDF із компілятором під архітектуру чипа, CMake і Ninja. CLI ESP-IDF керує конфігурацією й збіркою, esptool записує Flash через послідовний інтерфейс, а IDF Monitor показує журнал. OpenOCD і GDB забезпечують покрокове налагодження через доступний JTAG-інтерфейс. На S3 це може бути вбудований USB Serial/JTAG. Дивись також [набір інструментів Rust](development/rust/3.1-development-environment.md#312-цільова-платформа). [Початок роботи з ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/get-started/index.html), [JTAG на ESP32-S3](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-guides/jtag-debugging/configure-builtin-jtag.html)

**Приклади.** C3 може зчитувати датчик по I²C і публікувати вимірювання по Wi-Fi. S3 підходить для настінної панелі з дисплеєм або пристрою, який обробляє кадри з камери у буфері PSRAM.

## 1.3. STM32

STM32 охоплює мікроконтролери Arm Cortex-M з різними наборами периферії. STM32G474RE вирізняється аналоговими блоками, точними таймерами та апаратним обчисленням тригонометричних функцій і цифрової фільтрації за допомогою CORDIC і FMAC. [STM32G474RE](https://www.st.com/en/microcontrollers-microprocessors/stm32g474re.html)

**Периферія STM32G474RE.** Для вимірювання є 12-бітні АЦП, канали ЦАП, компаратори, операційні підсилювачі й опорна напруга. Для керування — таймер HRTIM, моторні PWM, універсальні й малопотужний таймери, RTC, DMA та watchdog. До інтерфейсів належать GPIO, UART/USART, I²C, SPI/I²S, SAI, FDCAN, USB Device, контролер USB Type-C Power Delivery (UCPD) і Quad-SPI. Доступність сигналів визначає таблиця альтернативних функцій виводів. [Опис блоків STM32G474RE](https://www.st.com/en/microcontrollers-microprocessors/stm32g474re.html)

**Інструменти розробки.** STM32CubeG4 містить HAL, LL і приклади для плат. STM32CubeMX налаштовує виводи, тактування й периферію. STM32CubeIDE забезпечує збірку через Arm GCC та налагодження, STM32CubeProgrammer — запис і перевірку Flash. Плата NUCLEO-G474RE має вбудований ST-LINK для SWD. [STM32CubeG4](https://www.st.com/resource/en/user_manual/um2492-getting-started-with-stm32cubeg4-for-stm32g4-series-stmicroelectronics.pdf), [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html), [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html), [Плата Nucleo](https://www.st.com/resource/en/user_manual/dm00556337.pdf)

**Приклади.** У контролері двигуна таймер задає ШІМ і синхронізує вимірювання струму. У вимірювальному модулі АЦП і DMA збирають серії відліків, а FMAC може виконувати цифрову фільтрацію.

## 1.4. Raspberry Pi RP2040 та RP2350

RP2040 лежить в основі першого Pico, RP2350 — Pico 2. На RP2350 режим завантаження обирає пару ядер Cortex-M33 або Hazard3 RISC-V. Плати з літерою W мають окремий радіочип для бездротового зв’язку. [Мікроконтролери RP](https://www.raspberrypi.com/documentation/microcontrollers/microcontroller-chips.html), [Плати Pico](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html)

**Периферія.** Обидва чипи мають GPIO, UART, SPI, I²C, USB Device/Host, PWM, 12-бітний АЦП, датчик температури, DMA, таймери та watchdog. PIO виконує точні послідовності сигналів на виводах. RP2040 має 8 машин станів PIO та інтерфейс QSPI для зовнішньої Flash і RTC. RP2350 має 12 машин станів PIO, інтерфейс QMI для зовнішньої Flash і PSRAM та HSTX для швидкого цифрового виводу. [Порівняння периферії RP2040 і RP2350](https://www.raspberrypi.com/documentation/microcontrollers/microcontroller-chips.html)

**Інструменти розробки.** Pico SDK надає бібліотеки C/C++, систему збірки на CMake та приклади. Компілятор вибирають під Arm або RISC-V. Для запису підходять UF2 через USB у режимі BOOTSEL або `picotool`. Для покрокового налагодження використовують Raspberry Pi Debug Probe через SWD з OpenOCD і GDB у конфігурації під вибрану ціль. Для інтерактивних прототипів є MicroPython. [Pico SDK](https://www.raspberrypi.com/documentation/microcontrollers/c_sdk.html), [picotool](https://github.com/raspberrypi/picotool), [Raspberry Pi Debug Probe](https://www.raspberrypi.com/documentation/microcontrollers/debug-probe.html)

**Приклади.** PIO може передавати дані на стрічку WS2812 або зчитувати сигнали квадратурного енкодера, поки процесор обробляє вимірювання. [Приклади Pico SDK](https://github.com/raspberrypi/pico-examples)

## 1.5. Nordic nRF

nRF52840 поєднує Cortex-M4F і радіо 2,4 ГГц для Bluetooth LE та IEEE 802.15.4. Останній використовується, зокрема, у Thread і Zigbee. Є режими сну з пробудженням за таймером або зовнішньою подією, що дає змогу вимикати частину блоків між вимірюваннями. [nRF52840](https://www.nordicsemi.com/Products/nRF52840), [Специфікація nRF52840](https://docs.nordicsemi.com/r/bundle/ps_nrf52840/page/keyfeatures_html5.html?contentId=lQyrwMZHlFE5yAQpDbfoWQ)

**Периферія nRF52840.** Доступні GPIO і GPIOTE для подій на виводах, UART, SPI/QSPI, I²C-сумісний TWI, I²S, PDM, USB Device, NFC-A, радіоконтролер і CryptoCell. Вимірювання та керування забезпечують 12-бітний АЦП, компаратори, датчик температури, PWM, квадратурний декодер, таймери, RTC та watchdog. EasyDMA переносить дані, а PPI пов’язує події й дії периферійних блоків. [Специфікація nRF52840](https://docs.nordicsemi.com/r/bundle/ps_nrf52840/page/keyfeatures_html5.html?contentId=lQyrwMZHlFE5yAQpDbfoWQ)

**Інструменти розробки.** nRF Connect SDK на базі Zephyr і його комплект компілятора встановлюють через nRF Connect for VS Code або nRF Util. Збірку й запис прошивки можна виконувати через `west`. Для запису застосовується `nrfutil device`, а для налагодження через SWD у nRF52840 DK є вбудований SEGGER J-Link. [Встановлення SDK](https://nrfconnectdocs.nordicsemi.com/ncs/latest/nrf/installation/install_ncs.html), [Програмування плати](https://nrfconnectdocs.nordicsemi.com/ncs/latest/nrf/app_dev/programming.html), [nRF52840 DK](https://www.nordicsemi.com/Products/Development-hardware/nRF52840-DK)

**Приклади.** Батарейний датчик дверей може прокидатися від зміни GPIO і надсилати подію через Thread. Пульт або клавіатура можуть передавати дії через Bluetooth LE.

## 1.6. AVR / ATmega

ATmega328P — 8-бітний AVR, на якому побудована Arduino Uno R3. Окрім Flash і SRAM, він має 1 КБ EEPROM для налаштувань і калібрувальних даних. За 2 КБ SRAM варто заздалегідь оцінити розміри буферів і стека. [ATmega328P](https://www.microchip.com/en-us/product/ATmega328P), [Arduino Uno R3](https://docs.arduino.cc/hardware/uno-rev3/)

**Периферія ATmega328P.** GPIO та переривання на виводах, три таймери з порівнянням і PWM, 10-бітний АЦП, аналоговий компаратор, USART, SPI, TWI (I²C), watchdog і контроль зниження напруги живлення. Кількість доступних аналогових входів залежить від корпусу, а виводи на роз’ємах — від плати. [ATmega328P](https://www.microchip.com/en-us/product/ATmega328P)

**Інструменти розробки.** Для Uno R3 підійдуть Arduino IDE або Arduino CLI з Arduino AVR Core. Завантажувач плати приймає прошивку через USB-послідовний інтерфейс. Для роботи з окремим чипом або покрокових перевірок є MPLAB X і XC8 для AVR, програмування через ISP та налагодження через debugWIRE з підтримуваним адаптером, наприклад PICkit 5. [Arduino AVR Core](https://github.com/arduino/ArduinoCore-avr), [AVR debugWIRE у MPLAB X](https://developerhelp.microchip.com/xwiki/bin/view/software-tools/ides/x/debugging/avr-debugwire/), [PICkit 5](https://www.microchip.com/en-us/development-tool/pg164150)

**Приклади.** Таймер поливу, панель кнопок з індикатором або контролер вентилятора, який зчитує аналоговий датчик і зберігає поріг в EEPROM.

## 1.7. PIC / dsPIC

**PIC16F18855.** Це 8-бітний контролер із 256 Б EEPROM та апаратною логікою. Периферія охоплює GPIO, 10-бітний АЦП з апаратними обчисленнями, ЦАП, компаратори, детектор переходу через нуль, таймери й PWM, конфігуровані логічні комірки CLC, генератори сигналів NCO/CWG, EUSART, SPI/I²C, watchdog, CRC та сканування пам’яті. Логічні комірки дають змогу обробляти прості події без ручного опитування сигналів у головному циклі. [PIC16F18855 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/PIC16LF18855-75-Data-Sheet-40001802F.pdf)

**dsPIC33CK256MP508.** 16-бітний контролер цифрових сигналів має блок множення з накопиченням для обчислень у контурі керування. Периферія включає швидкі 12-бітні АЦП, високоточні PWM, ЦАП, компаратори й операційні підсилювачі, інтерфейс квадратурного енкодера (QEI), таймери, захоплення й порівняння сигналів, DMA, UART, SPI/I²S, I²C, SENT, CAN FD і watchdog. [dsPIC33CK256MP508 Datasheet](https://www.microchip.com/content/dam/mchp/documents/MCU16/ProductDocuments/DataSheets/dsPIC33CK256MP508-Family-Data-Sheet-DS70005349J.pdf)

**Інструменти розробки.** MPLAB X IDE використовують із компілятором XC8 для PIC16 або XC-DSC для dsPIC33. MCC допомагає налаштувати виводи й підтримувану периферію. Для прошивання та налагодження використовують PICkit 5 через ICSP, для окремого запису прошивки — MPLAB IPE. Для перевірки алгоритмів двигуна Microchip має плати з dsPIC33CK256MP508 і прикладами керування. [Компілятори MPLAB](https://onlinedocs.microchip.com/oxy/GUID-D79ACEBE-41BD-43EF-8E1B-9462847AE13E-en-US-12/GUID-9C77E750-D06A-4B4C-9AE2-1769840A597B.html), [MCC](https://www.microchip.com/en-us/tools-resources/configure/mplab-code-configurator), [PICkit 5 та MPLAB IPE](https://www.microchip.com/content/dam/mchp/documents/DEV/ProductDocuments/UserGuides/MPLAB-PICkit-5-In-Circuit-Debugger-Users-Guide-50003525.pdf), [Плата для керування двигуном](https://www.microchip.com/en-us/development-tool/EV62P66A)

**Приклади.** PIC16 може періодично вимірювати напругу й керувати індикатором. dsPIC може вимірювати фазні струми та перераховувати ШІМ у циклі керування безщітковим двигуном.

## 1.8. NXP MCX / i.MX RT

**MCXN947.** Два Cortex-M33 і прискорювач eIQ Neutron NPU дозволяють відокремити прикладні задачі від обробки сигналів та виконувати підтримувані моделі на пристрої. Периферія включає GPIO, таймери й PWM, АЦП, ЦАП, компаратори й операційні підсилювачі, сенсорні входи, DMA, Flexcomm для UART/SPI/I²C, I3C, SAI, FlexIO, два CAN FD, USB High-Speed, Ethernet, uSDHC і FlexSPI. Для Ethernet на FRDM-MCXN947 є окремий PHY. [MCX N94x Datasheet](https://www.nxp.com/docs/en/data-sheet/MCXNx4xDS.pdf), [MCX N Series](https://www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/general-purpose-mcus/mcx-arm-cortex-m/mcx-n-series-microcontrollers:MCX-N-SERIES), [FRDM-MCXN947](https://docs.nxp.com/bundle/UM12018/page/topics/Evk_overview.html)

**i.MX RT1062.** Cortex-M7 і 1 МБ SRAM дають місце для буферів дисплея, аудіо й даних. Периферія охоплює LCD, камеру, 2D-графіку, DMA, GPIO, таймери, PWM, АЦП і компаратори, FlexIO, UART/SPI/I²C, SAI та S/PDIF, USB, Ethernet, CAN, SD/eMMC і контролери зовнішньої Flash та оперативної пам’яті. Для плати з RT1062 вибір зовнішньої Flash і режиму завантаження є частиною налаштування проєкту. [i.MX RT1060 Datasheet](https://www.nxp.com/docs/en/nxp/data-sheets/IMXRT1060CEC.pdf)

**Інструменти розробки.** MCUXpresso SDK забезпечує драйвери й приклади. Для збірки є MCUXpresso IDE або MCUXpresso for VS Code з Arm GNU Toolchain. Config Tools допомагають налаштувати виводи, тактування та периферію. LinkServer підтримує запис і налагодження з вбудованими адаптерами MCU-Link на FRDM-MCXN947 та LPC-Link2 на MIMXRT1060-EVKB. Для зовнішньої Flash проєкт і засіб програмування мають відповідати конфігурації плати. [Початок роботи з FRDM-MCXN947](https://www.nxp.com/document/guide/getting-started-with-frdm-mcxn947%3AGS-FRDM-MCXN947), [Адаптери плат](https://mcuxpresso.nxp.com/mcuxsdk/latest/gsd/package/default_debug_interfaces.html), [MIMXRT1060-EVKB](https://mcuxpresso.nxp.com/mcuxsdk/25.03.00/html/boards/RT/evkbmimxrt1060/index.html)

**Приклади.** MCXN947 може збирати вимірювання з кількох шин і класифікувати сигнали перед передаванням. RT1062 може одночасно керувати графічною панеллю та обмінюватися даними через Ethernet.

## 1.9. TI MSP430 / SimpleLink

**MSP430FR2433.** 16-бітний мікроконтролер із FRAM. Крім 15 КБ програмної FRAM, є 512 Б окремої інформаційної FRAM. Периферія охоплює GPIO з перериваннями, 10-бітний АЦП, опорну напругу й внутрішній датчик температури, таймери з PWM, RTC, watchdog, UART з IrDA, SPI, I²C та CRC. FRAM зручна для журналу, який часто оновлюється і має зберігатися після вимкнення живлення. [MSP430FR2433 Datasheet](https://www.ti.com/lit/ds/symlink/msp430fr2433.pdf)

**SimpleLink CC2652R.** 32-бітний мікроконтролер з радіо 2,4 ГГц для Bluetooth LE та протоколів на кшталт Thread і Zigbee. Він має автономний контролер датчиків, GPIO, 12-бітний АЦП, компаратори з опорним ЦАП, ємнісні входи, датчик температури, монітор батареї, таймери, RTC, UART, SPI, I²C, I²S і криптографічні блоки. Його 8 КБ кешу можна переналаштувати як оперативну пам’ять. [CC2652R Datasheet](https://www.ti.com/lit/ds/symlink/cc2652r.pdf)

**Інструменти розробки.** Для MSP430 — Code Composer Studio з компілятором MSP430 і бібліотеками MSP430 SDK. Плата MSP-EXP430FR2433 має програматор і налагоджувач eZ-FET. Для CC2652R — Code Composer Studio, SimpleLink Low Power F2 SDK, SysConfig і XDS110 на LAUNCHXL-CC26X2R1. UniFlash призначений для окремого запису прошивок. [MSP430 SDK](https://www.ti.com/tool/MSP430-SDK), [MSP430 LaunchPad](https://www.ti.com/tool/MSP-EXP430FR2433), [SimpleLink SDK](https://www.ti.com/tool/SIMPLELINK-LOWPOWER-SDK), [SysConfig](https://www.ti.com/tool/SYSCONFIG), [CC2652R LaunchPad](https://www.ti.com/tool/LAUNCHXL-CC26X2R1), [UniFlash](https://www.ti.com/tool/UNIFLASH)

**Приклади.** MSP430FR2433 може вести журнал температури без мережі. CC2652R може періодично опитувати датчик і відправляти вимірювання через Thread, використовуючи режим сну між сеансами.

## 1.10. WCH CH32

У серії CH32 є RISC-V чипи QingKe, зокрема CH32V003 і CH32V203, та окремі сімейства Arm Cortex-M3, зокрема CH32F203. Для кожного потрібні відповідні бібліотеки й конфігурація компілятора. [CH32V003](https://github.com/openwch/ch32v003), [CH32V203](https://github.com/openwch/ch32v20x), [CH32F203](https://www.wch.cn/products/CH32F203.html)

**Периферія CH32V003 та CH32V203.** У V003 є GPIO, UART, SPI, I²C, 10-бітний АЦП, аналоговий компаратор, таймери з PWM, DMA та однопровідний інтерфейс налагодження. V203 додає 12-бітний АЦП, операційні підсилювачі, ємнісні входи TouchKey, контролер CAN 2.0B, USB Full Speed з режимами Device та Host/Device, більше послідовних інтерфейсів і двопровідне налагодження. Доступність виводів залежить від корпусу. [Периферія CH32V003](https://github.com/openwch/ch32v003), [Периферія CH32V203](https://github.com/openwch/ch32v20x)

**Периферія CH32F203.** У варіантах Arm Cortex-M3 до 144 МГц доступні таймери з моторним PWM, 12-бітні АЦП і ЦАП, аналогові підсилювачі та компаратори, TouchKey, UART, I²C, SPI/I²S, USB Full Speed, CAN і SDIO. Набір сигналів залежить від варіанта й корпусу. [WCH CH32F203](https://www.wch.cn/products/CH32F203.html), [Довідник CH32F203](https://www.wch.cn/uploads/file/20230404/1680573065146901.pdf)

**Інструменти розробки.** Пакет EVT від WCH містить бібліотеку периферії, приклади та схеми плат. Для CH32V типовий набір — MounRiver Studio або компілятор WCH RISC-V GCC, GDB і версія OpenOCD для WCH-LinkE. Для V003 і V203 обирають відповідний режим налагодження адаптера. WCH також підтримує ці моделі у своєму Arduino Core. Для CH32F203 використовують пакет CH32F20x EVT і компілятор під Arm, наприклад у Keil MDK. WCH-LinkE під’єднують через SWD у режимі Arm. [MounRiver Studio](https://mounriver.com/), [Інструментарій та Arduino Core WCH](https://github.com/openwch/arduino_core_ch32), [CH32F20x EVT](https://www.wch.cn/downloads/CH32F20xEVT_ZIP.html), [WCH-Link і SWD](https://www.wch.cn/products/wch-link)

**Приклади.** V003 підходить для клавіатурної матриці, сигналізації стану або керування індикатором. V203 можна використати для USB-панелі керування чи CAN-вузла з окремим трансивером.

## 1.11. Спільне оснащення для роботи з платою

| Задача | Що потрібно |
| --- | --- |
| Під’єднати плату | Плата розробки, USB-кабель із лініями даних або джерело живлення під плату. Для роботи з датчиками — макетне поле, дроти та сумісні рівні сигналів. |
| Записати й налагодити прошивку | Налагоджувальний адаптер або вбудований програматор під конкретну платформу, драйвер за потреби та її програма прошивання. Підтримуваний завантажувач може забезпечити запис через USB або UART. |
| Зібрати журнали | Віртуальний послідовний порт плати, USB–UART-адаптер або RTT через підтримуваний налагоджувач. Для UART звіряють рівень сигналу й швидкість порту. |
| Перевірити сигнали й живлення | Мультиметр для напруги та струму, логічний аналізатор для цифрових шин, осцилограф для форми сигналу й часових затримок. Для батарейних пристроїв корисний профілювальник струму, якщо потрібно побачити окремо сон та активність. |
| Перевірити всю функцію | Зовнішня периферія, потрібна для сценарію, наприклад датчик I²C, дисплей SPI або CAN-трансивер. Для радіопротоколів перевіряють роботу з другим пристроєм чи шлюзом. |

Процес запуску та перевірки прошивки описано в [«Збірка, прошивання й налагодження»](development/rust/3.6-build-flash-debug.md).

Початковий список сімейств наведено в [огляді екосистеми мікроконтролерів](chatgpt-conversation://6b04a2c2-f338-83eb-a441-ece177062370).
