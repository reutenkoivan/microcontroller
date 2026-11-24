# Microchip PIC16F18855 та dsPIC33CK256MP508

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для PIC16 визнач допустиму затримку реакції на сигнал та вимоги до EEPROM. Для dsPIC задай частоту PWM, фазу вимірювання ADC, допустиму затримку обчислення і стан виходів після reset або збою. [PIC16F18855](https://www.microchip.com/en-us/product/pic16f18855), [Приклад синхронізації PWM і ADC](https://github.com/microchip-pic-avr-examples/dspic33ck-power-pwm-adc-trigger)

## Цільова платформа

Для PIC16F18855 використовуй MPLAB Xpress Evaluation Board DM164140. Перевіряй LED2 на RA2 і програмуй плату через її USB-накопичувач. Для dsPIC33CK256MP508 використовуй Explorer 16/32 із модулем MA330042 та вбудованим PICkit (PKOB). Вивід RA3 модуля доступний на P25 плати. [MPLAB Xpress Pinout](https://ww1.microchip.com/downloads/en/DeviceDoc/30010190A.pdf), [Запис HEX через USB](https://ww1.microchip.com/downloads/en/DeviceDoc/MPLAB_Data_Visualizer_50003001.pdf), [Explorer 16/32 і MA330042](https://github.com/microchip-pic-avr-examples/dspic33ck-dac-dma-sinewave/tree/cf73ba0a1a86918f200689a5d19fecb026feef02), [Апаратна довідка](../wiki/1-hardware-platforms.md#17-pic--dspic)

## Вибір стека прошивки

**MCC** підходить, коли вибраний компонент підтримує потрібні тактування, виводи й периферію. **Bare metal** залишає конфігураційні біти та регістрове налаштування у власному коді, що доречно для невеликої цілі або спеціалізованого таймінгу. [MPLAB Code Configurator](https://www.microchip.com/en-us/tools-resources/configure/mplab-code-configurator)

## Межі для розробника

Признач власника конфігураційних бітів, тактування і зв’язку PWM з ADC. Розробник змінює логіку контролера у власному модулі й перевіряє її через наявний інтерфейс. Зміни налаштувань MCC або регістрового коду startup мають бути видимі в diff, а вплив на виходи перевіряється вимірюванням.

## Відтворюване середовище розробки

### Сетап через MPLAB Code Configurator

1. На Windows 11 x64 встанови MPLAB X 6.20 і MCC 5.7.0. Для PIC16F18855 вибери XC8 2.46 з `PIC16F1xxxx_DFP` 1.23.382. Створи проєкт MCC Classic, вибери внутрішній генератор, на стенді вимкни watchdog і налаштуй RA2 як цифровий вихід. Згенеруй код і після `SYSTEM_Initialize()` встанови RA2 у високий рівень через згенерований API Pin Manager. [XC8 2.46 і DFP](https://www.microchip.com/content/dam/mchp/documents/DEV/ProductDocuments/ReleaseNotes/xc8-v2.46-full-install-release-notes-PIC.pdf), [MCC](https://www.microchip.com/en-us/tools-resources/configure/mplab-code-configurator)
2. Для dsPIC імпортуй `dspic33ck-dac-dma-sinewave.X` з ревізії `cf73ba0a1a86918f200689a5d19fecb026feef02`. Використовуй XC-DSC 3.00, MCC Melody 2.6.5 і `dsPIC33CK-MP_DFP` 1.13.366. Збери наявний згенерований код. Після запису на P25 має бути синусоїда 100 Гц. [Версії та підключення](https://github.com/microchip-pic-avr-examples/dspic33ck-dac-dma-sinewave/tree/cf73ba0a1a86918f200689a5d19fecb026feef02), [Проєкт MPLAB X](https://github.com/microchip-pic-avr-examples/dspic33ck-dac-dma-sinewave/tree/cf73ba0a1a86918f200689a5d19fecb026feef02/dspic33ck-dac-dma-sinewave.X)
3. Для кожного проєкту окремо додай `make` із MPLAB X до `PATH` і виконай збірку з каталогу `.X`. Використовуй окремий чистий каталог для кожної CI-збірки. [Збірка поза MPLAB X](https://developerhelp.microchip.com/xwiki/bin/view/software-tools/ides/x/outside/building/)

   ```bat
   make CONF=default && make CONF=default TYPE_IMAGE=DEBUG_RUN
   ```

4. Після успішної збірки скопіюй production HEX із `dist/default/production/` на USB-накопичувач Xpress і перевір LED2 після перезапуску. Для Explorer 16/32 вибери PKOB у MPLAB X і виконай Make and Program Device. Для покрокової перевірки на Explorer 16/32 використовуй Debug Main Project. Для debug на Xpress потрібен зовнішній сумісний ICSP-адаптер через колодку J4 (позначену DNP на схемі). У CI зберігай конфігурації DFP, MCC і профіль оптимізації разом із логом. [Схема Xpress](https://ww1.microchip.com/downloads/en/DeviceDoc/MPLAB%20Xpress.pdf), [MPLAB CI/CD Wizard](https://developerhelp.microchip.com/xwiki/bin/view/software-tools/ides/x/cicd/)

### Сетап без фреймворку

1. Створи порожні MPLAB X проєкти з тими самими MCU, компіляторами та DFP. Додай `main.c` із `<xc.h>` і окремий файл конфігураційних бітів. Використовуй стартовий код і компонування, які вибирає компілятор для MCU. Для власного Makefile вибирай чип через `xc8-cc -mcpu=16F18855` або `xc-dsc-gcc -mcpu=33CK256MP508` і явно задай сумісний DFP, якщо він потрібен вибраній версії. [XC8 CLI](https://onlinedocs.microchip.com/oxy/GUID-BB433107-FD4E-4D28-BB58-9D4A58955B1A-en-US-9/GUID-F3413FD5-1EAD-45CE-9914-8A4B7B74F9B7.html), [XC-DSC для dsPIC33C](https://onlinedocs.microchip.com/oxy/GUID-C4E60FF5-3DAB-44F1-BA61-4BD962D8F469-en-US-9/GUID-CAB7CCD0-C7FD-4C3A-9448-0569A7CD11CD.html)
2. У конфігураційних бітах тестового образу вибери внутрішній генератор (HFINTOSC для PIC16 і FRC для dsPIC) і вимкни watchdog. Для PIC16 встанови `ANSELAbits.ANSA2 = 0`, `LATAbits.LATA2 = 1` і `TRISAbits.TRISA2 = 0`. Для dsPIC залиш DAC вимкненим і встанови `ANSELAbits.ANSELA3 = 0`, `LATAbits.LATA3 = 1` і `TRISAbits.TRISA3 = 0`. Заверши `main()` нескінченним циклом. [PIC16F18855 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/PIC16LF18855-75-Data-Sheet-40001802F.pdf), [dsPIC33CK256MP508 Datasheet](https://www.microchip.com/en-us/product/dspic33ck256mp508)
3. Збери обидва профілі командами з попереднього маршруту. Запиши свіжий production HEX через USB-накопичувач Xpress або PKOB Explorer 16/32 відповідно. Після холодного старту перевір LED2 або високий рівень на P25.

## Перевірка й фіксація рішення

Для PIC16 перевір стан виходів під час reset, обсяг RAM і збереження EEPROM, якщо вона входить у контракт випуску. Для dsPIC на стенді перевір синхронізацію PWM і зчитування ADC, реакцію на втрату живлення та перехід виходів у безпечний стан, якщо цього вимагає виріб. Зафіксуй ревізію плати, конфігураційні біти та версію production HEX. [Приклад PWM → ADC для dsPIC33CK](https://github.com/microchip-pic-avr-examples/dspic33ck-power-pwm-adc-trigger)

Зафіксуй вибір налаштування через MCC або регістри, версії компілятора й DFP та виміри, що підтверджують потрібну поведінку.
