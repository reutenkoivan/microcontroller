# Nordic nRF52840 на nRF52840 DK

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для батарейного вузла задай роль у радіопротоколі, час від пробудження до обміну, допустимий струм у сні та поведінку після втрати зв’язку. Якщо є оновлення, визнач, які налаштування та ключі воно має зберегти. [Специфікація nRF52840](https://docs.nordicsemi.com/r/bundle/ps_nrf52840/), [nRF52840 DK](https://www.nordicsemi.com/Products/Development-hardware/nRF52840-DK)

## Цільова платформа

На DK для запису й налагодження використовуй вбудований J-Link через SWD. На Ubuntu встанови `nrf-udev` і перевір, що `nrfutil device list` бачить DK зі звичайного облікового запису. Для бездротової перевірки підготуй другий пристрій або шлюз, а для вимірювання струму визнач режим живлення стенда. [nRF52840 DK](https://www.nordicsemi.com/Products/Development-hardware/nRF52840-DK), [Вимоги Nordic для Linux](https://github.com/nrfconnect/sdk-nrf/blob/v3.2.0/doc/nrf/installation/recommended_versions.rst), [Апаратна довідка](../wiki/1-hardware-platforms.md#15-nordic-nrf)

## Вибір стека прошивки

**nRF Connect SDK із Zephyr** підходить для BLE, Thread та виробів, де потрібні керування живленням і завантажувач. **Bare metal** підходить для локальної периферії з простою моделлю виконання. Для радіо окремо потрібен підтримуваний стек. [nRF Connect SDK](https://www.nordicsemi.com/Products/Development-software/nRF-Connect-SDK), [Nordic nrfx](https://github.com/NordicSemiconductor/nrfx)

## Межі для розробника

Признач власника devicetree, Kconfig і карти пам’яті. Розробник додає обробку вимірювань через наявний драйвер. Зміни призначення виводів, розділів, джерел пробудження або політики стирання звіряй у рев’ю та перевіряй на відповідному стенді.

## Відтворюване середовище розробки

### Сетап через nRF Connect SDK і Zephyr

1. Для Ubuntu 24.04 x86-64 встанови nRF Util, потім `nrfutil install sdk-manager` і `nrfutil sdk-manager install v3.2.0`. Використовуй SDK `v3.2.0` та парний toolchain `v3.2.0`. Відкрий його shell через `nrfutil sdk-manager toolchain launch --ncs-version v3.2.0 --shell`. Для запису використовуй `nrfutil device` з цього комплекту і SEGGER J-Link 8.76. [Release Notes 3.2.0](https://nrfconnectdocs.nordicsemi.com/ncs/latest/nrf/releases_and_maturity/releases/release-notes-3.2.0.html), [Встановлення SDK та toolchain](https://docs.nordicsemi.com/r/bundle/nrfutil/page/nrfutil-sdk-manager/nrfutil-sdk-manager-install.html/basic-usage?contentId=6kjZSBGwAuqG_HqnSzfLXg), [Запуск toolchain](https://docs.nordicsemi.com/r/bundle/nrfutil/page/nrfutil-sdk-manager/nrfutil-sdk-manager-toolchain-launch.html/syntax)
2. З кореня workspace задай `$APP_DIR` як шлях до `zephyr/samples/hello_world` своєї версії SDK. Візьми `PROBE_SN` з `nrfutil device list`, збери, запиши на DK і перевір повідомлення у віртуальному UART-порті. Для покрокового запуску використовуй `west debug -d build/nrf52840`. [nRF52840 DK в Zephyr](https://docs.zephyrproject.org/latest/boards/nordic/nrf52840dk/doc/index.html#programming-and-debugging)

   ```sh
   west build -b nrf52840dk/nrf52840 -d build/nrf52840 "${APP_DIR:?Вкажи каталог застосунку}" &&
   west flash -d build/nrf52840 -r nrfutil --dev-id "${PROBE_SN:?Вкажи серійний номер J-Link}"
   ```

3. Для застосунку зафіксуй `west.yml` з ревізіями, `prj.conf`, devicetree overlays і sysbuild-конфігурацію за її наявності. Якщо збірка включає MCUboot або кілька образів, зберігай об’єднаний HEX, якщо його створює вибрана конфігурація, і ELF потрібних доменів. Підбери режим стирання, що зберігає потрібні розділи. [Збірка nRF Connect SDK](https://nrfconnectdocs.nordicsemi.com/ncs/latest/nrf/app_dev/config_and_build/building.html), [Артефакти збірки](https://nrfconnectdocs.nordicsemi.com/ncs/latest/nrf/app_dev/config_and_build/output_build_files.html), [Програмування](https://nrfconnectdocs.nordicsemi.com/ncs/latest/nrf/app_dev/programming.html)

### Сетап без фреймворку

1. Встанови Arm GNU Toolchain 13.3.Rel1 і візьми `mdk` із `nrfx` `v3.14.0` та CMSIS Core `5.9.0`. Для запису можна використати `nrfutil device` з комплекту вище або окремо встановити `nrfutil install device=2.15.2` для J-Link 8.76. Задай `NRFX_MDK` як шлях до `mdk`, а `CMSIS_CORE_INCLUDE` — до `CMSIS/Core/Include` у CMSIS_5. [Nordic nrfx v3.14.0](https://github.com/nordicsemi/nrfx/tree/v3.14.0), [CMSIS Core 5.9.0](https://github.com/ARM-software/CMSIS_5/tree/5.9.0/CMSIS/Core), [Сумісність nRF Util і J-Link](https://docs.nordicsemi.com/r/bundle/nrfutil/page/guides/installing.html/prerequisites)
2. У `main.c` підключи `nrf.h`. У `int main(void)` виконай `NRF_P0->OUTCLR = 1u << 13;`, `NRF_P0->DIRSET = 1u << 13;` і залиш нескінченний цикл. LED1 на DK має засвітитися після reset. У збірці startup Nordic викликає `SystemInit` і `main`, а `nrf52840_xxaa.ld` задає базу Flash `0x00000000`. [Виводи DK](https://github.com/nrfconnect/sdk-zephyr/blob/ncs-v3.2.0/boards/nordic/nrf52840dk/nrf52840dk_nrf52840.dts), [Startup](https://github.com/nordicsemi/nrfx/blob/v3.14.0/mdk/gcc_startup_nrf52840.S), [Linker script Nordic](https://github.com/nordicsemi/nrfx/blob/v3.14.0/mdk/nrf52840_xxaa.ld)

   ```sh
   mkdir -p build/nrf52840-register &&
   arm-none-eabi-gcc -mcpu=cortex-m4 -mthumb -mfloat-abi=soft -Os -g -ffreestanding \
     -DNRF52840_XXAA -D__START=main -D__STARTUP_CLEAR_BSS \
     -I"${NRFX_MDK:?}/" -I"${CMSIS_CORE_INCLUDE:?}" -L"$NRFX_MDK" \
     -T "$NRFX_MDK/nrf52840_xxaa.ld" -nostartfiles \
     -Wl,-Map=build/nrf52840-register/firmware.map -o build/nrf52840-register/firmware.elf \
     "$NRFX_MDK/gcc_startup_nrf52840.S" "$NRFX_MDK/system_nrf52840.c" main.c &&
   arm-none-eabi-objcopy -O ihex build/nrf52840-register/firmware.elf build/nrf52840-register/firmware.hex
   ```

3. Після успішної збірки візьми `$PROBE_SN` із `nrfutil device list` для виділеної тестової плати. Звір адреси свіжого HEX і межі сторінок Flash. Режим нижче стирає сторінки, яких торкається образ. Переконайся, що потрібні дані розташовані поза цими сторінками і HEX не містить записів для UICR, якщо її потрібно зберегти. Типовий режим без явного `chip_erase_mode` стирає внутрішню пам’ять і UICR. [Режими стирання nRF Util](https://docs.nordicsemi.com/r/bundle/nrfutil/page/nrfutil-device/nrfutil-device-program.html/programming-options?contentId=SP2VE_VVx2DTxeKpoa_iJg)

   ```sh
   nrfutil device program --serial-number "${PROBE_SN:?Вкажи серійний номер J-Link}" \
     --firmware build/nrf52840-register/firmware.hex \
     --options chip_erase_mode=ERASE_RANGES_TOUCHED_BY_FIRMWARE,verify=VERIFY_READ &&
   nrfutil device reset --serial-number "$PROBE_SN"
   ```

4. Відкрий той самий ELF через Arm GDB і SEGGER J-Link GDB Server, вибравши nRF52840 та SWD. Перевір reset і роботу GPIO, а для продукту відпрацюй дозволене відновлення через SWD. [SEGGER J-Link GDB Server](https://kb.segger.com/J-Link_GDB_Server)

## Перевірка й фіксація рішення

Для Zephyr проєкту опиши стенди в hardware map і запускай тести через Twister. Для радіовиробу перевір повторне з’єднання, поведінку після втрати живлення, струм у сні та споживання під час обміну. Для образу з MCUboot перевір оновлення та відновлення на конкретній розкладці розділів. [Zephyr Twister](https://docs.zephyrproject.org/latest/develop/twister/index.html), [nRF52840 DK](https://www.nordicsemi.com/Products/Development-hardware/nRF52840-DK)

Запиши обраний стек, варіант плати, конфігурацію пробудження та виміряні результати за заданого циклу сну й обміну.
