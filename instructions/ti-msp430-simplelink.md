# TI MSP430FR2433 та SimpleLink CC2652R

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для журналу на MSP430 задай швидкість накопичення записів і стан, допустимий після обриву живлення посеред запису. Для вузла на CC2652R визнач радіопротокол, час повторного з’єднання та бюджет енергоспоживання. [MSP430FR2433 Datasheet](https://www.ti.com/lit/ds/symlink/msp430fr2433.pdf), [CC2652R Datasheet](https://www.ti.com/lit/ds/symlink/cc2652r.pdf)

## Цільова платформа

Для MSP430FR2433 використовуй `MSP-EXP430FR2433` із вбудованим eZ-FET, для CC2652R — `LAUNCHXL-CC26X2R1` з XDS110. Визнач, які області FRAM або Flash мають переживати запис нового образу. [MSP430 LaunchPad](https://www.ti.com/tool/MSP-EXP430FR2433), [SimpleLink LaunchPad](https://www.ti.com/tool/LAUNCHXL-CC26X2R1), [Апаратна довідка](../wiki/1-hardware-platforms.md#19-ti-msp430--simplelink)

## Вибір стека прошивки

**MSP430 SDK або SimpleLink SDK** використовуй, коли доступні потрібні драйвери чи радіостек. **Bare metal** підійде для простого локального вимірювання, GPIO або журналу, якщо команда підтримує стартову конфігурацію й доступ до пам’яті. [MSP430 SDK](https://www.ti.com/tool/MSP430-SDK), [SimpleLink Low Power F2 SDK](https://www.ti.com/tool/SIMPLELINK-LOWPOWER-SDK)

## Межі для розробника

Закріпи карту ділянок, які має зберігати запис нового образу. Для CC2652R визнач власника CCFG і SysConfig за його наявності. Розробник змінює формат запису або логіку через наявний інтерфейс, а рев’ю linker configuration і стендовий reset показують, чи зачеплено збережені дані.

## Відтворюване середовище розробки

### Сетап через SDK і Code Composer Studio

1. На Windows 10 x64 встанови Code Composer Studio (CCS) 12.8.0 із підтримкою обох серій. Для **MSP430FR2433** встанови TI MSP430 CGT 21.6.2.LTS і MSP430 DriverLib 2.91.13.01. Створи проєкт `MSP430FR2433` із прикладу GPIO для `MSP430FR2xx_4xx` або додай DriverLib цієї родини до порожнього проєкту. Для першого тесту зупини watchdog, налаштуй P1.0 як вихід із високим рівнем, виклич `PMM_unlockLPM5()` і перевір червоний LED1 із установленою перемичкою J10. [TI MSP430 CGT](https://software-dl.ti.com/codegen/esd/cgt_public_sw/MSP430/21.6.2.LTS/README.html), [DriverLib і приклади](https://software-dl.ti.com/msp430/msp430_public_sw/mcu/msp430/MSP430_Driver_Library/2_91_13_01/exports/driverlib/msp430_driverlib_2_91_13_01/release_notes.html), [Посібник MSP-EXP430FR2433](https://www.ti.com/lit/ug/slau739/slau739.pdf)
2. Для **CC2652R** встанови SimpleLink Low Power F2 SDK 8.33.00.16 і TI Arm Clang 3.2.2.LTS. Використовуй SysConfig із установленого CCS 12.8.0. Імпортуй `gpiointerrupt` для `CC26X2R1_LAUNCHXL` із прикладів TI Drivers для TI-RTOS7, збережи його `.syscfg` і `.ccxml`. Після запису кнопка має перемикати LED прикладу. Для радіопротоколу імпортуй приклад відповідного стека для цієї плати. [SimpleLink SDK 8.33.00.16 Release Notes](https://software-dl.ti.com/simplelink/esd/simplelink_cc13xx_cc26xx_sdk/8.33.00.16/exports/release_notes_simplelink_cc13xx_cc26xx_sdk_8_33_00_16.html), [Обробка кнопок і LED на LaunchPad](https://software-dl.ti.com/ccs/esd/training/workshop/ccsv10/ccs_simplelink_fundamentals_workshop.html)
3. Для headless-збірки з `cmd.exe` додай каталог `ccs\eclipse` із CCS 12.8.0 до `PATH`. Задай `CCS_WORKSPACE` як новий каталог, `PROJECT_DIR` як шлях до CCS-проєкту й `PROJECT_NAME` як його ім’я з `.project`. Задай `CCS_CONFIGURATION` за конфігурацією імпортованого проєкту. [CLI класичного CCS](https://software-dl.ti.com/ccs/esd/documents/ccs_projects-command-line.html)

   ```bat
   eclipsec.exe -noSplash -data "%CCS_WORKSPACE%" -application com.ti.ccstudio.apps.projectImport -ccs.location "%PROJECT_DIR%" && eclipsec.exe -noSplash -data "%CCS_WORKSPACE%" -application com.ti.ccstudio.apps.projectBuild -ccs.projects "%PROJECT_NAME%" -ccs.configuration "%CCS_CONFIGURATION%" -ccs.buildType full
   ```

4. Після успішної збірки в UniFlash відкрий `.ccxml` цієї плати й щойно створений `.out`. Перевір налаштування стирання та діапазони оновлення, виконай Program і Verify. Для автоматизації збережи параметри цілі й стирання через Generate CLI Package. У CCS запускай debug із тим самим `.out` через eZ-FET або XDS110. [CLI UniFlash](https://software-dl.ti.com/ccs/esd/uniflash/docs/latest_qsguide.html)

### Сетап без фреймворку

1. **MSP430FR2433.** Встанови TI MSP430-GCC `9_3_1_2` із GCC 9.3.1.11 і support files 1.212. У `main.c` підключи `<msp430.h>` і в `main()` послідовно виконай `WDTCTL = WDTPW | WDTHOLD`, `P1OUT |= BIT0`, `P1DIR |= BIT0` та `PM5CTL0 &= ~LOCKLPM5`, потім залиш нескінченний цикл. Збери командою `msp430-elf-gcc -mmcu=msp430fr2433 -I"%MSP430_SUPPORT%" -L"%MSP430_SUPPORT%" -Os -g -o fr2433.elf main.c`, де `MSP430_SUPPORT` вказує на каталог із `msp430fr2433.h` і `msp430fr2433.ld`. Після старту має світитися червоний LED1. Для виробу зарезервуй визначені ділянки журналу та інформаційної FRAM. [MSP430-GCC 9_3_1_2](https://software-dl.ti.com/msp430/msp430_public_sw/mcu/msp430/MSPGCC/9_3_1_2/export/Release_Notes_9.3.1.2.html), [Регістрові приклади TI](https://www.ti.com/tool/download/SLAC700)
2. **CC2652R.** Встанови `gcc-arm-none-eabi-9-2019-q4-major` і використовуй пакет, установлений офіційним інсталятором SimpleLink SDK 8.33.00.16. Задай `TI_DEVICE` як шлях до `source/ti/devices/cc13x2_cc26x2` і перевір наявність `driverlib/bin/gcc/driverlib.lib` у цьому каталозі. Візьми пристроєві startup, CCFG і linker script. Збірка використовує `SetupTrimDevice()` із низькорівневої бібліотеки DriverLib для заводських калібрувань. `main.c` керує GPIO через регістри. Для BLE, Zigbee чи Thread потрібен окремий сумісний стек. [TI GCC startup](https://github.com/TexasInstruments/simplelink-lowpower-f2-sdk/blob/main/source/ti/devices/cc13x2_cc26x2/startup_files/startup_gcc.c), [TI CCFG](https://github.com/TexasInstruments/simplelink-lowpower-f2-sdk/blob/main/source/ti/devices/cc13x2_cc26x2/startup_files/ccfg.c), [Linker script CC26x2R1F](https://github.com/TexasInstruments/simplelink-lowpower-f2-sdk/blob/main/source/ti/devices/cc13x2_cc26x2/linker_files/cc26x2r1f.lds), [CC2652R Technical Reference Manual](https://www.ti.com/lit/pdf/swcu185)

   ```c
   #include "inc/hw_gpio.h"
   #include "inc/hw_ioc.h"
   #include "inc/hw_memmap.h"
   #include "inc/hw_prcm.h"
   #include "inc/hw_types.h"

   int main(void)
   {
       HWREG(PRCM_BASE + PRCM_O_PDCTL0PERIPH) = 1u;
       while (!(HWREG(PRCM_BASE + PRCM_O_PDSTAT0PERIPH) & PRCM_PDSTAT0PERIPH_ON)) {}
       HWREG(PRCM_BASE + PRCM_O_GPIOCLKGR) |= PRCM_GPIOCLKGR_CLK_EN;
       HWREG(PRCM_BASE + PRCM_O_CLKLOADCTL) = PRCM_CLKLOADCTL_LOAD;
       while (!(HWREG(PRCM_BASE + PRCM_O_CLKLOADCTL) & PRCM_CLKLOADCTL_LOAD_DONE)) {}
       HWREG(IOC_BASE + IOC_O_IOCFG6) = IOC_IOCFG6_PORT_ID_GPIO;
       HWREG(GPIO_BASE + GPIO_O_DOUTSET31_0) = 1u << 6;
       HWREG(GPIO_BASE + GPIO_O_DOE31_0) |= 1u << 6;
       for (;;) {}
   }
   ```

   Збери з `cmd.exe` у каталозі `main.c`. Перевір, що `.resetVecs` починається за `0x00000000`, а `.ccfg` лежить в області `FLASH_CCFG`. Після холодного запуску на LAUNCHXL-CC26X2R1 має світитися червоний LED на DIO6. [Виводи LaunchPad](https://software-dl.ti.com/zephyr-sdk/esd/AM62L/01_00_00_02/exports/zephyr/boards/ti/cc26x2r1_launchxl/doc/index.html)

   ```bat
   arm-none-eabi-gcc -mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard -Os -g -ffreestanding -nostartfiles ^
     -I"%TI_DEVICE%" -T "%TI_DEVICE%\linker_files\cc26x2r1f.lds" -Wl,-Map=cc2652r.map -o cc2652r.elf ^
     main.c "%TI_DEVICE%\startup_files\startup_gcc.c" "%TI_DEVICE%\startup_files\ccfg.c" ^
     "%TI_DEVICE%\driverlib\bin\gcc\driverlib.lib"
   ```
3. Запиши отриманий ELF або `.out` через UniFlash із відповідним `.ccxml`. Для покрокової перевірки відкрий цей самий файл символів у CCStudio і під’єднай eZ-FET або XDS110 відповідно до плати. Після цього перевір автономний запуск з пам’яті. [UniFlash](https://www.ti.com/tool/UNIFLASH)

## Перевірка й фіксація рішення

На MSP430 перевір відновлення журналу після перерваного запису, а також збереження потрібних областей FRAM під час оновлення прошивки. На CC2652R із радіостеком перевір підключення до другого пристрою, відновлення сесії й споживання в робочому циклі. Збережи карту пам’яті та конфігурацію програматора разом із результатом стенда. [MSP430FR2433 Datasheet](https://www.ti.com/lit/ds/symlink/msp430fr2433.pdf), [CC2652R Datasheet](https://www.ti.com/lit/ds/symlink/cc2652r.pdf)

Запиши обраний стек, розкладку енергонезалежної пам’яті та результати відновлення журналу або радіозв’язку.
