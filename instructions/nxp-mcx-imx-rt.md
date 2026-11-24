# NXP MCXN947 та i.MX RT1062

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для MCXN947 визнач, чи потрібні обидва ядра, що вони обмінюють та коли запускається друге ядро. Для RT1062 задай обсяг буферів, тип зовнішньої NOR Flash і допустимий час холодного старту. Для обох цілей опиши відновлення після невдалого оновлення. [Архітектура MCX N](https://www.nxp.com/docs/en/data-sheet/MCXNP184M150F70.pdf), [i.MX RT1060 Datasheet](https://www.nxp.com/docs/en/nxp/data-sheets/IMXRT1060CEC.pdf)

## Цільова платформа

Для FRDM-MCXN947 почни з ядра `cm33_core0` і внутрішньої Flash. Для MIMXRT1060-EVKB на RT1062 використовуй FlexSPI NOR і вбудований LPC-Link2 із сумісною прошивкою. FRDM використовує вбудований MCU-Link. [FRDM-MCXN947](https://www.nxp.com/document/guide/getting-started-with-frdm-mcxn947%3AGS-FRDM-MCXN947), [Адаптери плат NXP](https://mcuxpresso.nxp.com/mcuxsdk/latest/gsd/package/default_debug_interfaces.html), [Апаратна довідка](../wiki/1-hardware-platforms.md#18-nxp-mcx--imx-rt)

## Вибір стека прошивки

**MCUXpresso SDK** підходить, коли потрібно узгодити драйвери, плати, багатоядерний запуск або конфігурацію XIP. **Bare metal** має сенс для вузької функції з власним startup, якщо команда може перевірити образ і конкретний Boot ROM сценарій. [Розробка через MCUXpresso CLI](https://mcuxpresso.nxp.com/mcuxsdk/latest/gsd/cli.html), [Формат образу RT1060](https://mcuxpresso.nxp.com/mcuxsdk/25.06.00/html/middleware/mcu_bootloader/docs/iMXRT1060_Manufacturing_User_Guide/topics/imx_rt_bootable_image.html)

## Межі для розробника

Власник конфігурації плати визначає розміщення образів, час запуску ядер і налаштування Flash. Розробник змінює обробку даних через наявний інтерфейс. Зміни linker script, заголовка завантаження або спільної пам’яті видно в рев’ю і порівнянні карт секцій, а працездатність підтверджує холодний старт.

## Відтворюване середовище розробки

### Сетап через MCUXpresso SDK

1. Для Ubuntu 24.04 x86-64 бери MCUXpresso SDK `25.12.00` і Arm GNU Toolchain `14.2.Rel1` (`arm-none-eabi`). Встанови CMake, Ninja, LinkServer і Python-залежності за інструкцією цього випуску. Створи workspace через `west init -m https://github.com/nxp-mcuxpresso/mcuxsdk-manifests.git --mr v25.12.00 mcux-workspace` і виконай `west update` у `mcux-workspace`. Для збірки перейди в `mcux-workspace/mcuxsdk` та експортуй `ARMGCC_DIR` на каталог Arm GNU Toolchain. [Manifest v25.12.00](https://github.com/nxp-mcuxpresso/mcuxsdk-manifests/blob/v25.12.00/west.yml), [Документація 25.12.00](https://mcuxpresso.nxp.com/mcuxsdk/25.12.00/html/gsd/cli.html)
2. Із `mcux-workspace/mcuxsdk` виконай `west list_project -p examples/demo_apps/hello_world`, потім вибери один із двох блоків нижче. На EVKB конфігурація `flexspi_nor_sdram_debug` використовує виконання із зовнішньої Flash і SDRAM. `-p always` перебудовує типовий каталог збірки, тому в CI використовуй незалежні робочі каталоги для двох плат. [MCUXpresso CLI 25.12.00](https://mcuxpresso.nxp.com/mcuxsdk/25.12.00/html/gsd/cli.html), [Конфігурації `hello_world` з SDK 25.12.00](https://github.com/nxp-mcuxpresso/mcuxsdk-examples/blob/8bbe2c59cf419ea0222fd5309756a07483267c80/demo_apps/hello_world/example.yml)

   ```sh
   # Лише для FRDM-MCXN947
   west build -b frdmmcxn947 examples/demo_apps/hello_world --toolchain armgcc --config debug -Dcore_id=cm33_core0 -p always &&
   west flash -r linkserver
   ```

   ```sh
   # Лише для MIMXRT1060-EVKB
   west build -b evkbmimxrt1060 examples/demo_apps/hello_world --toolchain armgcc --config flexspi_nor_sdram_debug -p always &&
   west flash -r linkserver
   ```

3. Під’єднуй лише плату, для якої запускаєш блок. Після запису перевір `hello world.` через UART на `115200` бод і запусти `west debug -r linkserver` для налагодження. Для власного застосунку зафіксуй linker configuration, вхідні файли Config Tools за їх наявності та boot-конфігурацію. Для MCXN947 з кількома ядрами вкажи, яке ядро запускає інше і які образи потрібні для стенда. [Налагодження NXP](https://mcuxpresso.nxp.com/mcuxsdk/latest/gsd/cli.html#flashing-and-debugging), [Приклад EVKB з SDK 25.12.00](https://github.com/nxp-mcuxpresso/mcuxsdk-examples/blob/8bbe2c59cf419ea0222fd5309756a07483267c80/_boards/evkbmimxrt1060/demo_apps/hello_world/example_board_readme.md)

### Сетап без фреймворку

1. Використовуй Arm GNU Toolchain 14.2.Rel1 і пристроєві файли з workspace SDK 25.12.00. У своєму каталозі створи `main.c` з тестом запуску нижче. Обидва мінімальні образи використовують внутрішню RAM та тактування після Boot ROM і reset. Вони пропускають `SystemInit`, а startup очищає `.bss` і передає керування `main`. Для продукту перевір вимоги до `SystemInit` і захисту окремо.

   ```c
   volatile unsigned int startup_marker;

   int main(void)
   {
       startup_marker = 0x4E585031u;
       for (;;) {}
   }
   ```

2. **MCXN947.** Задай `MCX_DEVICE` як шлях до `devices/MCX/MCXN/MCXN947` у checkout SDK. Для виділеної плати в режимі незахищеного старту з внутрішньої Flash перевір джерело завантаження за документацією CMPA. У карті образу вектори мають починатися за `0x00000000`. [Пристроєві файли MCXN947](https://github.com/nxp-mcuxpresso/mcux-devices-mcx/tree/143c345eaa07b259ab93b06243fc5b64ab09552b/MCXN/MCXN947/gcc), [Завантаження MCX N Series](https://mcuxpresso.nxp.com/secure/26.09/06_processor_specific_workflow.html)

   ```sh
   arm-none-eabi-gcc -mcpu=cortex-m33 -mthumb -mfloat-abi=soft -Os -g -ffreestanding \
     -D__START=main -D__STARTUP_CLEAR_BSS -D__NO_SYSTEM_INIT -nostdlib \
     -T "${MCX_DEVICE:?}/gcc/MCXN947_cm33_core0_flash.ld" \
     -Wl,-Map=mcxn947-register.map -o mcxn947-register.elf \
     "$MCX_DEVICE/gcc/startup_MCXN947_cm33_core0.S" main.c
   ```

3. **RT1062.** Для QSPI NOR на EVKB використовуй Flash Configuration Block цієї плати та пристроєвий заголовок, який формує Image Vector Table (IVT) і Boot Data. Задай `RT_SERIES` як шлях до `devices/RT/RT1060`, `RT_XIP` — до `devices/RT/RT1050/MIMXRT1052/xip`, `EVKB_XIP` — до `examples/_boards/evkbmimxrt1060/xip`, `SDK_COMMON` — до `drivers/common`, `CMSIS_CORE_INCLUDE` — до `CMSIS/Core/Include` в `arch/arm/CMSIS`. У карті перевір базу XIP-образу `0x60000000` та IVT за `0x60001000`. У цьому тесті використовується внутрішня RAM, тому Device Configuration Data для зовнішньої SDRAM не потрібні. [Linker script RT1062](https://github.com/nxp-mcuxpresso/mcux-devices-rt/blob/29b3e73812f8f2a194e7d4a7a370f645a0cd5484/RT1060/MIMXRT1062/gcc/MIMXRT1062xxxxx_flexspi_nor.ld), [FCB для EVKB](https://github.com/nxp-mcuxpresso/mcuxsdk-examples/blob/8bbe2c59cf419ea0222fd5309756a07483267c80/_boards/evkbmimxrt1060/xip/evkbmimxrt1060_flexspi_nor_config.c), [Заголовок XIP](https://mcuxpresso.nxp.com/mcuxsdk/25.12.00/html/boards/RT/evkbmimxrt1060/gettingStarted/topics/how_to_add_or_remove_boot_header_for_xip_targets.html)

   ```sh
   RT_DEVICE="${RT_SERIES:?}/MIMXRT1062" &&
   arm-none-eabi-gcc -mcpu=cortex-m7 -mthumb -mfloat-abi=soft -Os -g -ffreestanding \
     -D__START=main -D__STARTUP_CLEAR_BSS -D__NO_SYSTEM_INIT -DXIP_BOOT_HEADER_ENABLE=1 \
     -DCPU_MIMXRT1062DVL6B -DBOARD_FLASH_SIZE=0x800000U -nostdlib \
     -I"$RT_DEVICE" -I"$RT_DEVICE/drivers" -I"$RT_SERIES/periph" \
     -I"${SDK_COMMON:?}" -I"${CMSIS_CORE_INCLUDE:?}" \
     -T "$RT_DEVICE/gcc/MIMXRT1062xxxxx_flexspi_nor.ld" \
     -Wl,-Map=rt1062-register.map -o rt1062-register.elf \
     "$RT_DEVICE/gcc/startup_MIMXRT1062.S" "${RT_XIP:?}/fsl_flexspi_nor_boot.c" \
     "${EVKB_XIP:?}/evkbmimxrt1060_flexspi_nor_config.c" main.c
   ```

4. Після успішної збірки задай `FIRMWARE_ELF` для свіжого ELF вибраної плати. У LinkFlash вибери програматор і точний пристрій. Перевір, що в конфігурації пристрою до діапазону образу прив’язаний Flash driver для потрібної пам’яті. Значення `$DEVICE` і `$PROBE_SN` візьми з перегляду команд LinkFlash. Після запису вимкни й увімкни плату. Під’єднай GDB до запущеної цілі з режимом attach без запису та reset. Значення `startup_marker` має бути `0x4E585031`. [LinkFlash](https://mcuxpresso.nxp.com/linkserver/26.09/LinkFlash.html), [LinkServer Flash drivers і CLI](https://mcuxpresso.nxp.com/linkserver/latest/FlashDrivers.html)

   ```sh
   LinkServer flash --probe "${PROBE_SN:?Вкажи серійний номер програматора}" \
     "${DEVICE:?Вкажи модель із LinkFlash}" load "${FIRMWARE_ELF:?Вкажи ELF поточної збірки}" &&
   LinkServer flash --probe "$PROBE_SN" "$DEVICE" verify "$FIRMWARE_ELF"
   ```

## Перевірка й фіксація рішення

На FRDM із двома активними ядрами перевір порядок старту, спільну пам’ять і відновлення після reset. На EVKB з XIP перевір холодний запуск із конкретної зовнішньої Flash і відновлення через передбачений режим Boot ROM. На стенді можна використати LinkServer Runner для запуску й збору результатів тесту. [LinkServer Runner](https://mcuxpresso.nxp.com/linkserver/26.09/Runner.html), [Boot-конфігурація RT1060](https://mcuxpresso.nxp.com/secure/26.09/06_processor_specific_workflow.html)

Запиши вибір стека, активні ядра, джерело завантаження й версію образу, з яким підтверджено холодний запуск.
